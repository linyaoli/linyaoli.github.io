# List of important/useful rubygems

### Warden
General Rack Authentication Framework.

### CarrierWave
Upload files and map them to ORMs, store them on different backends.

### Kaminari
Kaminari is a Scope & Engine based, clean, powerful, customizable and sophisticated paginator.

### Sinatra
Sinatra is a DSL for quickly creating web applications in Ruby with minimal effort.

### Rubocop
Assures that your code conforms to the Ruby Style Guide.

### Rack
Rack provides a minimal, modular, and adaptable interface for developing web applications in Ruby. 

### Faraday 
Simple, but flexible HTTP client library, with support for multiple backends.

### Factory\_girl
factory\_girl provides a framework and DSL for defining and using factories.
```
require 'factory_girl'

Factory.define :user do |f|
  f.sequence(:user_name){ |n| "fooAPL#{n}" }
  f.password 'ups'
  f.email 'test@autoplenum.de'
end

Factory(:user)
```

### RMagick
RMagick is an interface between the Ruby programming language and the ImageMagick and GraphicsMagick image processing libraries.

### CanCan
CanCan is an authorization library for Ruby on Rails which restricts what resources a given user is allowed to access and is decoupled from user roles.

### Nokogiri
Nokogiri is an HTML, XML, SAX, and Reader parser.

### Capistrano
Capistrano is a utility and framework for executing commands in parallel on multiple remote machines, via SSH. It uses a simple DSL (borrowed in part from Rake) that allows you to define tasks, which may be applied to machines in certain roles. 

### Omniauth
OmniAuth is a Ruby authentication framework that provides a standardized interface to many different authentication providers such as Facebook, OpenID, and even traditional username and password.

### Bundler
Bundler is a tool that manages gem dependencies for your ruby application.

### Resque
Resque is a Redis-backed library for creating background jobs, placing those jobs on multiple queues, and processing them later. Resque is heavily inspired by DelayedJob.

### Chef
Chef is a system integration framework designed to bring the benefits of configuration management to your entire infrastructure. With Chef, you can manage your servers by writing code, not by running commands.

### Rspec
a testing framework mainly used in BDD and TDD environments.

### Simplecov 
Show the percentage of code covered by unit tests.

### pry-byebug
It extends the functionality of the Pry and Byebug gems.

### Brakeman
Static security scanner that identifies vulnerabilities in Rails applications.

### Fog
Upload and store your files on external servers like Amazon’s S3 or Google’s Cloud Storage.
