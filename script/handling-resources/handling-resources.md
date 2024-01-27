<!-- .slide: id="handling-resources" data-auto-animate -->

## Handling Resources

notes: The purpose of types in a programming language is to provide an abstraction over the binary data we are manipulating. They give meaning to the 1s and 0s that make up our systems and programs by defining valid states (combination of bits) our type can be in as well as the operations (instructions) that can be performed on them.<br><br>As we've explored, types are just the aggregation of their data members. These data members are known as resources and are held by an<!-- instance --> object of the type. This means it is the type's responsibility to *free* those resources and give them back to the system. C++ gives us a great deal of control when it come to freeing resources...for better or for worse.

===

<!-- .slide: id="handling-resources/resource-types" data-auto-animate -->

### Resource Types

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

<!-- .slide: id="handling-resources/resource-types/memory" data-auto-animate -->

#### Memory

<!-- Diagram of different memory locations -->

notes: The most common resource we can obtain is memory. All data must be stored somewhere; even the handles to the other resource types on this list, making memory the most sought after resource on a system. In general there are three types of memory available to a program; static, automatic and dynamic, with each having slightly different semantics for allocation and deallocation.<br><br>Static memory refers memory encoded directly into the executable file for a program. Static memory is allocated when the program is being executed and loaded into memory (RAM) by the kernel. Global variables and variables marked `static` are allocated as static memory.<br><br>Automatic memory or local memory is memory allocated onto the stack of a program at runtime. It is dubbed *automatic* memory because the lifetimes of these objects are managed by system. The instructions for allocating them into a stack frame is handled by the compiler and deallocation is handled by the deallocation of a whole stack frame.<br><br>Dynamic memory is memory that must be explicitly requested from the system. The lifetime of this memory is managed by the program (you) and thus must be manually returned to the system to avoid a *memory leak*. This memory often comes from a region of memory called the free store or sometimes referred to as *the heap*.

---

<!-- .slide: id="handling-resources/resource-types/files" data-auto-animate -->

#### Files

<!-- Diagram of file mapped to memory -->

notes: Some resources are acquired or obtained rather than allocated because the resource is unique. Files are one such resource because each file has a unique descriptor used to locate the file. Processes and threads are able to simultaneously hold a file's descriptor however, a file is obtained when it is opened ie. the files contents are loaded into memory with it being freed once closed.

---

<!-- .slide: id="handling-resources/resource-types/threads" data-auto-animate -->

#### Threads

<!-- Diagram of multiple threads running in a process -->

notes: Threads are are another allocated resource as it is something an OS hands out specifically due to there being a finite amount available, similar to how a system has a finite amount of memory it can allocate. Threads refer to a *thread of execution* or *an execution pathway*. By default, a process runs only one thread however it is possible for a process to have multiple threads. A thread is allocated once it has been allocated its own call stack and program counter, used to load the instructions the thread is going to execute. Threads are freed once they have terminated due to exhausting their work load and have been rejoined to the main thread ie. the spawning thread.

---

<!-- .slide: id="handling-resources/resource-types/locks" data-auto-animate -->

#### Locks

<!-- Diagram showing two threads trying to access object protected by lock with one succeeding and the other attempting and failing -->

notes: A lock (or mutex) is a bit of a combination of an allocated resource and an acquired resource (at least in C++). A lock is used to prevent multiple threads or processes from accessing shared memory at the same time, preventing a race condition. Locks are usually first allocated by the system; usually due to them being implemented using hardware and OS primitives, and later are acquired or *locked* during a critical region. A lock is freed when it is *unlocked* by the locking thread or process and deallocated when a process or scope terminates and it is removed from memory by the system (provided correct memory management is applied).

---

<!-- .slide: id="handling-resources/resource-types/sockets" data-auto-animate -->

#### Sockets

<!-- Diagram of data transmitting through socket -->

notes: Sockets are a resource in a similar vein to files. A socket is usually described using *file descriptor-like* object (on Unix systems at least) meaning they can be opened to be written to or read from and closed when not in use. Unlike files, sockets have many more operations available that are used to control how the socket is used eg. binding an address to a socket or have a socket listen for incoming connections.

---

<!-- .slide: id="handling-resources/resource-types/manual-memory-management" data-auto-animate -->

#### Manual Resource Management

notes: You might observe that all of the resources describe above have a *lifetime*, some duration of time in which the resource is owned by a program or process. However, many of these resources (excluding static and stack memory) have to be manually managed ie. you must explicitly allocate or acquire the resource and once finished, explicitly free the resource. Manually managing resources is a notorious source of bugs as it can be hard to determine when the lifetime of a resource actually ends, especially when they are used in a concurrent environment where there may be multiple references to the resource.<br><br>One solution that was adopted was to introduce a parallel process that observed how objects and resources are used throughout the lifetime of the process and clean up any that are not in use. This technique is called Garbage Collection however, requiring an additional process to clean up resources can often be a no-go for programs that need to to be as efficient and fast as possible.

===

<!-- .slide: id="handling-resources/raii" data-auto-animate -->

### RAII

<!-- Diagram contrasting RAII object (mutex + lock) to non-RAII object (mutex) in terms of scope lifetime -->

notes: What if we could tie the lifetime of a resource to some *owning* object such that when the object was constructed it acquired the resource and when destroyed it freed said resources? This would allow us to create handles to resources which can live on the stack, while owning some external resource like a region of dynamic memory, a socket etc.. In this model, handles are constructed and pushed onto the stack only once all resources acquired and when the handle is popped off the stack, it frees any resources it is holding onto, ensuring that no resource lives longer than it's owning object and thus preventing leaks! This idiom is nearly as old as C++ and is named *Resource Acquisition Is Initialization* or RAII.

---

<!-- .slide: id="handling-resources/raii/value-reference-pointer-and-move-semantics" data-auto-animate -->

#### Value, Reference, Pointer and Move Semantics

notes: Before we can talk about how to utilise RAII for our own types we need to discuss semantics. Semantics are the meaning a piece of code has in a programming language. The semantics of different expressions are important to understand as they describe how resources are copied, referenced or transferred between objects.

---

<!-- .slide: id="handling-resources/raii/value-reference-pointer-and-move-semantics/value-semantics" data-auto-animate -->

##### Value Semantics

<!-- Diagram of bytes being copied on different slide then code -->

```cpp
                       auto x = 123;  // x = 123
                       auto y = x;    // y = 123
                       x == y;        // true
                       &x == &y;      // false
```
<!-- .element: class="fragment" data-id="semantics-ex1" -->

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡:<a href="https://godbolt.org/z/5Prroaqr1">https://godbolt.org/z/5Prroaqr1</a></span>

notes: By default, C++ expressions have value semantics (or copy semantics) meaning that the resulting value of an expression is copied or duplicated when bound to a new object. Take the example (above), the semantics of `=` here are to bind the value from the RHS to the label (variable) on the LHS by copying the underlying data. This is true regardless of what the type is on the RHS but does depend on the type denoted on the LHS.

---

<!-- .slide: id="handling-resources/raii/value-reference-pointer-and-move-semantics/reference-semantics/1" data-auto-animate -->

##### Reference Semantics

<!-- Diagram of bytes being referred on different slide then code -->

```cpp
                       auto x = 123;  // x = 123
                       auto& y = x;   // y = 123
                       auto z = y;    // z = 123
                       x == y;        // true
                       &x == &y;      // true
                       x == z;        // true
                       &x == &z;      // false

```
<!-- .element: data-id="semantics-ex1" -->

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/5njr3nM6Y">https://godbolt.org/z/5njr3nM6Y</a></span>

notes: C++ allows you to introduce reference semantics as a second class type. Reference semantics are probably the type of semantics you are most familiar with as they are very popular in GC languages. In GC languages, a variables is implicitly a reference to an object; rather than being the object itself, with no way to explicitly dereference or follow the reference, it is all automatic. When you copy a reference in a GC language, it doesn't copy the underlying object, just the reference to it.<br><br>References in C++ operate in a similar way to GC languages except that you must explicitly denote when an object's type is a reference type otherwise value semantics will be used. We can see observe this in the example (above). First we create a reference to `x`, `y` and then bind a non-reference `z` to `y`. We can see that C++ sees `x` and `y` as the same object, ie. `y` is really just an alias to the object `x` however, because `z` is not a reference type the value of `x` was copied through `y` into a new object `z`.

---

<!-- .slide: id="handling-resources/raii/value-reference-pointer-and-move-semantics/reference-semantics/2" data-auto-animate -->

##### Reference Semantics

```cpp
                   auto x = 123;           // x = 123
                   auto& y = x;            // y = 123
                   y = 456;                // z = 123
                   x == y;                 // true
                   &x == &y;               // true
                   std::println("{}", x);  // 456

```
<!-- .element: data-id="semantics-ex1" -->

notes: It is important to note that references in C++ cannot be rebound, if you assign to an existing reference it will mutate the referred to object.

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/edj1Tc9Wj">https://godbolt.org/z/edj1Tc9Wj</a></span>

---

<!-- .slide: id="handling-resources/raii/value-reference-pointer-and-move-semantics/pointer-semantics/1 data-auto-animate -->

##### Pointer Semantics

<!-- Diagram of bytes being copied for pointer but pointing to the same thing on different slide then code -->

```cpp
                       auto x = 123;  // x = 123
                       auto* y = &x;  // y -> x
                       auto z = y;    // z -> x
                       x == *y;       // true
                       &x == y;       // true
                       x == *z;       // true
                       &x == z;       // true
                       y == z;        // true


```
<!-- .element: data-id="semantics-ex1" -->

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/8fxPjKT1d"></a>https://godbolt.org/z/8fxPjKT1d</span>

notes: Pointers allow us to achieve reference semantics as well however pointers themselves store data, that being some memory address. Because of this, any operation performed on the pointer object directly will affect the pointer; not the pointed-to object. You must instead dereference a pointer using the indirection operator (prefix `*`) in order to access the object at the stored address. Notice in the example that assigning from a pointer will copy the pointer/address even though we haven't designated `z` is a pointer type like we did with `y` (can talk about `auto` type deduction).

---

<!-- .slide: id="handling-resources/raii/value-reference-pointer-and-move-semantics/pointer-semantics/2 data-auto-animate -->

##### Pointer Semantics

```cpp
                       auto x = 123;  // x = 123
                       auto* y = &x;  // y -> x
                       auto z = *y;   // z = 123
                       *y == z;       // true
                       &x == y;       // true

                       y = &z;        // y -> z
                       &x == y;       // false
                       &z == y;       // true
                       *y == x;       // true


```
<!-- .element: data-id="semantics-ex1" -->

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/7a3vfrrn9">https://godbolt.org/z/7a3vfrrn9</a></span>

notes: Unlike references, pointers can be rebound to store a different address. This will change which object is access through the indirection operator.

---

<!-- .slide: id="handling-resources/raii/value-reference-pointer-and-move-semantics/move-semantics data-auto-animate -->

##### Move Semantics

<!-- Diagram of ownership being transferred on different slide then code -->

```cpp
                  auto x = "abc"s;          // x = "abc"
                  std::println("{:?}", x);
                  
                  auto y = std::move(x);    // y <- x
                  std::println("{:?}", x);  // x = ""
                  std::println("{:?}", y);  // y = "abc"


```
<!-- .element: data-id="semantics-ex1" -->

notes: Compared to reference and pointer semantics; which are used to share a resource, move semantics use used when we wish to transfer ownership of a resource. In this example we have constructed a string `x`. We then transfer or *move* ownership of any resources; in this case memory and data, from `x` to `y` and observe that now `y` owns that data previously found in `x` and with `x` itself is empty.

---

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
