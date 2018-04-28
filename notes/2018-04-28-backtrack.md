# Under the hood of backtracks

Algorithm questions of permutation and powerset can be complicated, but essentially they share similar patterns.

permutations:

```python
  for j in xrange(i,n):
    swap(a[i],a[j])
    helper(i+1)
    swap(a[i],a[j])
```

subsets/powersets:

```python
  for j in xrange(i,n):
     push(a[j])
     helper(j+1)
     pop()
```

See?
