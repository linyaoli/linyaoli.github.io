# A Typical Design of Database Backfill System

At some point, we may need to backfill new columns in a frequently queried table. Assume there are massive amount of
records, and each record has blah blah..

Generate backfill audits
-----------------------
To maintain durable backfill tasks in the rails application, especially while
backfilling huge loads of records, we'd like to make these audits trackable, and
work on them in a specific order.

For that, we need a table as the following:

|backfill_audits|
|--|
|id|
|account_id|
|task|
|created_at|
|started_at|
|completed_at|
|updated_at|

The only notable thing here is, on an extreme case when we are backfilling shits during a migration, new record can be
inserted, therefore a re-backfill is needed. For this situation, we put a waterline inside the audits marking a period
to avoid a whole table re-backfilling.

Since this case does not happen constantly, one day gap is good enough (or extend to even longer depending on the amount of records).

```ruby
module DurableBackfill
  class BackfillAudit < ActiveRecord::Base
    WATERLINE_ACCOUNT_ID = -10

    default_scope { where("account_id != #{WATERLINE_ACCOUNT_ID}") }

    scope :for_task, ->(task_name) { where(task: task_name) }
    scope :incomplete, -> { where(completed_at: nil) }

    belongs_to :account

    def self.waterline_date(task)
      unscoped.where(account_id: WATERLINE_ACCOUNT_ID, task: task).first_or_create.created_at + 1.day
    end
  end
end
```

Daemon to run backfill tasks
---------------

Before starting the actual backfill task, let's create backfill audits.
Unless we have fully backfilled the table, backfill task need to run continuously in the background.
Three minutes is not necessarily a good gap, but it is always open to change based on actual circumstances.

```ruby
require_relative 'task_list'

module DurableBackfill
  class Runner
    def self.tasks
      DurableBackfill::Task.descendants
    end

    def run
      generate_backfills
      while true do
        work_backfills
        sleep(3.minutes)
      end
    end

    def generate_backfills
      tasks.each do |task|
        ActiveRecord::Base.on_all_shards do |shard_id|
          task.create_audits!
        end
      end
    end

    def work_backfills
      tasks.each do |task|
        ActiveRecord::Base.on_all_shards do |shard_id|
          task.audits.incomplete.find_each do |audit|
            task.new(audit).work_audit
          end
        end
      end
    end

    private

    def tasks
      self.class.tasks
    end
  end
end
```

A General Task
-----------------

A waterline account is not a real account, therefore do be aware that its id must not conflict with real account ids.

```ruby
module DurableBackfill
  class Task
    class RescheduleTask < StandardError ; end
    WATERLINE_ACCOUNT_ID = -10

    class << self
      attr_accessor :title

      def audits
        DurableBackfill::BackfillAudit.for_task(title)
      end

      def create_audits!
        waterline = DurableBackfill::BackfillAudit.waterline_date(title)

        DurableBackfill::BackfillAudit.transaction do
          created_accounts = DurableBackfill::BackfillAudit.for_task(title).pluck(:account_id).to_set

          Account.shard_local.where("created_at < ?", waterline).where("id != #{Account.system_account_id}").each do |account|
            next if created_accounts.include?(account.id)

            DurableBackfill::BackfillAudit.create!(task: title, account: account)
          end
        end
      end
    end

    def initialize(audit)
      @audit = audit
    end

    def work(account)
      raise "must implement work!"
    end

    def work_audit
      @audit.started_at = Time.now
      if @audit.account(true).shard_local?
        puts "work on account#{@audit.account.id}"
        work(@audit.account)
      end

      @audit.completed_at = Time.now
      @audit.save!
    rescue RescheduleTask => e
      # leave audit untouched
    end

    def reschedule!
      raise RescheduleTask.new
    end
  end
end
```

Task list
----------

We use a tasks folder to keep all actual tasks.

```ruby
files = Dir.glob(File.dirname(__FILE__) + "/tasks/*.rb").each do  |f|
  require f
end
```

A sample backfill task
-----------

Now I'd like to backfill the column number_of_employee in table `company`.

```ruby
module DurableBackfill
  class BackfillNumberOfEmployee < Task
    self.title = 'backfill_number_of_employee'

    def work(account)
      Rails.cache.write("durable-backfill-number-of-employee-#{account.id}", true)

      Company.where(:account_id => account.id).find_each do |t|
        t.update_attribute(:number_of_employee, t.count_employee)
      end
    end
  end
end
```
