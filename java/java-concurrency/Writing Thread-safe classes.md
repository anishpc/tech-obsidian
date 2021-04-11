
Following techniques for thread-safe classes
- Have no _state_ (classes with no instance and static variables)
- Have _state,_ but don't share it (for example, use instance variables via `Runnable`, `ThreadLocal`, and so on)
- Have _state,_ but an immutable _state_
- Use message-passing (for example, as Akka framework)
- Use `synchronized` blocks
- Use `volatile` variables
- Use data structures from the `java.util.concurrent` package
- Use synchronizers (for example, `CountDownLatch` and `Barrier`)
- Use locks from the `java.util.concurrent.locks` package