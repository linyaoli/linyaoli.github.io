### Interesting stuffs I read in
# Ruby Under a Microscope
This book explains some fundamental designs and methods beneath ruby. You will need basic knowledge of data structures(esp. trees), some algorithms and complier. I'm sure most ruby developers will like it.

### Modules Are Classes
Internally Ruby implements modules as classes. When you create a module, Ruby creates another RClass/rb_classext_struct structure pair, just as it would for a new class.

#### So what happens when you include a module Foo in class Bar?
Ruby creates a copy of RClass for the Foo and uses it as Bar's superclass. Therefore also, same methods in Foo will be overriden by ones in Bar. If Bar has a superclass Ran at first, the inheritance of methods will be Ran->Foo->Bar.(Basically Foo will be insert between Bar and its original superclass)

Essentially, there is no difference between including a module and specifying a superclass.

#### Included modules are ordered
Considering the following code:
```
class Bar
  include Fooa
  include Foob
end
```
Methods in Foob always override methods in Fooa because it is included afterwards.

# WIP ...
