## Thread pools

### Classes hierarchy
![Executor_class_hierarchy.png](Executor_class_hierarchy.png)

### Executor
- The simplest interface in the `java.util.concurrent` package to execute tasks is the `Executor`
- Executor interface has only one method `void execute(Runnable)` (no return type, no exception)
```java
public class SimpleExecutor implements Executor {  
  @Override  
  public void execute(Runnable r) {  
    (new Thread(r)).start();  
  }  
}  
SimpleExecutor se = new SimpleExecutor();  
se.execute(() -> {  
  System.out.println("Simple task executed via Executor interface");  
});
```

![executorservice_class.png](executorservice_class.png)
### ExecutorService
- Discussion on link : [ThreadPoolExecutor Javadoc](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html)
- `ExecutorService` interface extends `Executor` and it provides methods for managing lifecycle of threads.
- The full-fledged implementation of `ExecutorService` is the `ThreadPoolExecutor`
```java
new ThreadPoolExecutor(  
  int corePoolSize,  
  int maximumPoolSize,  
  long keepAliveTime,  
  TimeUnit unit,  
  BlockingQueue<Runnable> workQueue,  
  ThreadFactory threadFactory,  
  RejectedExecutionHandler handler)
```
- Constructor arguments : 
	- `corePoolSize` : the number of threads to keep in the pool even if they are idle (unless `allowCoreThreadTimeout` is set)
	- `maximumPoolSize`: the maximum number of allowed threads
	- `keepAliveTime`: when this time has elapsed, the idle threads will be removed from the pool (these are idle threads that exceed `corePoolSize`) 
		- **NOTE:** A time value of zero will cause excess threads to ***terminate immediately*** after executing tasks
	- `unit` : the time unit for the `keepAliveTime` argument
	- `workQueue`: a queue for holding the instances of `Runnable` (only the `Runnable` tasks submitted by the `execute()` method) before they are executed
	- `threadFactory` : this factory is used when the executor creates a new thread
	- `handler` : when `threadPoolExecutor` cannot execute a `Runnable` due to saturation, this is when the thread bounds and queue capacities are full (e.g. `workQueue` has fixed size and `maximumPoolSize` is set as well) then it gives the control and decision to this handler
- In order to optimize the pool size, collect the following information : 
	- number of CPUs (`Runtime.getRuntime().getAvailableProcessors()`)
	- target CPU utilization (in range [0,1])
	- wait time (w)
	- compute time (c)
> Number of threads   
  = Number of CPUs \* Target CPU utilization \* (1 + W/C)
- Note : As a rule of thumb, for compute-intensive tasks (usually small tasks), it can be a good idea to benchmark the thread pool with the number of threads equal with to number of processors or number of processors + 1 (to prevent potential pauses). For time-consuming and blocking tasks (for example, I/O), a larger pool is better since threads will not be available for scheduling at a high rate. Also, pay attention to interferences with other pools (for example, database connections pools, and socket connection pools).
```java
    BlockingQueue<Runnable> queue = new LinkedBlockingQueue<>(5);  
    final AtomicInteger counter = new AtomicInteger();  
    ThreadFactory threadFactory = (Runnable r) -> {  
      System.out.println("Creating a new Cool-Thread-"   
        + counter.incrementAndGet());  
      return new Thread(r, "Cool-Thread-" + counter.get());  
    };  
    RejectedExecutionHandler rejectedHandler  
      = (Runnable r, ThreadPoolExecutor executor) -> {  
        if (r instanceof SimpleThreadPoolExecutor) {  
          SimpleThreadPoolExecutor task=(SimpleThreadPoolExecutor) r;  
          System.out.println("Rejecting task " + task.taskId);  
        }  
    };  
    ThreadPoolExecutor executor = new ThreadPoolExecutor(10, 20, 1,  
      TimeUnit.SECONDS, queue, threadFactory, rejectedHandler);  
    for (int i = 0; i < 50; i++) {  
      executor.execute(new SimpleThreadPoolExecutor(i));  
    }  
    executor.shutdown();  
    executor.awaitTermination(  
      Integer.MAX\_VALUE, TimeUnit.MILLISECONDS);  
  }
```

### ScheduledExecutorService
- `ScheduledExecutorService` is an `ExecutorService` that can schedule tasks for execution after a given delay, or execute periodically
- While `schedule()` is used for one-shot tasks, `scheduleAtFixedRate()` and `scheduleWithFixedDelay()` are used for periodic tasks

### Executors
- defines static utility methods
#### Executors#newSingleThreadExecutor()
- This is a thread pool that manages only one thread with an unbounded queue, which only executes one task at a time
```java
public static ExecutorService newSingleThreadExecutor() {  
    return new FinalizableDelegatedExecutorService  
        (new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,  
		 new LinkedBlockingQueue<Runnable>()));  
}
// corePoolSize:1, maximumPoolSize:1, keepAliveTime:0L
```

#### Executors#newCachedThreadPool()
- Calls will reuse previously constructed threads. If no existing thread is available, a new thread will be created and added to the pool.  
- Improve performance for "*short lived asynchronous tasks*" 
- Threads that have not been used for sixty seconds are terminated and removed from  the cache. Thus, a pool that remains idle for long enough will not consume any resources.
- the core pool size is 0 and the maximum pool size is `Integer.MAX_VALUE` (this thread pool expands when demand increases and contracts when demand decreases)
```java
public static ExecutorService newCachedThreadPool() {  
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS,
      new SynchronousQueue<Runnable>());  
}
// corePoolSize:0, maximumPoolSize:MAX_VALUE, keepAliveTime:60L
```

#### Executors#newFixedThreadPool()
- creates a thread pool that reuses a fixed number of threads operating off a shared unbounded queue
- if all threads are active and a new task is submitted then it would wait in the queue until a thread is available
- if any thread terminates due to failure prior to shutdown, a new one will take its place if needed to 
```java
public static ExecutorService newFixedThreadPool(int nThreads) {  
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS 
 		new LinkedBlockingQueue<Runnable>());  
}
// corePoolSize:nThreads,maximumPoolSize:nThreads,keepAliveTime:0L
```

### Shutdown
#### Method shutdown
- Initiates an orderly shutdown in which previously submitted tasks are executed, but no new tasks will be accepted
- This method does NOT wait for previously submitted tasks to complete execution. Use `awaitTermination` to do that

### Await Termination
```java
boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException
```
- Blocks until all tasks have completed execution after a shutdown request, or the timeout occurs, or the current thread is interrupted, whichever happens first

### Shutdown Now
```java
List<Runnable> shutdownNow()
```
- Attempts to stop all actively executing tasks, halts the processing of waiting tasks, and returns a list of the tasks that were waiting for execution
	- Typical implementation will cancel via `Thread.interrupt`, so any task that fails to respond to interrupts may never terminate
- This method does NOT wait for actively executing tasks to terminate. Use `awaitTermination` for that

## Limitations of Shutdown & ShutdownNow
- There's no shutdown option in which 
	- tasks not yet started are returned to the caller but
	- tasks in progress are allowed to complete
	- Java Concurrency In Practice has an example of an `ExecutorService` which keeps cancelled tasks in a list so that those can be processed again.

## Work stealing Thread Pool
```java
public static ExecutorService newWorkStealingPool() {  
  
  return new ForkJoinPool(Runtime.getRuntime().availableProcessors(),  
    ForkJoinPool.defaultForkJoinWorkerThreadFactory,  
      null, true);  
}
```

## Pool based on tasks
### Large number of small tasks
![pools_small_tasks.png](pools_small_tasks.png)

### Small number of time consuming tasks
![pools_timeconsuming_tasks.png](pools_timeconsuming_tasks.png)