# Goroutines

A few information on goroutines and channels.
[Why goroutine?](https://golang.org/doc/faq#goroutines)

### What are goroutines?
Goroutines can be considered as light-weight threads. While most languages can only run smoothly on tens of threads,
Goroutines can run thousands.

A goroutine is basically just a stack with some additional context. Changing from goroutine to another means changing the stack pointer and the thread-local
variable that points to the currently running goroutine.

### Difference: goroutines vs. threads
- Creation of goroutines is cheap (appx. 2kb of stack space); threads usually use 1mb at least. This makes goroutines much less possible to cause OOM error.
- Goroutines are also cheap to destroy.
- Switching between threads costs many more register operations than goroutines.
- Goroutines communicate via channels. By design channels can prevent race conditions(because it is essentially a queue). This is an improvement from the bottom,
  that communications within programs should not use memory sharing.
- We are talking about concurrency in goroutines, not parallelism. They are different.

### Process switching
In a time-sharing system the operating systems must constantly switch the attention of CPU between these processes by recording the state of the current process(as
discussed above, `process switching` takes tens of register operations to save all states).

### 
```go
package main

import (
    "fmt"
    "sync"
    "time"
    )

func main() {
wg := &sync.WaitGroup{}
    for i := 0; i < 3; i++ {
      wg.Add(1)
        go func(i int) {
          defer wg.Done() //release wait when done
            time.Sleep(time.Second * time.Duration(i))
            fmt.Print(i)
        }(i)
    }
    wg.Wait()
      fmt.Println()
}
```
