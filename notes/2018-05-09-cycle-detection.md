# Cycle Detection
## Definition
The following is from Wikipedia:

For any function `f` that maps a finite set `S` to itself, and any initial value `x0` in `S`, the sequence of iterated function values
```
x0, x1 = f(x0), x2 = f(x1), ... , xi = f(x[i-1]), ...
```
must eventually use the same value twice: there must be some pair of distinct indices `i` and `j` such that `xi = xj`. Once this happens,
the sequence must continue periodically, by repeating the same sequence of values from `xi` to `x[j-1]`. Cycle detection is the problem of
finding `i` and `j`, given `f` and `x0`.

## Floyd's Tortoise and Hare
### Definition 
Given a tortoise and a hare. The hare runs twice as fast as the tortoise. If they both run in a circular race track, the hare will
always(eventually) lap the tortoise. That being said, the hare will be at the same spot as the tortoise after they start running.

### Proof of Correctness
Assume that the velocity of hare is `v1` and of tortoise is `v2`, and the starting point of them are `p` and `q` respectively. The time passed during
the race is `t`. The length of the circular track is `m`. 
When the tortoise gets lapped by the hare, for hare we have:
```
hare(t) = p + v1 * t 
```
and for the tortoise we have:

```
tortoise(t) = q + v2 * t
```

And because the tortoise is lapped, we have:
```
tortoise(t) = hare(t) mod m 
```

Therefore, 
```
p + v1 * t = q + v2 * t + k * m
```
where `k` is a numeric constant.

Because `p = q = 0` and `v1 = 2 * v2`, then:
```
v2 * t = k * m
```

If we let hare start at beginning point 0 again and advance both animals one step at a time (or the velocity of tortoise `v2`), 
then according to the above equation, they will always meet at the beginning of the circular track.
 
### What if the race track is only partly circular?
TODO

## Applications of Cycle Detection
One of the most classic implementation of Floyd's algorithm is [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/description/) problem.

### Linked List Cycle
<b>Q: Given a linked list, determine if it has a cycle in it without using extra space. </b>

Let's say we have a linked list of such:
```

 x_0 -> x_1 -> ... x_k -> x_{k+1} ................ -> x_{k+j}
                       ^                                  |
                       |                                  |
                       +.. x_{k+j+m} <- ... <- x_{k+j+1}--+

```
TODO

### Find the duplicate number
TODO
