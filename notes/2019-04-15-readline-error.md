# Ruby's readline error

Last time after I upgraded several libraries, ruby command line mode couldn't open any more, the error is like below:
```
    Sorry, you can't use byebug without Readline. To solve this, you need to
    rebuild Ruby with Readline support. If using Ubuntu, try `sudo apt-get
    install libreadline-dev` and then reinstall your Ruby.
/Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require': dlopen(/Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/x86_64-darwin17/readline.bundle, 9): Library not loaded: /usr/local/opt/readline/lib/libreadline.7.dylib (LoadError)
  Referenced from: /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/x86_64-darwin17/readline.bundle
  Reason: image not found - /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/x86_64-darwin17/readline.bundle
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/byebug-10.0.2/lib/byebug/history.rb:4:in `<top (required)>'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/byebug-10.0.2/lib/byebug/interface.rb:4:in `<top (required)>'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/byebug-10.0.2/lib/byebug/core.rb:7:in `<top (required)>'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/pry-byebug-3.6.0/lib/byebug/processors/pry_processor.rb:1:in `<top (required)>'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/pry-byebug-3.6.0/lib/pry-byebug/pry_ext.rb:1:in `<top (required)>'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/pry-byebug-3.6.0/lib/pry-byebug/cli.rb:2:in `<top (required)>'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/pry-0.12.2/lib/pry/plugins.rb:41:in `load_cli_options'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/pry-0.12.2/lib/pry/cli.rb:37:in `block in add_plugin_options'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/pry-0.12.2/lib/pry/cli.rb:36:in `each'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/pry-0.12.2/lib/pry/cli.rb:36:in `add_plugin_options'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/pry-0.12.2/lib/pry/cli.rb:127:in `<top (required)>'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/pry-0.12.2/lib/pry.rb:121:in `<top (required)>'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:68:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:68:in `require'
	from /Users/m199/.rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/pry-0.12.2/bin/pry:4:in `<top (required)>'
	from /Users/m199/.rbenv/versions/2.3.0/bin/pry:23:in `load'
	from /Users/m199/.rbenv/versions/2.3.0/bin/pry:23:in `<main>'
  ```

  If you have checked github conversations and the given solutions don't work, please try to switch to older version of readline, with
  ```
  brew switch readline 7.0.5
  ```

  If you cannot install 7.0.5 anymore by it, try
  ```
  brew reinstall https://raw.githubusercontent.com/Homebrew/homebrew-core/98c7e5b72e309adce82dbe2aae8cb1f68ae84830/Formula/readline.rb
  ```
