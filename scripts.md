# Structured C++

## Overview

During this meetup we will be discussing how to define our own types in C++ using classes
and structures. This will involve looking at how C++ structures data in memory by created
basic aggregations. We will then briefly look at how to hide parts of a type to users of
the type followed by looking at how to define operations on our custom types. We will
then look at how to control the lifetime of our types and bind the lifetime or resources
our type owns to the lifetime of the containing class itself using RAII. At the end we
will look at some extra properties possessed by classes and highlight how they can be
useful.

- [Structured C++](#structured-c)
  - [Overview](#overview)
  - [Some Terminology](#some-terminology)
    - [Some Small Notes](#some-small-notes)
  - [Basic Data Aggregations](#basic-data-aggregations)
  - [Data Layout](#data-layout)
    - [Padding](#padding)
      - [Natural Alignment](#natural-alignment)
        - [CPU Memory Access (Packed Data)](#cpu-memory-access-packed-data)
        - [CPU Memory Access (Padded Data)](#cpu-memory-access-padded-data)
    - [Application Binary Interface](#application-binary-interface)
  - [Access Control](#access-control)
  - [Defining Operations on Types](#defining-operations-on-types)
    - [Constant Operations](#constant-operations)
    - [Operator Overloading](#operator-overloading)
    - [Friend Functions](#friend-functions)
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
actual 1s and 0s that make up the piece of data. An object has *some* value which is
of *some* type.

---v

A *"value"* is the interpretation of some collection of bits according to a type. What
this means is that the same set of bits *might* have different meanings depending on
whatever type they are bound by.

<!-- Diagram illustrating this example: the bits `11110010011011010101111010111101` have the 
value `-227713347` when interpreted as a C `int` but have the value `4067253949` when
interpreted as a C `unsigned int`. -->

---v

The *"type"* of some value is an abstraction within the programming language which constrains
what operations and values an object can have. This includes built-in data types like `int`,
`float`, `char` etc., often called POD (Plain Old Data) types in C++ as well as user-defined
types introduced using the `class` and `struct` keywords.

---v

When referring to an *"identifier"* I am referring to the in-source name given to datums (variables),
free functions and member functions.

<!-- Diagram of variable syntax indicating identifier, type and value. -->

---v

*"System"* refers to the Operating System (OS) kernel your computer is using in combination with any
core libraries used to make a complete OS and the architecture of the CPU used to run the system.

<!-- Diagram of kernel in system stack -->

---v

*"Target"* refers to the type of machine code a compiler generates for a given CPU architecture from
some source code.

<!-- Diagram of compiler pipeline to different targets -->

---v

*"(CPU) Architecture"* is the set of instructions available on a CPU eg. x86/x86_64, ARM etc.

===>

### Some Small Notes

---v?

Throughout these slides I will be using C++23's `std::print` and `std::println` functions.
These are not widely available in compilers yet but but there are mirror versions available
from the [`{fmt}`](https://fmt.dev) library.

<!-- ---v?

Throughout the examples in these slides I will exclusively be using the `struct` keyword to
define types in C++. This is so that you will not confuse the use of the `class` with OOP
classes -->

---v?

In some slides you might see

"See it on Godbolt ⚡:"

followed by a link. These take you to an
online compiler instance setup to run the example which run in your browser!

===>

## Basic Data Aggregations

We can introduce a new type using the <code>struct</code> or <code>class</code> keywords.

> First we will look at how to build the basic structures in C++. These are known as aggregates
> are just an amalgamation of data and they are declared with the `struct` or `class` keywords.

```cxx
struct A {
    // ... details of the type
};
```

> We won't use the `class` keyword for now due to it having some different default access
> permissions to `struct` which will be discussed [later](#access-control).

--->

To add data to the type we simply declare some member variables without an initial value.

> We can then add member variables by declaring them the same as we would in free functions.
> You can add any number of members to a type. Note that our structure has no special properties,
> it is simply an amalgamation of its members.

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
So how does this look in memory? What shape does the compiler give our structure?

===>

## Data Layout

> In C++, structures will (generally) have their members located right next to each other
> in memory making the structure very compact. Which is part of C++'s goal for Zero Cost
> Abstraction meaning what you don't use you don't pay for.

<!-- Diagram of structures layout -->

--->

<!-- New diagram with animation showing the size of members and total size -->

...almost. If we actually check this on [Godbolt](https://godbolt.org/z/5hEfeaTK9) using
the `sizeof` operator we can see that our structures size is 12 bytes, not 9 bytes. Why
has the compiler made `A` 3 bytes larger than it needs to be?

---v

### Padding

In some situations the compiler may add empty bytes; known as padding, around member variables
of a structure so that it is *"byte aligned"* or *"naturally aligned"*. This is done to help
optimises the CPU's ability to read and write to the address the member is located at.

---v

#### Natural Alignment

*Natural Alignment* means that a objects' starting memory address is a multiple of its size in bytes.

---v

The CPU will always accesses memory by a single memory word at a time. Let's see how padding affects
how the CPU accesses memory when reading in a fixed size word with our `A` type.

<!-- auto-slide sections showing CPU reading memory one word at a time with svg's -->

> This means that the largest
> primitive > data type supported by the computer must be able to fit into the size of a memory word
> otherwise the CPU would have to access a single data type in chunks causing the CPU to have to
> coordinate between two > memory pages. Luckily most, if not all systems behave have a memory word
> size that is at least as large as its largest supported primitive type.

> However, when dealing structured data we often have data with different sizes. If they are packed
> tightly together, the memory can become misaligned resulting in the split memory access issues mention
> before. We can see this in our `A` type.

---v

##### CPU Memory Access (Packed Data)

Misaligned memory causes the CPU to fetch partial data meaning it need to find the missing data.

<!-- diagram of retrieved memory and how data has been left out using auto-slide and svgs -->

> Let's say we want to access each member of an instance of `A` and the members are packed right next to
> each other. First the CPU with fetch the first full memory word size (assumed to be 64-bits or 8 bytes)
> which will retrieve all of `chr`, `num` and only 3 bytes of `dec`, losing the rest. In order to operate
> on `dec` the CPU would have to verify if the remaining bytes are available in its cache; as it may have
> been lost along, retrieve them if they are not; performing a full RAM access request which is very costly
> when you have the rest of the data ready, and combine it with the existing data of `dec`. This would
> require lots of complex circuitry to achieve.

---v

##### CPU Memory Access (Padded Data)

The compiler will add 3 bytes of padding after `chr` so that `dec` is naturally aligned on the
32-bit boundary.

> Instead, compilers will add padding so that certain datums start at some power-of-2 memory address
> boundary. In this case, the compiler add 3 bytes of padding after `chr` so that any data for the member
> `dec` is pushed out of the memory word containing `chr` and `num` which means the data of `dec` is not
> split across memory words. This makes `num` now live on the 32-bit boundary line and `dec` on the
> 64-bit boundary line.

<!-- True structure diagram with shift -->

---v

Now the CPU has a much easier time access memory correctly.

<!-- diagram of retrieved memory and how data isn't being left out anymore using auto-slide and svgs -->

> This dramatically reduced the logic the CPU needs to perform as it simply just fetches performs another
> access to the needed datum (`dec`) and it can guarantee it will all be there. This is a common theme in
> the relationship between software and hardware. Often by making certain things true to the benefit of
> the hardwares design we can dramatically improve the performance of our software.

--->

### Application Binary Interface

But how does the compiler *know* what shape to give a programming language's types and structures in memory?
To understand this we will need to look at how two programs interact.

---v

The symbols you use from other modules form an Application Programming Interface (API) between your source
code and the other module.

> When building a library you will often expose certain *symbols* which can be used in the source code of a
> dependant program or library. The signature of these symbols; along with the semantics imposed by the
> language these two programs are written in, creating the interface in which these two source code program
> modules can interact with each other. This is know as an API (Application Programming Interface).

<!-- Show example of source code importing a module -->

---v

The manner in which two compiled (binary) modules/programs interact forms an Application Binary Interface
(ABI).

<!-- Show example of exe interacting with .so or other lib -->

> The compiled version of an API is what is known as the ABI (Application Binary Interface). An ABI is how
> two binary program modules (ie. compiled source code) interact with each other. The ABI is usually defined
> and implemented by compilers and describes how function/subroutines are executed, how types and structures
> get mapped to memory etc. in a systems machine code.
>
> Therefore, how structures are mapped onto memory is described by the systems ABI such that different binary
> program modules can communicate and understand each other. This is essential because it is what allows your
> programs to interact with your Operating System (OS) which is how it is able to execute!

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

> C++ allows us to control which members can be accessed from outside the class. This is done
> with access modifier labels. These will apply to any members; data or function, below the
> label in the types definition. Access modifiers allow use to encapsulate/internalize parts
> of the type so it cannot be modified by anyone outside the type itself.

--->

<!-- table of rules for access modifiers -->

> C++ has three access control labels, public, private and protected. `public` members can be
> accessed from anyone outside the type and is used to define the API of our type. `private`
> members are completely inaccessible to anything outside the type. `protected` labels members
> that are private to anything outside the type except for types that are a part of the same
> inheritance tree.
>
> We will look more at `protected` when we talk about virtual polymorphism.
>
>
> Types declared with the `struct` keyword will have the `public` modifier applied to the
> whole type by default.
>
> Types declared with the `class` keyword will have the `private` modifier applied to the
> whole type by default.
>
> This is the only difference between the `class` and `struct` keyword in C++.

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

    auto a = A { };
    a.chr = 'a';
    a.num = 123;

    std::println("{}", a.chr);  // a
    std::println("{}", a.num);  // 123
    std::println("{}", a.dec);  //! Now fails to compile

    return 0;
}
```

<!-- fragment -->
See it on Godbolt ⚡: <https://godbolt.org/z/6PPqn9nsW>

## Defining Operations on Types

> But if we hide members of our structure how are we able to access them? This is where defining member
> functions (methods) come into play. Members are special functions that are bound to our type. To define
> a method for a type we simply declare the function and its definition within the types own definition
> exactly the same as member variables.

```cxx
struct A {
private:
    char chr = 'a';
    int num = 123;
    float dec = 3.14f;

public:
    auto to_string() -> std::string {
        return "todo...";
    }
};
```

--->

We can call the method on instance of `A` by using the `.` (member access) operator.

```cxx
struct A {
private:
    char chr = 'a';
    int num = 123;
    float dec = 3.14f;

public:
    auto to_string() -> std::string {
        return "todo...";
    }
};

auto main() -> int {

    auto a = A { };

    std::println("{}", a.to_string());  // todo...

    return 0;
}
```

<!-- fragment -->
See it on Godbolt ⚡: <https://godbolt.org/z/4Yebv5bv8>

--->

But why do we have to access `to_string()` through the `.` operator? The `.` operator allows the method to
access the internals of the type, even private members. Implicitly all methods are defined to take a hidden
first parameter. This hidden parameter is a pointer to the actual object in memory of which is an instance
of the type the method was declared on called `this`.

```cxx
struct A {

    // member details...

public:
    auto to_string(A* this) -> std::string {
        return "todo...";
    }
};
```

--->

This is because C++ does not store the instructions in the **.data** portion of the resulting executable ie.
not with constant, static and stack variables but rather in the **.text** section of the executable alongside
the rest of the programs instructions.

<!-- Diagram of ELF file layout -->

--->

As such a method needs to be able to access the other data and members
associated with type which is achieved by taking the address of the object and passing that to the method as
its first parameter. We can see this effect illustrated by making `to_string()` a free function and passing
a `A*` to it.

```cxx
struct A {
    char chr = 'a';
    int num = 123;
    float dec = 3.14f;
};

auto to_string(A* this) -> std::string {
    return "todo...";
}

auto main () -> {
    auto a = A { };

    std::println("{}", to_string(&a))
}
```

--->

Since `this` is a pointer we can access any members using regular pointer operators like `*` and `->`. We
can also omit `this` in contexts where there is no naming conflict.

```cxx
struct A {

    // member details...

public:
    auto to_string() -> std::string {
        return std::format("{{ .chr = '{}', .num = {}, .dec = {:.2f} }}", this->chr, (*this).num, dec)
    }
};

auto main() -> int {

    auto a = A { };

    fmt::println("{}", a.to_string());

    return 0;
}
```

<!-- fragment -->
See it on Godbolt ⚡: <https://godbolt.org/z/ne87489bK>

--->

### Constant Operations

Sometimes we want an object to be immutable however this would break the previous example. This is because
the member `to_string()` along expects to take a first argument of type `A*` not `const A*`. Passing a
constant variable to a non constant-accepting parameter cause us to discard the qualifiers (`const`) from
the object which would make it mutable.

```cxx
struct A {

    // member details...

public:
    auto to_string() -> std::string {
        return std::format("{{ .chr = '{}', .num = {}, .dec = {:.2f} }}", this->chr, (*this).num, dec)
    }
};

auto main() -> int {

    auto a const = A { };

    fmt::println("{}", a.to_string());  // error: passing 'const A' as 'this' argument discards qualifiers

    return 0;
}
```

<!-- fragment -->
See it on Godbolt ⚡: <https://godbolt.org/z/K3PGaG4ee>

---v

But this should work because we know `A::to_string()` doesn't modify `a` internals. Luckily we can mark such
functions that are constant as `const` to the compiler so they work with immutable instances of the type. This
also works with mutable variables as C++ is able to promote them to `const` for the scope of the method.

```cxx
struct A {

    // member details...

public:
    // added const after function name and parameters...
    //       ...here vvvvv
    auto to_string() const -> std::string { /* ... */ }
};

auto main() -> int {

    auto const a = A { };
    auto a2 = A { };

    fmt::println("{}", a.to_string());  // works
    fmt::println("{}", a2.to_string());  // also works

    return 0;
}
```

<!-- fragment -->
See it on Godbolt ⚡: <https://godbolt.org/z/hEE178Pje>

---v

In essence this simple makes the hidden `this` parameter of type `const A*` instead of `A*`.

```cxx
struct A {

    // member details...

public:
    auto to_string(const A* this) -> std::string {
        return std::format("{{ .chr = '{}', .num = {}, .dec = {:.2f} }}", this->chr, (*this).num, dec)
    }
};
```

### Operator Overloading

We can also overload C++'s operators for custom types. This works similar to regular operator
overloading however, as we've seen with other methods, the first argument is the `this` pointer
to the type's object in memory. This means that you only have to define the RHS argument.

```cxx
struct A {

    // member details...

public:
    auto to_string() const -> std::string { /* ... */ }

    auto operator+ (A const& rhs) const -> A {
        auto result = A { };
        result.num = this->num + rhs.num;
        result.dec = this->dec + rhs.dec;
        return result;
    }
};

auto main() -> int {

    auto const a1 = A { };
    auto const a2 = A { };
    auto const a3 = a1 + a2;

    fmt::println("{}", a1.to_string());
    fmt::println("{}", a2.to_string());
    fmt::println("{}", a3.to_string());

    return 0;
}
```

<!-- fragment -->
See it on Godbolt ⚡: <https://godbolt.org/z/hxKfe4P8K>

### Friend Functions

Sometime we need external types and functions to have access to the internals of our type even
if they are unrelated. This is where friend functions and classes come in. For example, we
can only overload operators on a type such that the type is always the LHS argument. What if
we need an object of a different type to be on LHS? We can make a friend function inside the
our type!

```cxx
struct A {

    // member details...

public:
    auto to_string() const -> std::string { /* ... */ }

    friend auto operator< (int const& lhs, A const& rhs) -> bool {
        return lhs < rhs.num;
    }
};

auto main() -> int {

    auto const a = A { };

    fmt::println("123 < a is {}", 123 < a);
    fmt::println("111 < a is {}", 111 < a);
    fmt::println("200 < a is {}", 200 < a);

    return 0;
}
```

<!-- fragment -->
See it on Godbolt ⚡: <https://godbolt.org/z/xYP3Gszxz>

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
