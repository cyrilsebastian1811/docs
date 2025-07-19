# JVM

## Memory Architecture

<figure markdown="span">
    ![JVM Memory Architecture](../img/memory-architecture.png)
</figure>

### Stack

- Region of memory used for managing method execution, local variables, and method call hierarchy.
- Each thread has its own independent stack.
- Inside RAM, managed by the JVM/OS.
- Each method call creates a stack frame. That frame stores:
    - Method parameters: e.g., `int x` in `void foo(int x)`
    - Local variables: Any variable declared in the method.
    - Reference variables: e.g., `String s = "Hi";`
    - Return address: JVM uses this to resume after method finishes
- To configure per-thread stack size use `-Xss` JVM flag.
- Total threads can be limited by OS configurations.


### Heap

- The main memory area used for dynamic memory allocation.
- Stores all objects, arrays, and class instances.
- Memory is automatically managed by the Garbage Collector (GC).
- ==The memory is shared among all threads in the JVM.==
- ==Stack variables may reference objects stored in Heap.==
- To configure use `-Xms` and `-Xmx` JVM flags.

### String Pool

- Special memory region in the heap used to store interned string literals.
- Two string variables having the same literal content, reference the same object in the pool.

```java
String s1 = "hello"; // goes to pool
String s2 = "hello"; // reuses from pool

String s3 = new String("hello");
String s4 = s3.intern(); // s4 refers to pooled "hello"
```


### MetaSpace/Method Area

- Method Area: java 7 and earlier.
- MetaSpace: java 8+. configured by `-XX:MaxMetaspaceSize`.
- memory outside the heap.
- The Method Area is a part of the JVM memory where class-level information is stored.
- One per JVM instance.
- Shared among all threads.
- Stores:
    - Class metadata: Class name, modifiers, superclass, interfaces.
    - Field metadata: Field names, types, modifiers. 
    - Method metadata: Method names, signatures, return types, modifiers.
    - Static fields: Shared fields (with values) marked `static`.
    - Compiled code.


### PC Register

The PC (Program Counter) register is a per-thread register that holds the address (or offset) of the next bytecode instruction to execute for that thread. ==Think of it like a bookmark in a book for each reader (thread).==

- Each thread has its own PC register, isolated from others.
- Stores address of current or next instruction in the method being executed.
- Every JVM instruction updates the PC register.
- Very small (just needs to hold a number/index into method bytecode).
- The JVM, not the garbage collector, manages it
