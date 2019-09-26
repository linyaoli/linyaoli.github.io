## Ruby Interview Questions
1. `map` and `each`
`each` and `map` both operate on every element, but in addition `map` applies the result to the original element. 
2. `block`, `lambda` and `Proc`
- `lambda` and `Proc` are objects, `block` is not.
- `lambda` checks the number of arguments, `proc` does not.
- After return, `proc` will also return the method, while `lambda` does not.
3. `alias` and `alias_method`
- `alias_method` can only create for methods.
```ruby
alias_method :old_one, :new_one
```
4. `yield self`
Typical usage is `yield self if block_given?`, which means if a block is given, then call it and use `self` as the argument.
5. `scope`
6. `ActiveSupport::Concern`
7. `alias_method_chain`
8. Ruby objects
```
                   +------------+
                   | BasicObject|
                   +------------+
                        ^
                        |superclass
                        |
                   +----+-----+   superclass-----------+
                   |  Object  | ^----------|   Module  |
                   +----+-----+------+     +-+---------+
                        ^            |       ^
                        |superclass  |       |Superclass
                        |            |       |
+---------+           +-+-------+    +---> +-+-------+
| 'hello' |  class    | String  | class    | Class   |
|         +---------->+         +--------->+         +------>
+---------+           +---------+          +----+----+      |
                                                ^           |
                                                |  class    |
                                                <-----------+
```
9. `include`, `extend`, `load` and `require`
- `include` adds normal methods into current class.
- `extend` adds class methods.
- `require` and `load` both load libraries, but `load` can reload, and needs to specify the path of the lib file.
10. Metaprogramming
- `send`
- `eval`
- `respond_to?`
