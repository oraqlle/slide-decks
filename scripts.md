# C++ Classes

During this meetup we be discussing how to define our own types in C++ using classes and
structures. This will involve looking at how C++ structures data in memory by created
basic aggregations. We will then briefly look at how to hide parts of a type to users of
the type followed by looking at how to define operations on our custom types. We will
then look at how to control the lifetime of our types and bind the lifetime or resources
our type owns to the lifetime of the containing class itself using RAII. At the end we
will look at some extra properties possessed by classes and highlight how they can be
useful.

## Overview

- [C++ Classes](#c-classes)
  - [Overview](#overview)
  - [Some Terminology](#some-terminology)
    - [Small Notes](#small-notes)
  - [Basic Data Aggregations](#basic-data-aggregations)
  - [Data Layout](#data-layout)
    - [Padding](#padding)
      - [Natural Alignment](#natural-alignment)
    - [Application Binary Interface](#application-binary-interface)
  - [Access Control](#access-control)
  - [Defining Operations on Custom types](#defining-operations-on-custom-types)
  - [Handling Resources](#handling-resources)
    - [RAII](#raii)
    - [Constructors and Destructors](#constructors-and-destructors)
  - [Static Storage Duration](#static-storage-duration)
  - [Polymorphism](#polymorphism)
    - [Parametric Polymorphism](#parametric-polymorphism)
    - [Virtual Polymorphism](#virtual-polymorphism)

===>

## Some Terminology

> To get us started I want to clarify some terminology that will be used during this
> meetup.

---v

When I refer to an *"object"* I am refer to a piece of data that lives in memory ie. the
actual 1s and 0s that make up the piece of data. An object has some value which is
of *some* type.

---v

A *"value"* is the interpretation of some collection of bits according to a type. What
this means is that the same set of bits *might* have different meanings depending on
whatever type they are bound by eg. the bits `11110010011011010101111010111101` have the
value `-227713347` when interpreted as a C `int` but have the value `4067253949` when
interpreted as a C `unsigned int`.

---v

The *"type"* of some value is an abstraction within a programming language that allows
us to constrain what operations and values an object can have.

---v

When I refer to *"types"* or *"structure"* in C++, I am referring to C++'s various built-in
language-level data types; eg. `int`, `float`, `char` etc., often called POD (Plain Old Data)
types as well as classes and structures defined using the `class` and `struct` keywords
which can come from either the standard library, third-party libraries or your own code.
Combined with the previous definition; a type is anything in C++ that gives meaning to
objects* by defining and constraining what *values* (ie. valid bits) they can have as well
as what instructions (operations) can interact with those bits.

---v

When referring to an *"identifier"*...

===>

### Small Notes

---

Throughout these slides I will be using C++23's `std::print` and `std::println` functions.
These are not widely available in compilers yet but are available using the
[`{fmt}`](https://fmt.dev) library.

---

Some links are Godbolt links which a small C++ build instances. Follow them to test code
snippets in the browser!

===>

## Basic Data Aggregations

First we will look at how to build the most basic structures in C++. These are known as
data aggregates and a really an amalgamation of other pieces of data. The syntax for
defining a type is as follows.

```cxx
struct A {
    // ... type
};
```

--->

We can then add member variables by declaring them the same as we would in free functions.
You can add any number of members to a type and they can be access using the `obj.var`
syntax.

```cxx
struct A {
    char chr;
    int num;
    float dec;
};
```

<!-- fragment -->
Note that our structure has no special properties, it is simply an amalgamation of its members.
So how does it look in memory? What shape does the compiler give our structure?

===>

## Data Layout

In C++, structures will (generally) have their members located right next to each other
in memory making the structure very compact. Which is part of C++'s goal for Zero Cost
Abstraction meaning what you don't use you don't pay for.

<!-- Diagram of structures layout -->

--->

As we can see our structure only takes of up as much space as the sum of the sizes of its
members...

<!-- New diagram with animation showing the size of members and total size -->

...almost. If we actually [build & run](https://godbolt.org/z/5hEfeaTK9) we can see that our structures
size is 12 bytes, not 9 bytes. Why has the compiler made `A` 3 bytes larger than it needs to be?

<!-- Take questions -->

---v

### Padding

In some situations the compiler may add empty bytes; known as padding, before a member of a structure
so that it is *"byte aligned"* or *"naturally aligned"*. This is done to help optimises the CPU's
ability to read and write to the address the member is located at.

---v

#### Natural Alignment

*Natural Alignment* means that the starting address of some piece of data is located at an address that
is a multiple of its size.

<!-- fragment move in -->
The CPU always accesses memory by a single memory word at a time. This means that the largest primitive
data type supported by the computer must be able to fit into the size of a memory word otherwise the
CPU would have to access a single data type in chunks causing the CPU to have to coordinate between two
memory pages. Luckily most, if not all systems behave have a memory word size that is at least as large
as its largest supported primitive type.

<!-- fragment move in -->
However, when dealing structured data we often have datums with different sizes. If they are packed
tightly together, the memory can become misaligned resulting in the split memory access issues mention
before. We can see this in our `A` type.

---v

Let's say we want to access each member of an instance of `A` and the members are packed right next to
each other. First the CPU with fetch the first full memory word size (assumed to be 64-bits or 8 bytes)
which will retrieve all of `chr`, `num` and 3 bytes of `dec`.

<!-- diagram of retrieved memory -->

The CPU can manipulate `chr` and `num` fine because all of their data has been access however, we cannot
manipulate `dec`. From here the CPU would have to verify the remain bytes of `dec` are available in the
cache, retrieve them if they are not, and combine it with the existing data it holds. This would require
lots of complex circuitry to achieve.

---v

Instead, compilers will add padding so that certain datums start at some power-of-2 memory address
boundary. In this case, the compiler add 3 bytes of padding after `chr` so that any data for the member
`dec` is pushed out of the memory word containing `chr` and `num` which means the data of `dec` is not
split across memory words. This makes `num` now live on the 32-bit boundary line and `dec` on the
64-bit boundary line. This dramatically reduced the logic the CPU needs to perform as it simply just
fetches performs another access to the needed datum (`dec`) and it can guarantee it will all be there.

<!-- True structure diagram with shift -->

--->

<!-- ### Unnamed

This is because C++ has value semantics by default which means that
you interact with a variable you interact directly with the underlying object, not a reference
to a *"type compatible"* object. This means that there is only a single identifier for any
object in memory...

#### Reference Semantics in C++

...sorta. While C++ has value semantics by default it does have second class support for
pointer and reference types allowing you to share data via multiple identifiers. But how do
pointer and reference types still have value semantics?

##### Pointers

Pointers still obey C++'s value semantics because the value of a pointer is just the memory
address of some other piece of data, making a pointer a special (unsigned) integer. The value
of a pointer is just some number; but unlike other numbers, pointers can be *dereferenced*
making it appear as if it is the object itself.

<!-- Pointer diagram --\>

##### References

References are just like pointers but will automatically dereference themselves and cannot
point to nothing making them more an alias for another object. -->

### Application Binary Interface

How does a compiler map *types* in a programming language to an actual layout in memory? Well lets first look at how a program (executable) interacts with a library.

---v

When building a library you will expose certain *symbols* which can be used in the source code
of a dependant program or library. The signature of these symbols; along with the semantics
imposed by the language these two programs are written in, creating the interface in which these
two source code program modules can interact with each other. This is know as an API
(Application Programming Interface).

---v

The compiled version of an API is what is known as the ABI (Application Binary Interface). An
ABI is how two binary program modules (ie. compiled source code) interact with each other. The
ABI is usually defined and implemented by compilers and describes how function/subroutines
are executed, how types get mapped to memory and how executables find symbols exposed by
external libraries among many other things.

---v

Programming languages usually have an ABI between the target architecture and the programming
language (implementation) itself. This is used to define the layout, size and alignment of
the languages data types.

---v

Having a *"stable"* ABI for a library prevents dependant programs from having to be recompiled
if a new library is distributed, installed etc.. This is especially useful for programming
languages as it allows developers to ensure that the mapping between the program compiled
from source code written in said language is consistent. What this means is if your language
undergoes an upgrade to compiler, interpreter, standard library etc. you will still be able to link to it and use it with no problems. If the ABI of the language changed and became the default for your system you would have to recompile every program written in that language for it to work properly.

---v

One of C++'s most prominent ABI is Intel's C++ Itanium ABI which is a cross-architecture ABI
use by most compilers (not MSVC).

A language/systems ABI is also what allows two (or more) languages to interact with each other.
This is done by introducing a Foreign Function Interface (FFI) which wraps a symbol from one language in the language trying to call the *foreign* function. This allows new languages to
leverage the capabilities of libraries written in different languages like Operating System (OS)
calls. Often languages with create FFI to C libraries.

---v

## Access Control

~

## Defining Operations on Custom types

~

## Handling Resources

~

### RAII

~

### Constructors and Destructors

~

## Static Storage Duration

~

## Polymorphism

~

### Parametric Polymorphism

~

### Virtual Polymorphism

~
