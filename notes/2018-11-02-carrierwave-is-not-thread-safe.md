# Carrierwave Is NOT Thread-safe

> I'm not gonna assume that everyone knows what beneath Carrierwave, Resque and Sidekiq, so bear with me if you find this post too rigmarole.


# Background
In our Rails backend, we used Resque as background job system. As the traffic grew, Resque started to show signs of incapability. Therefore we decided to migrate from Resque to [Sidekiq](https://github.com/mperham/sidekiq), which can be considered as a multi-threaded version of Resque, not precisely((also horizonally scaling Resque is fine).

Image uploading/processing job is one of the most executed jobs, which is processed in the following steps:
1. Download images from a remote URL;
2. Resize/Optimize images using mini_magick;
3. Upload processed images to another S3 bucket, update corresponding records.

After I migrated these image-related jobs to Sidekiq, soon we noticed that many uploaded images became blurry. As Sidekiq is almost compatible with Resque jobs, the code remained unchanged during the migration, so what could possibly go wrong?

## How

For each uploaded image, it will be resized into multiple different dimensions. The below methods are executed in different jobs respectively, which means sometimes they could run concurrently. With Sidekiq processing these jobs, images which should have been resized to [1080, 810], [750, 562] and  [640, 640], are resized into [360, 120], thus when they are displayed on webpage and mobile applications, they become blurry.


```ruby
# job 1
def store_for_mobile(file)
  self.class.process resize_to_limit: [1080, 810]
  @directory = File.join('banners/mobile', device)
  store!(file)
  end

# job 2
def store_for_web(file)
  self.class.process resize_to_limit: [750, 562]
  @directory = 'stores/750x562/'
  store!(file)
end

# job 3
def store_header(file)
  self.class.process resize_to_limit: [640, 640]
  @directory = 'headers/images/consumer_app_brand_logos/'
  store!(file)
end

# job 4
def store_header_mailer(file)
  self.class.process resize_to_limit: [360, 120]
  @directory = 'headers/images/360x120/'
  store!(file)
end
```

Because the code is extremely simple(that not a single line could go wrong) and mainly uses Carrierwave...

## So I started to read Carrierwave's source code
No offense, but after more than ten years of continuous development, Carrierwave has become really painful to read. Luckily the issue is only tied to uploader.
Soon I noticed that in https://github.com/carrierwaveuploader/carrierwave/blob/master/lib/carrierwave/uploader/processing.rb, there are some interesting code:

First, the use of class attribute:
```ruby
included do
   class_attribute :processors, :instance_writer => false
   self.processors = []

   before :cache, :process!
end
```
Because Carrierwave allows you to use mini_magick to process images before upload, this class variable list `processors` is used to keep these specifications. For example, if you want to resize into a smaller image, let's say 100x100, you simply use class method `process` to add the specification to the list (https://github.com/carrierwaveuploader/carrierwave/blob/master/lib/carrierwave/uploader/processing.rb#L62). e.g.

```ruby
self.class.process resize_to_limit: [100, 100]
```

Then Carrierwave starts to process the image. The uploader will call `process!` to apply the specification to mini_magick ([code](https://github.com/carrierwaveuploader/carrierwave/blob/master/lib/carrierwave/uploader/processing.rb#L71)). The only problem here is, `process!` is not a class method but an instance one. No doubt that the processor list will be shared by all uploaders and then shits will hit the fan.

Alright then, let's reproduce the issue and see...

## What's going on
Let's check Sidekiq first.

### Sidekiq
Enqueue a bunch of the above image uploader jobs:

```ruby
['store_for_mobile', 'store_for_web', 'store_header', 'store_header_mailer'].each do
  SidekiqWorker::ImageUploaderWorker.perform_async(method, image_url)
end
```

The above code creates four jobs, each job resizes the same image from `image_url` (to different dimensions) and uploads them. When the first job `store_for_mobile` being processed, we have the `processors` as:

```ruby
[72, 81] in /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/carrierwave-0.11.2/lib/carrierwave/uploader/processing.rb
   72:       #
   73:       def process!(new_file=nil)
   74:         return unless enable_processing
   75:         byebug
   76:
=> 77:         self.class.processors.each do |method, args, condition|
   78:           if(condition)
   79:             if condition.respond_to?(:call)
   80:               next unless condition.call(self, :args => args, :method => method, :file => new_file)
   81:             else
(byebug) self.class.processors
[[:resize_to_limit, [1080, 810], nil]]
```

This looks fine, then the image will be processed by the code [here](https://github.com/carrierwaveuploader/carrierwave/blob/f15f4ba71dd2c78e8df494bf65938bbaf13960f9/lib/carrierwave/processing/mini_magick.rb#L73). Let's continue.

When the second job `store_for_web` being processed, the `processors` became:
```ruby
[[:resize_to_limit, [1080, 810], nil], [:resize_to_limit, [750, 562], nil]]
```

Now this looks not fine. So what will happen to the image?
Both operations will be applied to the image, so the image firstly gets resized into 1080x810, then gets resized into 750x562 by a [loop](https://github.com/carrierwaveuploader/carrierwave/blob/master/lib/carrierwave/uploader/processing.rb#L75) of executing all operations.

Because when you use `resize_to_limit` to process an image, image_magick will always try to resize it into smaller one but will never enlarge it, so eventually the image will be resized into 750x562 (depending on whether width/height reaches the limit first).

Therefore, after all four jobs been finished, the `processors` will become:

```ruby
[[:resize_to_limit, [1080, 810], nil], [:resize_to_limit, [750, 562], nil], [:resize_to_limit, [640, 640], nil], [:resize_to_limit, [360, 120], nil]]
```

And if we run these jobs one more time, the list will become:

```ruby
(byebug) self.class.processors
[[:resize_to_limit, [1080, 810], nil], [:resize_to_limit, [750, 562], nil], [:resize_to_limit, [640, 640], nil], [:resize_to_limit, [360, 120], nil],[:resize_to_limit, [1080, 810], nil], [:resize_to_limit, [750, 562], nil], [:resize_to_limit, [640, 640], nil], [:resize_to_limit, [360, 120], nil]]
```

As a result that 360x120 has been added in the first round, all images in the second round will be resized into 360x120. Thus when these images are displayed on webpage and mobile applications, they become blurry. Eventually after tens of thousands of these jobs, the list will grow massively and soon, we run out of memory, then OOM is triggered, the worker pod gets destroyed by Kubernetes.

Event reports from Datadog:
```
OOM k8s_sidekiq_backend-sidekiq-rmagick-stable-1260160206-67xth_backend_9a7406fe-bfaf-11e8-9aa3-069c71b490e4_2
DIE k8s_sidekiq_backend-sidekiq-rmagick-stable-1260160206-67xth_backend_9a7406fe-bfaf-11e8-9aa3-069c71b490e4_2
```

<b>Moreover, I also tried to run the code in Rails directly, the list still grew. </b>

### Resque
But we never saw these issues (blurry images, OOMs) while using Resque, does Resque actually have this growing `processors` list? Let's have a look at how the jobs are processed in Resque. The steps are pretty much the same as above.

For the first job `store_for_mobile`:
```ruby
[[:optimize, [1080, 810], nil]]
```

Then the second job `store_for_web`:
```ruby
[[:resize_to_limit, [750, 562], nil]]
```

Let's run job `store_for_header`:
```ruby
[[:resize_to_limit, [640, 640], nil]]
```

Finally `store_for_header_mailer`:
```ruby
[[:resize_to_limit, [360, 120], nil]]
```

Hah! That's why Resque didn't have the issues -- the `processors`s are always reset before being used by another job. Now with all these figured out, there is only one question left:

## Why does it only happen in Sidekiq?

Let's first look at Sidekiq's source code. It's pretty straight-forward, it just spawns some threads(depending on how many you specify in config) to process jobs. ([code](https://github.com/mperham/sidekiq/blob/aa9745e9b29d408ac423356af50859cb9c382135/lib/sidekiq/processor.rb#L62-L85))

```ruby
    def start
      @thread ||= safe_thread("processor", &method(:run))
    end

    private unless $TESTING

    def run
      begin
        while !@done
          process_one
        end
        @mgr.processor_stopped(self)
      rescue Sidekiq::Shutdown
        @mgr.processor_stopped(self)
      rescue Exception => ex
        @mgr.processor_died(self, ex)
      end
    end

    def process_one
      @job = fetch
      process(@job) if @job
      @job = nil
    end
```
So as long as the thread remains, the class attribute `processors` will always be shared by the jobs, just like what was found above, nothing particularly hard to believe.

Now let's look at Resque's source code on processing jobs. ([code](https://github.com/resque/resque/blob/master/lib/resque/worker.rb))
Resque has this ENV variable `FORK_PER_JOB`(`true` by default), it controls how Resque processes a job. With it on, Resque will call `perform_with_fork` to fork a new CHILD process to execute the job:
```ruby
      begin
        @child = fork do
          unregister_signal_handlers if term_child
          perform(job, &block)
          exit! unless run_at_exit_hooks
        end
      rescue NotImplementedError
        @fork_per_job = false
        perform(job, &block)
        return
      end
```

Although unix `fork()` does copy variables into the new process, but any allocation after the fork will not be shared between child processes. Therefore, even class variable `processors` is modified, other child processes won't be affected, and thus we will never see it grows.

This is most-likely one of the reasons why Resque is slower than Sidekiq.

## Conclusion
It is probably not precise enough to use the topic <b>Carrierwave is not thread-safe</b>, as even in single thread worker, the issue still exists. So be careful while using Sidekiq, when the author [claimed](https://github.com/mperham/sidekiq/wiki/Problems-and-Troubleshooting#threading):

> Sidekiq is multithreaded so your Workers must be thread-safe.

He really meant it :) , and you should be careful.
