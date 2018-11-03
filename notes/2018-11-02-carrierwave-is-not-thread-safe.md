# Carrierwave Is NOT Thread-safe

In our Rails backend, we use Resque as background job system. As the company's business grows, Resque starts to show signs of incapability. Therefore we decided to migrate from Resque to [Sidekiq](https://github.com/mperham/sidekiq), which can be considered as a multi-threaded version of Sidekiq, briefly.

Image uploading/processing jobs are ones of the most created jobs, they are processed in the following steps:
1. Download images from a remote URL;
2. Resize/Optimize images using mini_magick;
3. Upload processed images to another S3 bucket, update corresponding records.

After I migrated these image-related jobs to Sidekiq, soon I noticed that most uploaded images became blurry. The code remained almost unchanged during the migration, so what could possibly go wrong?

## How

Here is the code snippet:

```ruby
def store_for_mobile(file)
  self.class.process optimize: [1080, 810]
  %w(android ios).each do |device|
    @directory = File.join('banners/mobile', device)
    store!(file)
  end
end

def store_for_web(file)
  self.class.process optimize: [750, 562]
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

If nothing unexpected happened, the processor list will be shared by all uploaders and then shits will hit the fan.



# To be finished...
