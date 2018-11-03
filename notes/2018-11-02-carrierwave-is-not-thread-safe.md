# Carrierwave Is NOT Thread-safe

In our Rails backend, we use Resque as background job system. As the company's business grows, Resque starts to show signs of incapability. Therefore we decided to migrate from Resque to [Sidekiq](https://github.com/mperham/sidekiq), which can be considered as a multi-threaded version of Sidekiq, briefly.

Image uploading/processing jobs are ones of the most created jobs, they are processed in the following steps:
1. Download images from a remote URL;
2. Resize/Optimize images using mini_magick;
3. Upload processed images to another S3 bucket, update corresponding records.

After I migrated these image-related jobs to Sidekiq, soon I noticed that most uploaded images became blurry. The code remained almost unchanged during the migration, so what could possibly go wrong?

I'm not gonna assume that everyone knows Carrierwave, Resque and Sidekiq, therefore bear with me if you find this post too rigmarole.

## How

Here is the code snippet:

```ruby
def store_for_mobile(file)
  self.class.process resize_to_limit: [1080, 810]
  @directory = File.join('banners/mobile', device)
  store!(file)
  end

def store_for_web(file)
  self.class.process resize_to_limit: [750, 562]
  @directory = 'stores/750x562/'
  store!(file)
end

def store_header(file)
  self.class.process resize_to_limit: [640, 640]
  @directory = 'headers/images/consumer_app_brand_logos/'
  store!(file)
end

def store_header_mailer(file)
  self.class.process resize_to_limit: [360, 120]
  @directory = 'headers/images/360x120/'
  store!(file)
end
```

For each uploaded image, it will be resized into multiple different dimensions. The above methods are executed in different jobs respectively, which means sometimes they could be running concurrently. For uncertain reasons, images which should be resized to [1080, 810], [750, 562], [640, 640] are resized into [360, 120], thus when displayed on webpage and mobile applications, they become blurry.

Because the code is fairly simple and mainly uses Carrierwave...

## So I started to read Carrierwave's source code
No offense, but after more than ten years of continuous development, Carrierwave has become really painful to read. Luckily the issue is only tied to uploader.
Soon I noticed that in https://github.com/carrierwaveuploader/carrierwave/blob/master/lib/carrierwave/uploader/processing.rb, there are some interesting code:

First, the class attribute:
```ruby
included do
   class_attribute :processors, :instance_writer => false
   self.processors = []

   before :cache, :process!
end
```
Because Carrierwave allows you to use mini_magick to process images before upload, this class variable list `processors` is used to keep these specifications. For example, if you want to resize into a smaller image, let's say 100x100, you simply use class method `process`to add the specification to the list (https://github.com/carrierwaveuploader/carrierwave/blob/master/lib/carrierwave/uploader/processing.rb#L62).

```ruby
self.class.process resize_to_limit: [100, 100]
```
After this, while Carrierwave starts to process image, the uploader will call `process!` to apply the specification to mini_magick ([code](https://github.com/carrierwaveuploader/carrierwave/blob/master/lib/carrierwave/uploader/processing.rb#L71)). The only problem here is, `process!` is not a class method but an instance one.

If nothing unexpected happened, the processor list will be shared by all uploaders and then shits will hit the fan. Alright then, lets...

## Reproduce the issue and see what's going on
First start Sidekiq.

### Sidekiq
We enqueue bunch of image uploader jobs:
```ruby
['store_for_mobile', 'store_for_web', 'store_header', 'store_header_mailer'].each do
  SidekiqWorker::ImageUploaderWorker.perform_async(method, image_url)
end
```

The above code creates four jobs, each job resizes the same image (to different dimensions) and uploads them.

When the first job `store_for_mobile` being processed, we have the `processors` as:

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
[[[:resize_to_limit, [1080, 810], nil]]
```

This looks fine, and after this the image will be processed by the code [here](https://github.com/carrierwaveuploader/carrierwave/blob/f15f4ba71dd2c78e8df494bf65938bbaf13960f9/lib/carrierwave/processing/mini_magick.rb#L73). Let's continue.

When the second job `store_for_web` being processed, the `processors` became:
```ruby
[[:resize_to_limit, [1080, 810], nil], [:resize_to_limit, [750, 562], nil]]
```

Now this looks not fine. So what will happen to the image? Both operations will be applied to the image, so the image first gets resized into 1080x810, then gets resized into 750x562 by a [loop](https://github.com/carrierwaveuploader/carrierwave/blob/master/lib/carrierwave/uploader/processing.rb#L75) of executing all operations.

Because when you use `resize_to_limit` to process an image, image_magick will always try to resize into smaller one but never enlarge it, so eventually the image will be resized into 750x562 (depending on whether width/height reaches the limit first).

Therefore, after all four jobs being finished, the `processors` will become
```ruby
[[:resize_to_limit, [1080, 810], nil], [:resize_to_limit, [750, 562], nil], [:resize_to_limit, [640, 640], nil], [:resize_to_limit, [360, 120], nil]]
```

And if we run these jobs one more time, the list will become:

```ruby
(byebug) self.class.processors
[[:resize_to_limit, [1080, 810], nil], [:resize_to_limit, [750, 562], nil], [:resize_to_limit, [640, 640], nil], [:resize_to_limit, [360, 120], nil],[:resize_to_limit, [1080, 810], nil], [:resize_to_limit, [750, 562], nil], [:resize_to_limit, [640, 640], nil], [:resize_to_limit, [360, 120], nil]]
```

As as result of that 360x120 has been specified in the first round, all images in the second round has been resized into 360x120. Thus when these images are displayed on webpage and mobile applications, they become blurry. Eventually after tens of thousands of these jobs, the list will grow massively and soon, we run out of space, then OOM is triggered, the worker pod gets destroyed by Kubernetes.

```
OOM k8s_sidekiq_backend-sidekiq-rmagick-stable-1260160206-67xth_backend_9a7406fe-bfaf-11e8-9aa3-069c71b490e4_2
DIE k8s_sidekiq_backend-sidekiq-rmagick-stable-1260160206-67xth_backend_9a7406fe-bfaf-11e8-9aa3-069c71b490e4_2
```

### Resque
But we never saw these issues (blurry images, OOMs) while using Resque, does Resque actually have this growing `processors` list? Let's have a look at how the jobs are processed in Resque. The steps are pretty much the same as above.

The first job `store_for_mobile`:
```ruby
[[:optimize, [1080, 810], nil]]
```

Now the second job `store_for_web`:
```ruby
[[:resize_to_limit, [750, 562], nil]]
```

Now let's run job `store_for_header`:
```ruby
[[:resize_to_limit, [640, 640], nil]]
```

Then `store_for_header_mailer`:
```ruby
[[:resize_to_limit, [360, 120], nil]]
```


## Why it didn't happend in Resque but Sidekiq

# To be finished...
