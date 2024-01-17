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
    - [Small Note](#small-note)
  - [Basic Data Aggregations](#basic-data-aggregations)
  - [Data Layout](#data-layout)
    - [Unnamed](#unnamed)
      - [Reference Semantics in C++](#reference-semantics-in-c)
        - [Pointers](#pointers)
        - [References](#references)
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

## Some Terminology

To get us started I want to clarify some terminology that will be used during this
meetup.

When I refer to an *"object"* I am refer to a piece of data that lives in memory ie. the
actual 1s and 0s that make up the piece of data. An object has some value which is
of *some* type.

A *"value"* is the interpretation of some collection of bits according to a type. What
this means is that the same set of bits *might* have different meanings depending on
whatever type they are bound by eg. the bits `11110010011011010101111010111101` have the
value `-227713347` when interpreted as a C `int` but have the value `4067253949` when
interpreted as a C `unsigned int`.

The *"type"* of some value is an abstraction within a programming language that allows
us to constrain what operations and values an object can have.

When I refer to *"types"* or *"structure"* in C++, I am referring to C++'s various built-in
language-level data types; eg. `int`, `float`, `char` etc., often called POD (Plain Old Data)
types as well as classes and structures defined using the `class` and `struct` keywords
which can come from either the standard library, third-party libraries or your own code.
Combined with the previous definition; a type is anything in C++ that gives meaning to
objects* by defining and constraining what *values* (ie. valid bits) they can have as well
as what instructions (operations) can interact with those bits.

When referring to an *"identifier"*...

### Small Note

Throughout these slides I will be using C++23's `std::print` and `std::println` functions.
These are not widely available in compilers yet but are available using the [`{fmt}`](https://fmt.dev) library.

## Basic Data Aggregations

First we will look at how to build the most basic structures in C++. These are known as
data aggregates and a really an amalgamation of other pieces of data. The syntax for
defining a type is as follows.

```cxx
struct A {
    // ... type
};
```

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

Our class `A` has no special properties, it is simply an amalgamation of its members. What
this means is that nothing else is put into the type by the compiler\*. This is part of C++'s
goal for Zero Cost Abstraction meaning what you don't use you don't pay for. You can observe
this using the `sizeof` operator on the structure and compare its size in bytes to that size
of the sum of the types that makes up the structure.

```cxx
struct A {
    char chr;
    int num;
    float dec;
};

auto main() -> int {
    auto const sum_sz = sizeof(char) + sizeof(int) + sizeof(float);
    std::println("Sum size: {} bytes.", sum_sz);  // Sum size: 9 bytes.
    std::println("Size of A: {} bytes.", sizeof(A));  // Size of A: 9 bytes.
    
    return 0;
}
```

## Data Layout

But how are members laid out in memory? What shape does a compiler give structures in C++?
This is a relatively easy question to answer. In general C++ packs the members of a structures
all together in a sequence.

### Unnamed

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

<!-- Pointer diagram -->

##### References

References are just like pointers but will automatically dereference themselves and cannot
point to nothing making them more an alias for another object.

<!-- ### [[Digression | Semantics]]

#### Value Semantics

Value semantics means that when you manipulate a variable or piece of data you are handling
the actual object. This means by default there is only one variable per object ie. only a
single identifier to a location in memory.

#### Reference Semantics

Reference semantics mean that an identifier can refer to any memory location that stores a type
compatible object ie. every handle on data is a pointer to a different memory address. This may
seem like a flaw in a languages design but this property is what allows dynamic languages to be
*dynamic*.

Being dynamic means that the type of your object is resolved when the program is being executed
(runtime) by an interpreter and not when the program was compiled (if it was at all). This can
help alleviate the cognitive load while developing and lends well to both functional and OOP
paradigms. It also allows the same identifier to be used to reference an object of a different
type.

However, being dynamically typed means that you almost always have to use references to objects
which prevents you from being able to store data more compactly. Value semantics allow a
compiler or interpreter to more tightly pack data together saving on memory and reducing the
read and write load due to not having to follow references through memory. -->

### Padding

[\*Earlier](#basic-data-aggregations) I mentioned that C++ compilers will not add anything to
structures. I kinda lied. If we actually [build & run](https://godbolt.org/z/5hEfeaTK9) the example from before (and below) we see that `A` has a size of 12 bytes.

```cxx
struct A {
    char chr;
    int num;
    float dec;
};

auto main() -> int {

    auto const sum_sz = sizeof(char) + sizeof(int) + sizeof(float);
    fmt::println("Sum size: {} bytes.", sum_sz);  // Sum size: 9 bytes.
    fmt::println("Size of A: {} bytes.", sizeof(A));  // Size of A: 12 bytes.

    return 0;
}
```

In some situations the compiler may add padding which is just empty
memory contained in a structure, after certain members so that any following members are *"byte aligned"* or *"naturally aligned"*. Why would we want the compiler to use up extra memory that
isn't being used? Because this optimises the CPU's ability to read and write to the address the
data is located at.

#### Natural Alignment

*Natural Alignment* means that data is located at a memory address that is a multiple of its size. We can observe

### Application Binary Interface

How does a compiler map *types* in a programming language to an actual layout in memory? Well lets first look at how a program (executable) interacts with a library.

When building a library you will expose certain *symbols* which can be used in the source code
of a dependant program or library. The signature of these symbols; along with the semantics
imposed by the language these two programs are written in, creating the interface in which these
two source code program modules can interact with each other. This is know as an API
(Application Programming Interface).

The compiled version of an API is what is known as the ABI (Application Binary Interface). An
ABI is how two binary program modules (ie. compiled source code) interact with each other. The
ABI is usually defined and implemented by compilers and describes how function/subroutines
are executed, how types get mapped to memory and how executables find symbols exposed by
external libraries among many other things.

Programming languages usually have an ABI between the target architecture and the programming
language (implementation) itself. This is used to define the layout, size and alignment of
the languages data types.

Having a *"stable"* ABI for a library prevents dependant programs from having to be recompiled
if a new library is distributed, installed etc.. This is especially useful for programming
languages as it allows developers to ensure that the mapping between the program compiled
from source code written in said language is consistent. What this means is if your language
undergoes an upgrade to compiler, interpreter, standard library etc. you will still be able to link to it and use it with no problems. If the ABI of the language changed and became the default for your system you would have to recompile every program written in that language for it to work properly.

One of C++'s most prominent ABI is Intel's C++ Itanium ABI which is a cross-architecture ABI
use by most compilers (not MSVC).

A language/systems ABI is also what allows two (or more) languages to interact with each other.
This is done by introducing a Foreign Function Interface (FFI) which wraps a symbol from one language in the language trying to call the *foreign* function. This allows new languages to
leverage the capabilities of libraries written in different languages like Operating System (OS)
calls. Often languages with create FFI to C libraries.

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
