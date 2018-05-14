# Some ruby tricks
### Create a hash from a list of values
```
Hash["k1", "v1", "k2", "v2"]
#=> {"k1"=>"v1", "k2"=>"v2"}
```
### Lambda literal
```
a = -> (v) { v + 1 }
a.call(2)
#=> 3
```

### double start \*\*
```
def my_method(a, *b, **c)
end
```
`a`, normal argument;
`b`, takes all arguments after the first one in an array;
`c`, takes all arguments after the first one in a hash. in this case all arguments have been captured by `b` so `c` will be empty. 

### ||=
`||=` is equivalent to the following
```
a || a = b
```
not
```
a = a || b
```

for example,
```
def total
  @total ||= nums.sum
end
```

Calling `total` to get the value now will only be calculated at the first time.

### Mandatory hash parameters
```
def my_method(a:, b:, c: 'default')
  return a, b, c
end
```

### Generate array of alphabet or numbers
```
('a'..'z').to_a

(1..10).to_a
```

### tap
`tap` yields the calling object to the block and returns it.

```
def my_method
  u = User.new
  u.a = 1
  u.b = 1
  o
end
```
```
def my_method
  User.new.tap do |u|
    u.a = 1
    u.b = 1
  end
end
```

### Array#join
```
['hello', 'world'].join(",")
%w{hello world} * ","
```

### format decimal value
```
m = 1.2
"%.2f" % m
```

### quick mass assignment
```
def my_method(*args)
  a, b, c, d = args
end
```

### Use ranges instead of complex comparison for numbers
```
year = 1972
puts  case year
        when 1970..1979: "Seventies"
        when 1980..1989: "Eighties"
        when 1990..1999: "Nineties"
      end
```

### Use enumerations to cut down repetitive code
```
%w{rubygems daemons eventmachine}.each { |x| require x }
```

### Ternary operators
```
puts x == 10 ? "x is ten" : "x is not ten"
```
nested:
```
qty == 0 ? 'none' : qty == 1 ? 'one' : 'many'
```

### Each vs map
```
users.each do {|user| user_ids << user.id}
users.map {|user| user.id}
users.map(&:id)
```

### conditional assignment
```
result = if foo
            'foo'
         elsif bar
            'bar'
         else
            'foobar'
         end
```

### sum of array
```
nums.inject(0, :+)
```

### Array of strings or symbols
```
%w(Michael Jonh Mary Stuart)
=> ["Michael", "Jonh", "Mary", "Stuart"]

%i(Michael Jonh Mary Stuart)
=> [:Michael, :Jonh, :Mary, :Stuart]
```

### Use Fixnum#times
```
Rather than

for i in 1..10
  puts "My iteration #{i}"
end

Use

10.times do |i|
  puts "My iteration #{i + 1}"
end
```

### at\_exit kernel method
```
def do_at_exit(str1)
  at_exit { print str1 }
end
at_exit { puts "cruel world" }
do_at_exit("goodbye ")
exit

#=> "goodbye cruel world"
```

### Struct without assignment
```
Struct.new("Name", :first, :last) do
  def full
    "#{first} #{last}"
  end
end

franzejr = Struct::Name.new("Franze", "Jr")
p franzejr.full

# Result:
# "Franze Jr"
```

### Unused variable format
```
[
    ['Someone', 41, 'another field'],
    ['Someone2', 42, 'another field2'],
    ['Someone3', 43, 'another field3']
  ].each do |name,_,_|
    p name
  end

# Result:
# "Someone"
# "Someone2"
# "Someone3"
```

### add uniq value to array
```
bad
array << 2
array << 4

# Result:
# [1, 2, 3, 2, 4]

p array.uniq

# Result:
# [1, 2, 3, 4]
good (but need refactor)
array = array | [2]
array = array | [4]

p array

# Result:
# [1, 2, 3, 4]
good
array |= [2, 4]

p array

# Result:
# [1, 2, 3, 4]
```

