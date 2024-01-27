<!-- .slide: id="defining-operations-on-types" data-auto-animate -->

## Defining Operations on Types

===

<!-- .slide: id="defining-operations-on-types/methods/setup/1" data-auto-animate -->

```cxx [1:]
struct A {
    char chr;
    int num;

private:
    float dec;
};
```
<!-- .element: data-id="custom-ops-ex1" -->

notes: Members are special functions that are bound to our type. To define a method for a type we simply declare the function and its definition within the types own definition, exactly the same as member variables. Let's make all of our A type's members private to ensure that the type's internals are completely inaccessible.

===

<!-- .slide: id="defining-operations-on-types/setup/2" data-auto-animate -->

```cxx [1: 2-5]
struct A {
private:
    char chr;
    int num;
    float dec;
};
```
<!-- .element: data-id="custom-ops-ex1" -->

notes: We also will define default values for our members for the time being so we can observe them properly.

===

<!-- .slide: id="defining-operations-on-types/setup/3" data-auto-animate -->

```cxx [1: 3-5]
struct A {
private:
    char chr = 'a';
    int num = 123;
    float dec = 3.14f;
};
```
<!-- .element: data-id="custom-ops-ex1" -->

notes: Now we could define a method to access each member. This could be done using OOP getters and setters or we could just hand out a references, but screw that noise. Getters and setters are terrible practice and clutters our code and reference handouts are very unsafe and removes any benefit of making members private if we just make them accessible from members anyway, they should've of been public then. Doing this also means that end-users of the type still have to handle the members as if the type was an aggregate. No, let's use our brain a bit more.<br><br>When members are private, we often only want to provide a concise set of ways you can interact with the type. A set of *operations* that can be performed in the type, like how `+` is defined between two `int` objects. We don't have to mess with the bitwise logic of adding the binary representation of the objects together, it just adds the numbers together!<br><br>For our example, we'll define a way to *stringify* our type so we can print it out. Don't forget to make this method public so we can actually use it!

===

<!-- .slide: id="defining-operations-on-types/to-string/1" data-auto-animate -->

```cxx [1: 7-10]
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
<!-- .element: data-id="custom-ops-ex1" -->

notes: That's a better kind of method. It provides a discrete and concrete operation on our type, it produces the string representation of our type.<br><br>To call a method, we access it using the member access (`.`) operator; same as data members, along with parenthesis used to invoke the member and pass any parameters to the method (non in our case).

===

<!-- .slide: id="defining-operations-on-types/to-string/2" data-auto-animate -->

```cxx [1: 17]
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
<!-- .element: data-id="custom-ops-ex1" -->

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/4Yebv5bv8">https://godbolt.org/z/4Yebv5bv8</a></span>

notes: But why do we have to access `to_string()` through the `.` operator? The `.` operator allows the method to access the internals of the type, even private members.

---

<!-- .slide: id="defining-operations-on-types/to-string/the-this-pointer" data-auto-animate -->

### The `this` Pointer

```cxx [1: 5-8]
struct A {

    // data member details...

public:
    auto to_string(A* this) -> std::string {
        return "todo...";
    }
};
```
<!-- .element: data-id="custom-ops-ex1" -->

notes: Implicitly all methods are defined to take a hidden first parameter. This hidden parameter is a pointer to the actual object in memory of which is an instance of the type the method was declared on called `this`.

---

<!-- .slide: id="defining-operations-on-types/to-string/the-this-pointer/elf-files" data-auto-animate -->

<!-- Diagram of ELF file layout -->

notes: This is because C++ does not store the instructions in the **.data** portion of the resulting executable ie. not with constant, static and stack variables but rather in the **.text** section of the executable alongside the rest of the programs instructions.<br><br>As such a method needs to be able to access the other data and members associated with type which is achieved by taking the address of the object and passing that to the method as its first parameter. We can see this effect illustrated by making `to_string()` a free function and passing a `A*` to it.

---

<!-- .slide: id="defining-operations-on-types/to-string/the-this-pointer/free-function" data-auto-animate -->

```cxx [1: 7-9,14]
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

    std::println("{}", to_string(&a));

    return 0;
}
```
<!-- .element: data-id="custom-ops-ex2" -->

notes: In this way, we can think of `.` as a bind operation between a function and an object, prefilling the first parameter with the left argument of `.`.<br><br>*This is not actually true because you cannot store a pointer to a function being accessed via `.` (forbidden by ISO)*

---

<!-- .slide: id="defining-operations-on-types/methods/to-string/impl" data-auto-animate -->

```cxx [1: 7-14|9-13|11-13|11|12|13|21]
struct A {
private:
    char chr = 'a';
    int num = 123;
    float dec = 3.14f;

public:
    auto to_string() -> std::string {
        return std::format(
            "{{ .chr = '{}', .num = {}, .dec = {:.2f} }}",
            this->chr,
            (*this).num,
            dec);
    }
};

auto main() -> int {

    auto a = A { };

    fmt::println("{}", a.to_string());

    return 0;
}
```
<!-- .element: data-id="custom-ops-ex2" -->

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/ne87489bK">https://godbolt.org/z/ne87489bK</a></span>

notes: Since `this` is a pointer we can access any members using regular pointer operators like `*` and `->`. We
can also omit `this` in contexts where there is no naming conflict. Here we can see each technique being used to access the members of `A` and pass them to `std::format()` so we can create our string representation of the type. We can call the method as usual to obtain the string.

===

<!-- .slide: id="defining-operations-on-types/constant-operations/1" data-auto-animate -->

### Constant Operations

```cxx [1: 19|21-23]
struct A {

    // data member details...

public:
    auto to_string() -> std::string { /* ... */ }
};

auto main() -> int {

    auto a const = A { };

    //! error: passing 'const A' as 'this' argument
    //! discards qualifiers
    fmt::println("{}", a.to_string());

    return 0;
}
```
<!-- .element: data-id="custom-ops-ex2" -->

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/K3PGaG4ee">https://godbolt.org/z/K3PGaG4ee</a></span>

notes: Sometimes we want an object to be immutable however this would break the previous example. This is because the member `to_string()` expects to take a first argument of type `A*` not `const A*`. Passing a constant variable to a non constant-accepting parameter cause us to discard the qualifiers (`const`) from the object which would make it mutable. This is an error as we don't want to arbitrarily allow functions to mutate our object!

---

<!-- .slide: id="defining-operations-on-types/constant-operations/2" data-auto-animate -->

### Constant Operations

```cxx [1: 6-7|12,15|13,16]
struct A {

    // data member details...

public:
    // added const after function name and parameters...
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
<!-- .element: data-id="custom-ops-ex2" -->

notes: But this should be able to work because we know `A::to_string()` doesn't modify `a` internals. Luckily we can mark such functions that are constant as `const` to the compiler so they work with immutable instances of the type. This also works with mutable variables as C++ is able to promote them to `const` for the scope of the method.

---

<!-- .slide: id="defining-operations-on-types/constant-operations/free-function" data-auto-animate -->

### Constant Operations

```cxx [1: 6]
struct A {

    // member details...

public:
    auto to_string(const A* this) -> std::string {
        return std::format(
            "{{ .chr = '{}', .num = {}, .dec = {:.2f} }}",
            this->chr,
            (*this).num,
            dec)
    }
};
```
<!-- .element: data-id="custom-ops-ex2" -->

===

<!-- .slide: id="defining-operations-on-types/operator-overloading/1" data-auto-animate -->

### Operator Overloading

notes: C++ also allows us to overload the builtin operators for our own types. This works similar to regular operator overloading however, as we've seen with other methods, the first argument is the `this` pointer
to the type's object in memory. This means that you only have to define the RHS argument.<br><br>Note that you cannot overload all builtin operators, namely, the `.`, `.*` `::`, and `?:` operators. We also cannot introduce new operators.

---

<!-- .slide: id="defining-operations-on-types/operator-overloading/plus/1" data-auto-animate -->

### Operator Overloading

```cxx [1: 8-13]
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
```
<!-- .element: data-id="custom-ops-ex3" -->

notes: Here we've overloaded the `+` operator so we can add two `A`'s numerics values together.

---

<!-- .slide: id="defining-operations-on-types/operator-overloading/plus/2" data-auto-animate -->

### Operator Overloading

```cxx [1: 20]
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
<!-- .element: data-id="custom-ops-ex3" -->

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/hxKfe4P8K">https://godbolt.org/z/hxKfe4P8K</a></span>

notes: And just like that, we can add two `A` objects together!

===

<!-- .slide: id="defining-operations-on-types/friend-functions/1" data-auto-animate -->

### Friend Functions

notes: Sometime we need external types and functions to have access to the internals of our type even if they are unrelated. This is where friend functions and classes come in. For example, we can only overload operators on a type such that the type is always the LHS argument. What if we need an object of a different type to be on LHS? We can make a friend function inside the our type!<br><br>Let's demonstrate this with a different operator, `<`.

---

<!-- .slide: id="defining-operations-on-types/friend-functions/lt" data-auto-animate -->

### Friend Functions

```cxx [1: 8-10|17-19]
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
<!-- .element: data-id="custom-ops-ex4" -->

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/xYP3Gszxz">https://godbolt.org/z/xYP3Gszxz</a></span>

notes: Let's say we want to be able to check if an `int` is less than an `A` object's `A::num` member and we want this to feel as natural as possible for users of our type. We can make an overload for `<` on `A` as a friend function, allowing us to define an `int` as the left argument.
