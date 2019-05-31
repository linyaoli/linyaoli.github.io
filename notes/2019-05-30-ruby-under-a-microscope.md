### Interesting stuffs I read in
# Ruby Under a Microscope
This book explains some fundamental designs and methods beneath ruby. You will need basic knowledge of low-level languages(C and Lisp), data structures(esp. treesand hash), algorithms(e.g. sorting) and ideas behind complier.

### Modules Are Classes
Internally Ruby implements modules as classes. When you create a module, Ruby creates another RClass/rb_classext_struct structure pair, just as it would for a new class.

#### So what happens when you include a module Foo in class Bar?
Ruby creates a copy of RClass for the Foo and uses it as Bar's superclass. Therefore also, same methods in Foo will be overriden by ones in Bar. If Bar has a superclass Ran at first, the inheritance of methods will be Ran->Foo->Bar.(Basically Foo will be insert between Bar and its original superclass)

> Essentially, there is no difference between including a module and specifying a superclass.

#### Included modules are ordered
Considering the following code:
```
class Bar
  include Fooa
  include Foob
end
```
Methods in Foob always override methods in Fooa because it is included afterwards.

> Blocks are Ruby's implementation of closures.

### Garbage Collection
#### The Free List
A free list is a one-way linked list of memory blocks, each represents a small piece of memory that can be used for creating new objects. As the program runs, it cretes new objects and eventually MRI uses up blocks in the list; then GC will stop the program,identifies objects that your code is no longer using, and reclaims memory for new objects. If no unused objects are found, Ruby will ask operating system for more memory, or throw OOM if OS is out of memory as well.

#### Sweeping
During sweeping and marking, MRI stops executing your code. Beginning with 1.9.3, Ruby uses Lazy Sweeping. So instead of sweep all objects, lazy sweeping only sweeps enough objects back to the free list to create a few new objects and allow your program to continue, thus reducing the amount of time required to sweep, as well as the amount of time which stops the program's execution.

> You can manually start a garbage collection by run GC.start

There are some other techniques of GC used in JRuby and Rubinius, including the Garden of Eden, which is used in JVM.

# WIP ...
