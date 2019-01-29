# Sidekiq Notes
## About dead job callback
Sidekiq handles it by specifying `sidekiq_retries_exhausted`. Check [here](https://github.com/mperham/sidekiq/wiki/Error-Handling) for usage.
Not sure why but Mike didn't talk much about how to use this callback with examples, so below is a list of handler's input.

```
["class", "args", "retry", "queue", "backtrace", "jid", "created_at", "enqueued_at", "error_message", "error_class", "failed_at", "retry_count", "error_backtrace", "retried_at"]
```

The input is actually a hash, which has the above keys. Please be aware that it is serialized.
