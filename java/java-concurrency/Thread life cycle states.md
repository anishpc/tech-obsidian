## Thread life cycle states
- The states are expressed via the `Thread.state` enumeration
- Different lifecycle states are : 
	- NEW
	- RUNNABLE
	- BLOCKED
	- WAITING
	- TIMED_WAITING
	- TERMINATED

![thread_lifecycle_states.png](thread_lifecycle_states.png)

### NEW state
- is in NEW state if it's created but NOT started
- the thread constructor creates threads in NEW state & remains there till the `start` method is invoked
```java
public class NewThread {
  public void newThread() {  
    Thread t = new Thread(() -> {});  
    System.out.println("NewThread: " + t.getState()); // NEW  
  }  
}
NewThread nt = new NewThread();  
nt.newThread();
```

### RUNNABLE state
- The transition from NEW to RUNNABLE is obtained by calling the `start` method
- In this state, the thread can be running or read to run. When it's ready to run, it's waiting for the JVM thread-scheduler to allocate the needed resources and time to run it. As soon as the processor is available, the thread-scheduler will run the thread
```java
public class RunnableThread {
  public void runnableThread() {  
    Thread t = new Thread(() -> {});  
    t.start();  
    System.out.println("RunnableThread : " + t.getState());  // RUNNABLE  
   }  
}   
RunnableThread rt = new RunnableThread();  
rt.runnableThread();
```

### BLOCKED state
- When a thread is trying to execute IO tasks or synchronized blocks, it may enter into the BLOCKED state. If thread t1 tries to enter a synchronized block which is locked by t2 then thread t1 is kept in BLOCKED state till it can acquire the lock

### WAITING state
- A thread t1 that waits, without a timeout period, for another thread t2 to finish is in the WAITING state
- This can be seen by a thread waiting for other thread(s) using the `join` method

### TIMED_WAITING state
- A thread t1 that waits for an explicit period of time for another thread t2 to finish is in the TIMED_WAITING state
- If a thread is sleeping using the `sleep` method, then the thread would be in the TIMED_WAITING state

### TERMINATED state
- A thread that successfully finishes its job or is abnormally interrupted is in the TERMINATED state