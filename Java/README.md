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

## Best Practices
1. Use meaningful variable and method names
2. Follow SOLID principles
3. Write clean, maintainable code
4. Handle exceptions appropriately
5. Use Java 8+ features where applicable
6. Write unit tests for your code
7. Use proper logging instead of System.out.println
8. Close resources properly (try-with-resources)

## Resources
- Oracle Java Documentation
- Effective Java by Joshua Bloch
- Java Concurrency in Practice
- Head First Design Patterns
