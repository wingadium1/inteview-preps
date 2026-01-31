# Java Interview Preparation

## Core Java Concepts

### 1. Object-Oriented Programming (OOP)
- **Encapsulation**: Bundling data and methods that operate on that data
- **Inheritance**: Creating new classes based on existing classes
- **Polymorphism**: Objects taking many forms (compile-time and runtime)
- **Abstraction**: Hiding complex implementation details

### 2. Java Fundamentals
- **Data Types**: Primitive types (int, long, double, boolean, etc.) vs Reference types
- **Strings**: Immutable, String pool, StringBuilder vs StringBuffer
- **Arrays**: Fixed size, indexing, multi-dimensional arrays
- **Wrapper Classes**: Integer, Long, Double, Boolean, etc.

### 3. Collections Framework
- **List**: ArrayList, LinkedList, Vector
- **Set**: HashSet, LinkedHashSet, TreeSet
- **Map**: HashMap, LinkedHashMap, TreeMap, Hashtable
- **Queue**: PriorityQueue, ArrayDeque
- **Concurrent Collections**: ConcurrentHashMap, CopyOnWriteArrayList

#### Thread-Safe vs Non-Thread-Safe Data Structures

**Non-Thread-Safe Collections (Not Synchronized)**
- **ArrayList**: Fast, not synchronized. Use for single-threaded scenarios or when external synchronization is provided
- **LinkedList**: Not synchronized. Better for frequent insertions/deletions
- **HashMap**: Not synchronized. Fastest map implementation for single-threaded use
- **HashSet**: Not synchronized. Built on HashMap
- **TreeMap**: Not synchronized. Sorted map based on Red-Black tree
- **TreeSet**: Not synchronized. Sorted set
- **PriorityQueue**: Not synchronized. Heap-based priority queue
- **ArrayDeque**: Not synchronized. Resizable array-based double-ended queue

**Thread-Safe Collections (Synchronized)**

*Legacy Synchronized Collections (Less Efficient):*
- **Vector**: Synchronized version of ArrayList. Legacy class, prefer CopyOnWriteArrayList
- **Hashtable**: Synchronized version of HashMap. Legacy class, prefer ConcurrentHashMap
- **Stack**: Synchronized, extends Vector. Legacy class, prefer Deque implementations

*Modern Concurrent Collections (Highly Efficient):*
- **ConcurrentHashMap**: 
  - Segment-level locking (Java 7) / CAS operations (Java 8+)
  - High concurrency, better performance than Hashtable
  - Does not allow null keys or values
  - Weakly consistent iterators (don't throw ConcurrentModificationException)
  
- **CopyOnWriteArrayList**: 
  - Creates a new copy on every write operation
  - Ideal for read-heavy scenarios with infrequent writes
  - Iterators are snapshot-based, never throw ConcurrentModificationException
  
- **CopyOnWriteArraySet**: 
  - Based on CopyOnWriteArrayList
  - Same characteristics as CopyOnWriteArrayList
  
- **ConcurrentLinkedQueue**: 
  - Non-blocking, thread-safe queue
  - Based on CAS (Compare-And-Swap) operations
  - Efficient for high-concurrency scenarios
  
- **ConcurrentLinkedDeque**: 
  - Non-blocking, thread-safe double-ended queue
  - Similar to ConcurrentLinkedQueue but supports both ends
  
- **BlockingQueue Implementations**:
  - **ArrayBlockingQueue**: Bounded blocking queue backed by array
  - **LinkedBlockingQueue**: Optionally bounded blocking queue backed by linked nodes
  - **PriorityBlockingQueue**: Unbounded blocking queue with priority ordering
  - **SynchronousQueue**: Blocking queue with no internal capacity
  - **DelayQueue**: Unbounded blocking queue of delayed elements

**Synchronized Wrappers**:
```java
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Set<String> syncSet = Collections.synchronizedSet(new HashSet<>());
Map<String, String> syncMap = Collections.synchronizedMap(new HashMap<>());
```
Note: Synchronized wrappers provide thread safety but with less efficiency than concurrent collections

**Comparison Table**:

| Use Case | Non-Thread-Safe | Thread-Safe Alternative |
|----------|----------------|------------------------|
| General List | ArrayList | CopyOnWriteArrayList or Collections.synchronizedList() |
| General Map | HashMap | ConcurrentHashMap |
| General Set | HashSet | CopyOnWriteArraySet or Collections.synchronizedSet() |
| Sorted Map | TreeMap | ConcurrentSkipListMap |
| Sorted Set | TreeSet | ConcurrentSkipListSet |
| Queue | ArrayDeque | ConcurrentLinkedQueue |
| Deque | ArrayDeque | ConcurrentLinkedDeque |
| Priority Queue | PriorityQueue | PriorityBlockingQueue |

**When to Use Thread-Safe Collections**:
1. Multiple threads accessing the same collection
2. At least one thread modifying the collection
3. Need consistency without external synchronization
4. Producer-consumer patterns (use BlockingQueue)
5. Cache implementations (use ConcurrentHashMap)

**When to Use Non-Thread-Safe Collections**:
1. Single-threaded applications
2. Collections local to a method or thread
3. External synchronization is already in place
4. Maximum performance is needed and thread safety isn't required

### 4. Exception Handling
- **Checked vs Unchecked Exceptions**
- **try-catch-finally blocks**
- **Custom exceptions**
- **Best practices**: Specific exceptions, proper cleanup

### 5. Multithreading & Concurrency
- **Thread Creation**: Extending Thread class vs Implementing Runnable
- **Synchronization**: synchronized keyword, locks
- **Thread States**: New, Runnable, Blocked, Waiting, Timed Waiting, Terminated
- **Executor Framework**: ThreadPoolExecutor, ExecutorService
- **Concurrent Utilities**: CountDownLatch, CyclicBarrier, Semaphore
- **volatile keyword**
- **deadlock, livelock, starvation**

### 6. Java 8+ Features
- **Lambda Expressions**: Functional programming constructs
- **Stream API**: filter, map, reduce, collect
- **Optional**: Avoiding null pointer exceptions
- **Default Methods**: Interface evolution
- **Method References**: :: operator
- **Functional Interfaces**: Predicate, Function, Consumer, Supplier

### 7. Java Memory Management
- **Heap vs Stack**
- **Garbage Collection**: Types (Serial, Parallel, CMS, G1)
- **Memory Leaks**: Common causes and prevention
- **Generational GC**: Young generation, Old generation

### 8. Design Patterns
- **Creational**: Singleton, Factory, Builder, Prototype
- **Structural**: Adapter, Decorator, Proxy, Facade
- **Behavioral**: Observer, Strategy, Template Method, Command

## Common Interview Questions

### Basic Questions
1. What is the difference between JDK, JRE, and JVM?
2. Explain the difference between == and equals()
3. What is the difference between String, StringBuilder, and StringBuffer?
4. What are access modifiers in Java?
5. Explain method overloading vs method overriding

### Intermediate Questions
1. How does HashMap work internally?
2. Explain the difference between ArrayList and LinkedList
3. What is the difference between abstract class and interface?
4. How do you handle exceptions in Java?
5. Explain the Java memory model

### Advanced Questions
1. How does ConcurrentHashMap work internally?
2. Explain the happens-before relationship in Java
3. What are the different types of references in Java?
4. How does garbage collection work in Java?
5. Explain the fork-join framework
6. What is the difference between synchronized collections and concurrent collections?
7. When would you use CopyOnWriteArrayList vs Collections.synchronizedList()?
8. Explain the internal working of ConcurrentHashMap in Java 8+
9. What are the differences between BlockingQueue implementations?
10. Why is ConcurrentHashMap preferred over Hashtable?

## Code Examples

### Example 1: Stream API Usage
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Filter even numbers, double them, and collect
List<Integer> result = numbers.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * 2)
    .collect(Collectors.toList());
```

### Example 2: Custom Thread Pool
```java
ExecutorService executor = Executors.newFixedThreadPool(5);

for (int i = 0; i < 10; i++) {
    executor.submit(() -> {
        System.out.println("Task executed by: " + 
            Thread.currentThread().getName());
    });
}

executor.shutdown();
```

### Example 3: Singleton Pattern (Thread-Safe)
```java
public class Singleton {
    private static volatile Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

### Example 4: Thread-Safe Collections Usage
```java
// ConcurrentHashMap - High concurrency map
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put("key1", 1);
concurrentMap.putIfAbsent("key2", 2);
concurrentMap.computeIfAbsent("key3", k -> 3);

// CopyOnWriteArrayList - Read-heavy scenarios
CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
cowList.add("item1");
// Safe iteration even during concurrent modifications
for (String item : cowList) {
    System.out.println(item);
}

// BlockingQueue - Producer-Consumer pattern
BlockingQueue<String> queue = new LinkedBlockingQueue<>(100);

// Producer thread
new Thread(() -> {
    try {
        queue.put("message"); // Blocks if queue is full
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}).start();

// Consumer thread
new Thread(() -> {
    try {
        String message = queue.take(); // Blocks if queue is empty
        System.out.println("Received: " + message);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}).start();
```

### Example 5: Synchronized Wrapper vs Concurrent Collection
```java
// Synchronized wrapper - Entire collection locked on each operation
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
synchronized (syncList) {
    // Need manual synchronization for iteration
    for (String item : syncList) {
        System.out.println(item);
    }
}

// Concurrent collection - Fine-grained locking, better performance
CopyOnWriteArrayList<String> concurrentList = new CopyOnWriteArrayList<>();
// No manual synchronization needed for iteration
for (String item : concurrentList) {
    System.out.println(item);
}

// HashMap vs ConcurrentHashMap
Map<String, String> regularMap = new HashMap<>();
// Not thread-safe - can cause data corruption in multi-threaded scenarios

ConcurrentHashMap<String, String> threadSafeMap = new ConcurrentHashMap<>();
// Thread-safe - multiple threads can read/write safely
threadSafeMap.forEach((k, v) -> System.out.println(k + ": " + v)); // For demo only
```

## Best Practices
1. Use meaningful variable and method names
2. Follow SOLID principles
3. Write clean, maintainable code
4. Handle exceptions appropriately
5. Use Java 8+ features where applicable
6. Write unit tests for your code
7. Use proper logging instead of System.out.println
8. Close resources properly (try-with-resources)
9. **Choose the right collection for thread safety**:
   - Use non-thread-safe collections for single-threaded or method-local scenarios
   - Use ConcurrentHashMap instead of Hashtable or synchronized HashMap
   - Use CopyOnWriteArrayList for read-heavy, write-rare scenarios
   - Use BlockingQueue for producer-consumer patterns
   - Prefer concurrent collections over synchronized wrappers for better performance
10. **Avoid common concurrency pitfalls**:
    - Don't iterate over synchronized collections without external synchronization
    - Be aware that ConcurrentHashMap doesn't allow null keys/values
    - Use atomic operations like putIfAbsent() instead of check-then-act patterns
    - Consider using immutable collections when possible for thread safety

## Resources
- Oracle Java Documentation
- Effective Java by Joshua Bloch
- Java Concurrency in Practice
- Head First Design Patterns
