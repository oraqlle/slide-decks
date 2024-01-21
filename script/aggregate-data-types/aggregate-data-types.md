<!-- .slide: id="Aggregate Data Types" data-auto-animate -->

## Aggregate Data Types

notes: To begin, let's look at how to define our own types in C++.

===

<!-- .slide: data-auto-animate -->

###### Declaring Types

```cpp [1:]
struct A {
    // ... details of the type
};
```
<!-- .element: data-id="ADT-Ex1" -->

notes: New types are introduced using either the `struct` or `class` keyword followed by the name of the type and a pair of curly braces where the data and function members will be defined. Note that the closing brace must have a semicolon after it, unlike functions.<br><br>We won't be using the `class` keyword for now as its semantics are slightly different to `struct`.<br><br>Currently our type is empty which isn't very interesting. Let's add some members so we can store the data of our type.

===

<!-- .slide: data-auto-animate -->

###### Declaring Data Members

```cpp [1: 1-5|2|3|4]
struct A {
    char chr;
    int num;
    float dec;
};
```
<!-- .element: data-id="ADT-Ex1" -->

notes: That's better! Now our type can store a `char`, an `int` and a `float`. Currently our type is classified as a POD Type; or a Plain Old Data Type. All this means is our type is simply an aggregate; or collection, of its members.

===

<!-- .slide: data-auto-animate -->

###### Initialisation

```cpp [1: 1-16|10|2-4,10|13]
struct A {
    char chr;
    int num;
    float dec;
};

auto main() -> int {

    // Aggregate: initialisers' order required.
    auto const x = A { 'a', 123, 3.14f };

    //  Designated: initialisers' order not required.
    auto const y = A { .num = 123, .chr = 'a', .dec = 3.14f };

    return 0;
}
```
<!-- .element: data-id="ADT-Ex1" -->

<!-- [Aggregate Initialisation](https://en.cppreference.com/w/cpp/language/aggregate_initialization) -->

<!-- [Designated Initializers](https://en.cppreference.com/w/cpp/language/aggregate_initialization#Designated_initializers) -->

notes: Because our type is an aggregate, we can construct and initialise an instance of it using aggregate initialization or designated initializers (as of C++20).<br><br>Aggregate Initialisation requires the initializer list be ordered the same as how the type's member variables are ordered.<br><br>Designated Initialisers allow for the initialisers to be ordered however you please, because the names of the data members are used in the list.

===

<!-- .slide: data-auto-animate -->

###### Accessing Members

```cpp [1:]
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
<!-- .element: data-id="ADT-Ex1" -->

===

<!-- .slide: data-auto-animate data-visibility="uncounted" -->

###### Accessing Members

```cpp [1: 1-16|11-13|1-16]
struct A {
    char chr;
    int num;
    float dec;
};

auto main() -> int {

    auto const a = A { .chr = 'a', .num = 123, .dec = 3.14f };

    std::println("{}", a.chr);  // a
    std::println("{}", a.num);  // 123
    std::println("{}", a.dec);  // 3.14

    return 0;
}
```
<!-- .element: data-id="ADT-Ex1" -->

<span class="fragment" style="font-size: large;">
    See it on Godbolt ⚡: <a href="https://godbolt.org/z/MnGraPs8P">https://godbolt.org/z/MnGraPs8P</a>
</span>

notes: We can access the members of an object using the binary dot operator with the left argument being an object and the left being the name of the member (data or function) you wish to access.

===

So how does this look in memory? What shape does the compiler give our structure?
