# Structured C++

During this meetup we be discussing how to define our own types in C++ using classes and
structures. This will involve looking at how C++ structures data in memory by created
basic aggregations. We will then briefly look at how to hide parts of a type to users of
the type followed by looking at how to define operations on our custom types. We will
then look at how to control the lifetime of our types and bind the lifetime or resources
our type owns to the lifetime of the containing class itself using RAII. At the end we
will look at some extra properties possessed by classes and highlight how they can be
useful.

## Overview

- [Structured C++](#structured-c)
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
  - [Sources](#sources)

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

---v

*"System"* refers to the Operating System (OS) kernel your computer is using in combination with any
core libraries used to make a complete OS and the architecture of the CPU used to run the system.

---v

*"Target"* refers to the type of machine code a compiler generates for a given CPU architecture from
some source code.

---v

*" (CPU) Architecture"* is the instruction set of a CPU eg. x86/x86_64, ARM etc.

===>

### Small Notes

---v?

Throughout these slides I will be using C++23's `std::print` and `std::println` functions.
These are not widely available in compilers yet but are available using the
[`{fmt}`](https://fmt.dev) library.

<!-- ---v?

Throughout the examples in these slides I will exclusively be using the `struct` keyword to
define types in C++. This is so that you will not confuse the use of the `class` with OOP
classes -->

---v?

Some links are Godbolt links which a small C++ build instances. Follow them to test code
snippets in the browser!

===>

## Basic Data Aggregations

First we will look at how to build the most basic structures in C++. These are known as
data aggregates and a really an amalgamation of other pieces of data. Structures are
introduced by the `struct` or `class` keyword followed by the name of the new type along
with a semicolon-ending braces containing the types details.

```cxx
struct A {
    // ... type
};
```

---v

We won't use the `class` keyword for now due to it having some different default access
permissions to `struct` which will be discussed [later](#access-control).

--->

We can then add member variables by declaring them the same as we would in free functions.
You can add any number of members to a type. Note that our structure has no special properties,
it is simply an amalgamation of its members.

```cxx
struct A {
    char chr;
    int num;
    float dec;
};
```

--->

We can initialise the aggregate structure using [aggregate initialisation](https://en.cppreference.com/w/cpp/language/aggregate_initialization) with [designated initializers](https://en.cppreference.com/w/cpp/language/aggregate_initialization#Designated_initializers).

```cxx
struct A {
    char chr;
    int num;
    float dec;
};

auto main() -> int {

    auto const a = A { .chr = 'a', .num = 123, .dec = 3.14f };

    return 0;
}
```

--->

Members can be access using the `obj.var` syntax.

```cxx
struct A {
    char chr;
    int num;
    float dec;
};

auto main() -> int {

    auto const a = A { .chr = 'a', .num = 123, .dec = 3.14f };

    // Reads data stored in members.
    std::println("{}", a.chr);  // a
    std::println("{}", a.num);  // 123
    std::println("{}", a.dec);  // 3.14

    return 0;
}
```

See it on Godbolt ⚡: <https://godbolt.org/z/MnGraPs8P>

<!-- fragment -->
So how does it look in memory? What shape does the compiler give our structure?

===>

## Data Layout

> In C++, structures will (generally) have their members located right next to each other
> in memory making the structure very compact. Which is part of C++'s goal for Zero Cost
> Abstraction meaning what you don't use you don't pay for.

<!-- Diagram of structures layout -->

--->

> As we can see our structure only takes of up as much space as the sum of the sizes of its
> members...

<!-- New diagram with animation showing the size of members and total size -->

> ...almost. If we actually [build & run](https://godbolt.org/z/5hEfeaTK9) we can see that our structures
> size is 12 bytes, not 9 bytes. Why has the compiler made `A` 3 bytes larger than it needs to be?

<!-- Take questions -->

---v

### Padding

In some situations the compiler may add empty bytes; known as padding, before a member of a structure
so that it is *"byte aligned"* or *"naturally aligned"*. This is done to help optimises the CPU's
ability to read and write to the address the member is located at.

---v

#### Natural Alignment

> *Natural Alignment* means that the starting address of some piece of data is located at an address that
> is a multiple of its size.

> The CPU always accesses memory by a single memory word at a time. This means that the largest primitive
> data type supported by the computer must be able to fit into the size of a memory word otherwise the
> CPU would have to access a single data type in chunks causing the CPU to have to coordinate between two
> memory pages. Luckily most, if not all systems behave have a memory word size that is at least as large
> as its largest supported primitive type.

> However, when dealing structured data we often have datums with different sizes. If they are packed
> tightly together, the memory can become misaligned resulting in the split memory access issues mention
> before. We can see this in our `A` type.

---v

> Let's say we want to access each member of an instance of `A` and the members are packed right next to
> each other. First the CPU with fetch the first full memory word size (assumed to be 64-bits or 8 bytes)
> which will retrieve all of `chr`, `num` and 3 bytes of `dec`.

<!-- diagram of retrieved memory -->

> The CPU can manipulate `chr` and `num` fine because all of their data has been access however, we cannot
> manipulate `dec`. From here the CPU would have to verify the remain bytes of `dec` are available in the
> cache, retrieve them if they are not, and combine it with the existing data it holds. This would require
> lots of complex circuitry to achieve.

<!-- Diagram showing left out data from memory fetch of `A` -->

---v

> Instead, compilers will add padding so that certain datums start at some power-of-2 memory address
> boundary. In this case, the compiler add 3 bytes of padding after `chr` so that any data for the member
> `dec` is pushed out of the memory word containing `chr` and `num` which means the data of `dec` is not
> split across memory words. This makes `num` now live on the 32-bit boundary line and `dec` on the
> 64-bit boundary line. This dramatically reduced the logic the CPU needs to perform as it simply just
> fetches performs another access to the needed datum (`dec`) and it can guarantee it will all be there.

<!-- True structure diagram with shift -->

--->

### Application Binary Interface

But how does the compiler *know* what shape to give a programming language's types and structures in memory?
To understand this we will need to look at how two programs interact.

---v

> When building a library you will often expose certain *symbols* which can be used in the source code of a
> dependant program or library. The signature of these symbols; along with the semantics imposed by the
> language these two programs are written in, creating the interface in which these two source code program
> modules can interact with each other. This is know as an API (Application Programming Interface).

<!-- Show example of source code importing a module -->

---v

> The compiled version of an API is what is known as the ABI (Application Binary Interface). An ABI is how
> two binary program modules (ie. compiled source code) interact with each other. The ABI is usually defined
> and implemented by compilers and describes how function/subroutines are executed, how types and structures
> get mapped to memory etc. in a systems machine code.

<!-- Show example of exe interacting with .so or other lib -->

---v

> Therefore, how structures are mapped onto memory is described by the systems ABI such that different binary
> program modules can communicate and understand each other. This is essential because it is what allows your
> programs to interact with your Operating System (OS) which is how it is able to execute!

<!-- Some diagram, not sure yet -->

<!-- ---v

Having a *"stable"* ABI for a library prevents dependant programs from having to be recompiled if a new library
is distributed, installed etc.. This is especially useful for programming languages as it allows developers to
ensure that the mapping between the program compiled from source code written in said language is consistent.
What this means is if your language undergoes an upgrade to compiler, interpreter, standard library etc. you
will still be able to link to it and use it with no problems. If the ABI of the language changed and became the
default for your system you would have to recompile every program written in that language for it to work properly.

---v

One of C++'s most prominent ABI is Intel's C++ Itanium ABI which is a cross-architecture ABI
use by most compilers (not MSVC).

A language/systems ABI is also what allows two (or more) languages to interact with each other.
This is done by introducing a Foreign Function Interface (FFI) which wraps a symbol from one language in the language trying to call the *foreign* function. This allows new languages to
leverage the capabilities of libraries written in different languages like Operating System (OS)
calls. Often languages with create FFI to C libraries. -->

---v

Now that we have looked at some of the machine level details of C++'s structures lets move
back into some more language level constructs.

## Access Control

C++ allows us to control which members can be accessed from outside the class. This is done
with access modifier labels. These will apply to any members; data or function, below the
label in the types definition. Access modifiers allow use to encapsulate/internalize parts
of the type so it cannot be modified by anyone outside the type itself.

--->

<!-- table of rules for access modifiers -->
<!-- <style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  overflow:hidden;padding:10px 5px;word-break:normal;}
.tg th{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;}
.tg .tg-c3ow{border-color:inherit;text-align:center;vertical-align:top}
.tg .tg-0pky{border-color:inherit;text-align:left;vertical-align:top}
</style>
<table class="tg">
<thead>
  <tr>
    <th class="tg-c3ow">Specifiers</th>
    <th class="tg-c3ow">Same Class</th>
    <th class="tg-c3ow">Derived Class</th>
    <th class="tg-c3ow">Outside Class</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-0pky"><span style="color:#905;background-color:#ddd">`public`</span></td>
    <td class="tg-c3ow">Yes</td>
    <td class="tg-c3ow">Yes</td>
    <td class="tg-c3ow">Yes</td>
  </tr>
  <tr>
    <td class="tg-0pky"><span style="color:#905;background-color:#ddd">`private`</span></td>
    <td class="tg-c3ow">Yes</td>
    <td class="tg-c3ow">No</td>
    <td class="tg-c3ow">No</td>
  </tr>
  <tr>
    <td class="tg-0pky"><span style="color:#905;background-color:#ddd">`protected`</span></td>
    <td class="tg-c3ow">Yes</td>
    <td class="tg-c3ow">Yes</td>
    <td class="tg-c3ow">No</td>
  </tr>
</tbody>
</table> -->

<!-- fragment -->
We will look at the first two columns in more detail when discussing [virtual polymorphism](#virtual-polymorphism).

---v

Types declared with the `struct` keyword will have the `public` modifier applied to the
whole class by default.

Types declared with the `class` keyword will have the `private` modifier applied to the
whole class by default.

This is the only difference between the `class` and `struct` keyword in C++.

--->

Let's modify our `A` type from earlier by making `A::dec` private.

```cxx
struct A {
    char chr;
    int num;

private:
    float dec;
};

auto main() -> int {

    //! We can no longer use aggregate initializer
    //! because we've 'hidden' parts of A making
    //! it no longer an aggregate type.
    auto a = A { };
    a.chr = 'a';
    a.num = 123;

    std::println("{}", a.chr);  // a
    std::println("{}", a.num);  // 123
    // fmt::println("{}", a.dec);  //! Now fails if uncommented

    return 0;
}

```

<!-- fragment -->
See it on Godbolt ⚡: <https://godbolt.org/z/fW5xKPoEY>

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

## Sources

~
