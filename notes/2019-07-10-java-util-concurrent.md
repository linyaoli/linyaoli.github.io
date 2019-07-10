# Notes on java.util.concurrent
### Executor
Executor is an interface that represents an object that executes provided tasks.

```
public class Invoker implements Executor {
    @Override
    public void execute(Runnable r) {
        r.run();
    }
}

public void execute() {
    Executor executor = new Invoker();
    executor.execute( () -> {
        // task to be performed
    });
}
```

### ExecutorService
ExecutorService is a complete solution for asynchronous processing. It manages an in-memory queue and schedules submitted tasks based on thread availability.

```
public class Task implements Runnable {
    @Override
    public void run() {
        // task details
    }
}

// newSingleThreadExecutor(ThreadFactory threadFactory) to create a single-threaded instance.
ExecutorService executor = Executors.newFixedThreadPool(10);
```

```
public void execute() {
    executor.submit(new Task());
}
```
or
```
executor.submit(() -> {
    new Task();
});
```

### ScheduledExecutorService
ScheduledExecutorService is a similar interface to ExecutorService, but it can perform tasks periodically.
```
public void execute() {
    ScheduledExecutorService executorService
      = Executors.newSingleThreadScheduledExecutor();

    Future<String> future = executorService.schedule(() -> {
        // ...
        return "Hello world";
    }, 1, TimeUnit.SECONDS);

    ScheduledFuture<?> scheduledFuture = executorService.schedule(() -> {
        // ...
    }, 1, TimeUnit.SECONDS);

    executorService.shutdown();
}

```

### Future
Future is used to represent the result of an asynchronous operation. It comes with methods for checking if the asynchronous operation is completed or not, getting the computed result, etc.

```
public void invoke() {
    ExecutorService executorService = Executors.newFixedThreadPool(10);

    Future<String> future = executorService.submit(() -> {
        // ...
        Thread.sleep(10000l);
        return "Hello world";
    });
}
```

Then it can be used as:

```
if (future.isDone() && !future.isCancelled()) {
    try {
        str = future.get();
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}

// timeouts
try {
    future.get(10, TimeUnit.SECONDS);
} catch (InterruptedException | ExecutionException | TimeoutException e) {
    e.printStackTrace();
}
```

### CountDownLatch
CountDownLatch (introduced in JDK 5) is a utility class which blocks a set of threads until some operation completes.

### CyclicBarrier
CyclicBarrier works almost same as CountDownLatch except that we can reuse it.

### Semaphore
The Semaphore is used for blocking thread level access to some part of the physical or logical resource. A semaphore contains a set of permits; whenever a thread tries to enter the critical section, it needs to check the semaphore if a permit is available or not.

```
static Semaphore semaphore = new Semaphore(10);

public void execute() throws InterruptedException {

    LOG.info("Available permit : " + semaphore.availablePermits());
    LOG.info("Number of threads waiting to acquire: " +
      semaphore.getQueueLength());

    if (semaphore.tryAcquire()) {
        semaphore.acquire();
        // ...
        semaphore.release();
    }

}
```

### ThreadFactory
ThreadFactory acts as a thread (non-existing) pool which creates a new thread on demand.

```
public class BaeldungThreadFactory implements ThreadFactory {
    private int threadId;
    private String name;

    public BaeldungThreadFactory(String name) {
        threadId = 1;
        this.name = name;
    }

    @Override
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r, name + "-Thread_" + threadId);
        LOG.info("created new thread with id : " + threadId +
            " and name : " + t.getName());
        threadId++;
        return t;
    }
}
```

Then

```
BaeldungThreadFactory factory = new BaeldungThreadFactory(
    "BaeldungThreadFactory");
for (int i = 0; i < 10; i++) {
    Thread t = factory.newThread(new Task());
    t.start();
}
```

### BlockingQueue
This is designed to implement producer-consumer pattern.

- Unbounded queue
```
BlockingQueue<String> blockingQueue = new LinkedBlockingDeque<>();
```

- Bounded queue
```
BlockingQueue<String> blockingQueue = new LinkedBlockingDeque<>(10);
```

- Adding elements
  - `add()`: return true if successful, otherwise throw `IllegalStateException`
  - `put()`: wait for a free slot if necessary
  - `offer()`: return true if successful, otherwise false.

- Retrieving elements
  - `take()`: retrieve and removes the head of queue, blocks and waits for an element to be available if queue is empty.
  - `poll(long timeout, TimeUnit unit)`: waiting up until timeout, returns `null` if timeout.

### DelayQueue
DelayQueue is an infinite-size blocking queue of elements where an element can only be pulled if its expiration time (known as user defined delay) is completed.

### Locks
Lock is a utility for blocking other threads from accessing a certain segment of code, apart from the thread that’s executing it currently.
difference between a Lock and a Synchronized block is:
- synchronized block is fully contained in a method; however, we can have Lock API’s lock() and unlock() operation in separate methods.
- A synchronized block doesn’t support the fairness, any thread can acquire the lock once released, no preference can be specified.
- A thread gets blocked if it can’t get an access to the synchronized block. The Lock API provides tryLock() method. The thread acquires lock only if it’s available and not held by any other thread. This reduces blocking time of thread waiting for the lock
- A thread which is in “waiting” state to acquire the access to synchronized block, can’t be interrupted. The Lock API provides a method lockInterruptibly() which can be used to interrupt the thread when it’s waiting for the lock
```
Lock lock = ...;
lock.lock();
try {
    // access to the shared resource
} finally {
    lock.unlock();
}
```

### Phaser
Phaser is a more flexible solution than CyclicBarrier and CountDownLatch – used to act as a reusable barrier on which the dynamic number of threads need to wait before continuing execution. We can coordinate multiple phases of execution, reusing a Phaser instance for each program phase.

## Some other notes
### Runnable vs. Callable
The Callable interface is similar to Runnable, in that both are designed for classes whose instances are potentially executed by another thread. A Runnable, however, does not return a result and cannot throw a checked exception.

```
public class EventLoggingTask implements Runnable {
  private Logger logger
    = LoggerFactory.getLogger(EventLoggingTask.class);

  @Override
    public void run() {
      logger.info("Message");
    }
}
```

- The Callable interface is a generic interface containing a single call() method – which returns a generic value V:

```
public class FactorialTask implements Callable<Integer> {
    int number;

    // standard constructors

    public Integer call() throws InvalidParamaterException {
        int fact = 1;
        // ...
        for(int count = number; count > 1; count--) {
            fact = fact * count;
        }

        return fact;
    }
}
```

### ConcurrentMap<K, V>
A Map providing thread safety and atomicity guarantees.

### ConcurrentLinkedDeque<E>
An unbounded concurrent deque based on linked nodes.

### ConcurrentLinkedQueue<E>
An unbounded thread-safe queue based on linked nodes.

### CopyOnWriteArrayList<E>
A thread-safe variant of ArrayList in which all mutative operations (add, set, and so on) are implemented by making a fresh copy of the underlying array.

### PriorityBlockingQueue<E>
An unbounded blocking queue that uses the same ordering rules as class PriorityQueue and supplies blocking retrieval operations.

