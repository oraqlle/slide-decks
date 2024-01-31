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
  - [Basic Data Aggregations](#basic-data-aggregations)
  - [Data Layout](#data-layout)
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

When I refer to *"types"* in C++, I am referring to C++'s various built-in language-level
data types; eg. `int`, `float`, `char` etc., often called POD (Plain Old Data) types as
well as classes and structures defined using the `class` and `struct` keywords which can
come from either the standard library, third-party libraries or your own code. Combined with
the previous definition; a type is anything in C++ that gives meaning to *objects* by
defining and constraining what *values* (ie. valid bits) they can have as well as
what instructions (operations) can interact with those bits.

## Basic Data Aggregations

First we will look at how to build the most basic structures in C++. These are known as
data aggregates and a really an amalgamation of other pieces of data. The syntax for
defining a type is as follows.

```cxx
struct A {
    // ... type
};
```

## Data Layout

~

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
