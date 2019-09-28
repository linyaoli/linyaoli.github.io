### The Josephus Problem
#### Statement
Let's say there's a group of `n` sailors, starting from `0`, for each unfortunate who counts at `m-1`, he will be kicked out and thrown into sea. This will continue until only one sailor remains.
Find out who is the lucky one.
#### A Naive Solution
For a small amount of sailors, we shall just mock the whole process in the cycle until only one remains, which is fine. The time complexity is `O(mn)`.

#### With Memoization
A better solution is to utilize memoization, which is essentially dynamic programming.
1. Imagine when the first sailor `m-1` got kicked, the rest `n-1` will form a new circle.
2. The next process will start with the sailor next to him, who is `k = m mod n`. (`k`, `k+1`, `k+2`, ... `n-2`, `n-1`, `0`, `1`, ... `k-2`)
3. We can convert the above numbers into `k -> 0`, `k+1 -> 1`, ... `k-2 -> n-2`. So what do we have here? A new problem, which is a group of `n-1` sailors!

Does this look familiar to you? It is a very typical pattern of memoization. Therefore we have:
```
if i == 1
  f[1] = 0

if i > 1
  f[i] = (f[i - 1] + m) mod i
```

That's it!
