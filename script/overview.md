<!-- .slide: id="overview" -->

## Overview

- Terminology<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
- Aggregate Data Types<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
- Data Layout<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
- Access Control<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
- Defining Operations on Custom Types <!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
- Handling Resources<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
  - RAII<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
  - Constructors and Destructors<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
- Polymorphism<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
  - Parametric Polymorphism<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
  - Virtual Polymorphism<!-- .element: class="fragment fade-in-then-semi-out" style="font-size: xx-large;" -->
<!-- - Static Storage Duration -->

notes: During this meetup we will be discussing how to define our own types in C++ using classes
and structures. This will involve looking at how C++ structures data in memory by created
basic aggregations. We will then briefly look at how to hide parts of a type to users of
the type followed by looking at how to define operations on our custom types. We will
then look at how to control the lifetime of our types and bind the lifetime or resources
our type owns to the lifetime of the containing class itself using RAII. At the end we
will look at some extra properties possessed by classes and highlight how they can be
useful.
