<!-- .slide: id="Handling Resources" data-auto-animate -->

## Handling Resources

notes: The purpose of types in a programming language is to provide an abstraction over the binary data we are manipulating. They give meaning to the 1s and 0s that make up our systems and programs by defining valid states (combination of bits) our type can be in as well as the operations (instructions) that can be performed on them.<br><br>As we've explored, types are just the aggregation of their data members. These data members are known as resources and are held by an<!-- instance --> object of the type. This means it is the type's responsibility to *free* those resources and give them back to the system. C++ gives us a great deal of control when it come to freeing resources...for better or for worse.

===

<!-- .slide: data-auto-animate -->

### Resources

- Memory<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
  - Static Memory<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
  - Automatic Memory<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
  - Dynamic Memory<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
- Files<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
- Threads<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
- Locks<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
- Sockets<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->

<!-- List of different resources -->

notes: Let's first have a look at some of the resources we can obtain from the system.<br><br>

---

<!-- .slide: data-auto-animate -->

#### Memory

<!-- Diagram of different memory locations -->

notes: The most common resource we can obtain is memory. All data must be stored somewhere; even the handles to the other resource types on this list, making memory the most sought after resource on a system. In general there are three types of memory available to a program; static, automatic and dynamic, with each having slightly different semantics for allocation and deallocation.<br><br>Static memory refers memory encoded directly into the executable file for a program. Static memory is allocated when the program is being executed and loaded into memory (RAM) by the kernel. Global variables and variables marked `static` are allocated as static memory.<br><br>Automatic memory or local memory is memory allocated onto the stack of a program at runtime. It is dubbed *automatic* memory because the lifetimes of these objects are managed by system. The instructions for allocating them into a stack frame is handled by the compiler and deallocation is handled by the deallocation of a whole stack frame.<br><br>Dynamic memory is memory that must be explicitly requested from the system. The lifetime of this memory is managed by the program (you) and thus must be manually returned to the system to avoid a *memory leak*.

---

<!-- .slide: data-auto-animate -->

#### Files

<!-- Diagram of file mapped to memory -->

notes: Some resources are acquired or obtained rather than allocated because the resource is unique. Files are one such resource because each file has a unique descriptor used to locate the file. Processes and threads are able to simultaneously hold a file's descriptor however, a file is obtained when it is opened ie. the files contents are loaded into memory with it being freed once closed.

---

<!-- .slide: data-auto-animate -->

#### Threads

<!-- Diagram of multiple threads running in a process -->

Threads are are another allocated resource as it is something an OS hands out specifically due to there being a finite amount available, similar to how a system has a finite amount of memory it can allocate. Threads refer to a *thread of execution* or *an execution pathway*. By default, a process runs only one thread however it is possible for a process to have multiple threads. A thread is allocated once it has been allocated its own call stack and program counter, used to load the instructions the thread is going to execute. Threads are freed once they have terminated due to exhausting their work load and have been rejoined to the main thread ie. the spawning thread.

---

<!-- .slide: data-auto-animate -->

#### Locks

<!-- Diagram showing two threads trying to access object protected by lock with one succeeding and the other attempting and failing -->

notes: A lock (or mutex) is a bit of a combination of an allocated resource and an acquired resource (at least in C++). A lock is used to prevent multiple threads or processes from accessing shared memory at the same time, preventing a race condition. Locks are usually first allocated by the system; usually due to them being implemented using hardware and OS primitives, and later are acquired or *locked* during a critical region. A lock is freed when it is *unlocked* by the locking thread or process and deallocated when a process or scope terminates and it is removed from memory by the system (provided correct memory management is applied).

---

<!-- .slide: data-auto-animate -->

#### Sockets

<!-- Diagram of data transmitting through socket -->

notes: Sockets are a resource in a similar vein to files. A socket is usually described using *file descriptor-like* object (on Unix systems at least) meaning they can be opened to be written to or read from and closed when not in use. Unlike files, sockets have many more operations available that are used to control how the socket is used eg. binding an address to a socket or have a socket listen for incoming connections.

===

<!-- .slide: data-auto-animate -->

### RAII

<!-- What is RAII -->

---

#### Object Lifetimes

<!-- Talk about how the lifetime of members is bound to the outer scope (function or type). -->

---

#### Value, Reference and Move Semantics

---

##### Value Semantics

<!-- Copy Semantics -->

---

##### Reference Semantics

---

##### Move Semantics

===

### Constructors and Destructors

---

#### Constructors

<!-- - Syntax
  - Base
    - Special function
    - Parameters
    - Body
  - Initialiser List
- Explicit Constructor (Parameter constructor)
- Copy and Move Constructors
- Copy and Move Assignments -->

---

#### Destructors

<!-- - Syntax
- Do not throw from a constructor! -->
