## Producer waits for consumer to be available
- Class file is [AssembyLine.java](https://github.com/PacktPublishing/Java-Coding-Problems/blob/master/Chapter10/P202_ThreadPoolSingleThread_TransferQueue/src/modern/challenge/AssemblyLine.java)
### Queue
- `TransferQueue` is used which waits for the message to be put into the queue acting as a handoff
```java
TransferQueue<String> queue = new LinkedTransferQueue<>();
```
### Producer
- Producer uses the `tryTransfer` with a timeout to transfer the message
- If the timeout is reached without the message being consumed by the consumer then `transferred` variable has `false`
```java
boolean transferred = queue.tryTransfer(bulb, TIMEOUT_MS, TimeUnit.MILLISECONDS);
```

### Consumer
- Consumer uses the `poll` method with a timeout to consume the message
- If the timeout value is reached and there's no message to consume then the response is `null`
```java
String message = queue.poll(MAX_PROD_TIME_MS, TimeUnit.MILLISECONDS);
```


## Producer does NOT wait for consumer
### Queue
- Can rely on `ConcurrentLinkedQueue` (or `LinkedBlockingQueue`). This is an unbounded thread-safe queue based on linked nodes
```java
final Queue<String> queue = new ConcurrentLinkedQueue<>();
```
### Producer
```java
queue.offer(bulb);
```
### Consumer
```java
String bulb = queue.poll();
```