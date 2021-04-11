## Locking
A thread can achieve locks at the object level or at the class level

### Object level locking
- Locking at object level can be achieved by marking a non-`static` block of code or non-`static` method (the lock object for that method's object) with `synchronized`

### Class level locking
- In order to protect `static` data, locking at the class level can be achieved by marking a `static` method/block or acquiring a lock on the `.class` reference with `synchronized`
```java
// synchronized static method
public class ClassCll {  
  public synchronized static void methodCll() {  
    ...  
  }  
}
//== Synchronized Block on .class
public class ClassCll { 
  public void method() {  
    synchronized(ClassCll.class) {  
      ...  
    }  
  }  
}
// Synchronized on some other static object
public class ClassCll {   
  private final static Object aLock = new Object();   
  public void method() {  
    synchronized(aLock) {  
      ...  
    }  
  }  
}
```

## Executions
- Two threads CAN execute concurrently a `synchronized static` method and a non-`static synchronized` method of the same class. This works because the threads acquire locks on different objects
- Two threads CANNOT concurrently execute two different `synchronized static` methods (or the same `synchronized static` method) of the same class. This does not work because the first thread acquires a class-level lock.