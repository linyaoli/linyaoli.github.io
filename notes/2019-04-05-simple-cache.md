# Simple Cache!
```ruby
    class Cache
      def initialize
        @data = {} # of the form {key => [value, expires_at or nil]}
      end

      def read(name, options = nil)
        value, expires_at = *@data[name]
        if value && (expires_at.blank? || expires_at > Time.now)
          value
        else
          nil
        end
      end

      def delete(name)
        @data.delete(name)
      end

      def write(name, value, options = nil)
        expires_at = if options && options.respond_to?(:[]) && options[:expires_in]
          Time.now + options.delete(:expires_in)
        else
          nil
        end
        value.freeze.tap do |val|
          @data[name] = [value, expires_at]
        end
      end

      def clear
        @data.clear
      end
    end
```
