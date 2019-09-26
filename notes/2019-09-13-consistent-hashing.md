## Consistent Hashing
I'll explain it as naive as possible.

#### Shortcome of normal distributed hashing
When one of the machines is down, any data being hashed into it afterwards will be lost. This is the major problem that consistent hashing is gonna solve.

#### Why consistent hashing can handle failures
To illustrate this, first we will need a circular cache model.
```
     a ------------> b -----------> c
     ^                              |
     |                              |
     |                              |
     |                              v
     f <------------ e <----------- d
```
- Let's say this circle is split into 6 equal spans, and all random incoming data will be evenly distributed into them (based on the hash result, of course). 
- For each span, all data inside it belong to the next closest machine(a to f).
- So what happens when machine `b` failed? In normal hash model, data inside `b` and all data written into it afterwards will be lost. e.g.
```
m = hash(data) mod 6
if m = 1
 -> data is written into b and lost 
```
- We can just relocate data which should be in machine `b` into `c`, right? But there is another problem, the load of machine `c` could suddenly become very high. So the question is, can we evenly distribute data from `b` into all other machines?
- Of course we can :) 
First we need to split each actual machine into, let's say, 3 virtual machines, they are evenly distributed along the circle.
```
   a1 ---> b1 ---> c1 ---> d1 ---> e1 ----> f1 ----> a2 ---> b2
                                                             |
                                                             V
                                                             c2
                                                             |
                                                             V
                                                             d2
                                                             |
                                                             V
                                          ....          <--  e2
                                                            
```
So what happens when the actual machine `b` is removed? All the virtual `b` child machines are removed, while all other virtual machines are still evenly distributed. Therefore each machine will still have the same traffic load.

- How do we achieve this? Simple, we just need to use two hash keys. e.g.
```
m1 = hash1(data) mod 6
m2 - hash2(data) mod 3
if m1 = 1, and m2 = 2, it means data will be put into b3.
```
That's it.

#### Real-life application
Memcached uses consistent hashing to save cached data.

For more details, you should definitely check the [Wiki page](https://en.wikipedia.org/wiki/Consistent_hashing). 
