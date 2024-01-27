<!-- .slide: id="access-control/description" data-auto-animate -->

## Access Control

<!-- Change first diagram to be circle, split in half with one member blocked in and another having an opening in the circle with arrow going in and out -->

<div><span class="fragment" data-fragment-index="1"><code class="cpp hljs language-cpp">x.foo</code></span>
<svg class="fragment" data-fragment-index="3" xmlns="http://www.w3.org/2000/svg" height="50" width="45" viewBox="-10 -100 448 512"><!--Font Awesome Free 6.5.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license/free Copyright 2024 Fonticons, Inc.--><path fill="#63E6BE" d="M438.6 105.4c12.5 12.5 12.5 32.8 0 45.3l-256 256c-12.5 12.5-32.8 12.5-45.3 0l-128-128c-12.5-12.5-12.5-32.8 0-45.3s32.8-12.5 45.3 0L160 338.7 393.4 105.4c12.5-12.5 32.8-12.5 45.3 0z"/></svg></div>

<div><span class="fragment" data-fragment-index="2"><code class="cpp hljs language-cpp">x.bar</code></span>
<svg class="fragment" data-fragment-index="4" xmlns="http://www.w3.org/2000/svg" height="50" width="50" viewBox="-10 -100 384 512"><!--Font Awesome Free 6.5.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license/free Copyright 2024 Fonticons, Inc.--><path fill="#ff0000" d="M342.6 150.6c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0L192 210.7 86.6 105.4c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3L146.7 256 41.4 361.4c-12.5 12.5-12.5 32.8 0 45.3s32.8 12.5 45.3 0L192 301.3 297.4 406.6c12.5 12.5 32.8 12.5 45.3 0s12.5-32.8 0-45.3L237.3 256 342.6 150.6z"/></svg></div>

note: C++ allows us to control which members can be accessed from outside the type. This is done with access modifier labels. These will apply to any members; data or function, below the label in the types definition. Access modifiers allow use to encapsulate/internalize parts of the type so it cannot be modified by anyone outside the type itself.

---

<!-- .slide: id="access-control/table" data-auto-animate -->

## Access Control

| Specifiers   | Same Class | Derived Class | Outside Class |
|--------------|------------|---------------|---------------|
| `public`     | Yes        | Yes           | Yes           |
| `private`    | Yes        | No            | No            |
| `protected`  | Yes        | Yes           | No            |

notes: C++ has three access control labels, public, private and protected. `public` members can be accessed from anyone outside the type and is used to define the API of our type. `private` members are completely inaccessible to anything outside the type. `protected` labels members that are private to anything outside the type except for types that are a part of the same inheritance tree.<br>We will look more at `protected` when we talk about virtual polymorphism.<br><br>Types declared with the `struct` keyword will have the `public` modifier applied to the whole type by default.<br>Types declared with the `class` keyword will have the `private` modifier applied to the whole type by default.<br>This is the only difference between the `class` and `struct` keyword in C++.

---

<!-- .slide: id="access-control/example/before" data-auto-animate -->

## Access Control

```cxx [1:]
struct A {
    char chr;
    int num;
    float dec;
};

auto main() -> int {

    auto a = A { .chr = 'a', .num = 123, .dec = 3.14 };

    std::println("{}", a.chr);  // a
    std::println("{}", a.num);  // 123
    std::println("{}", a.dec);  // 3.14

    return 0;
}
```
<!-- .element: data-id="member-access-ex" -->

---

<!-- .slide: id="access-control/example/private" data-auto-animate -->

## Access Control

```cxx [1: 5-6,17|5-6|17|11-13]
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
<!-- .element: data-id="member-access-ex" -->

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/6PPqn9nsW">https://godbolt.org/z/6PPqn9nsW</a></span>

notes: Let's modify our `A` type from earlier by making `A::dec` private. We now get a compiler error on line 17 stating `'float A::dec' is private within this context`. Notice that we also can no longer use Aggregate Initialisation (w/wout Designated Initialisers) as we can't even access `A::dec` to initialise it! (Don't worry, we can fix this later).<br><br>But if we hide members of our structure how are we able to access them? This is where defining operations or member functions (methods) are useful.
