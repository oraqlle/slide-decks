<!-- .slide: id="polymorphism/subtyping-polymorphism" data-auto-animate -->

### Subtype Polymorphism

<span class="fragment" style="font-size: xx-large;">`int32_t` <: `int64_t` | subtype, not strictly a subclass</span>

<span class="fragment" style="font-size: xx-large;">`std::basic_ostream` <: & <- `std::basic_ios` | subtype and subclass</span>

notes: The other type of polymorphism C++ allows what is known as Subtype Polymorphism or Subtyping. This is the typical polymorphism you learn about in Object Oriented Programming (OOP). A subtype is a datatype that is related to another datatype; a *supertype*, by a notion of substitutability, meaning that any functions or subroutines that can operate on the supertype can also operate on the subtype. This subtyping relationship of types is often expressed using subclassing (inheritance) in which a child class (subtype) *inherits* the properties (method signatures) and data members of a base class (supertypes). This distinction between subclass based subtyping vs just subtyping is important as both concepts are orthogonal but often intersect in *"OOP"* languages. A subtype is one in which you can simply substitute it in place of the super type. For example the type `int32_t` in C++ is a subtype of `int64_t` meaning anywhere you need a `int64_t` you can supply a `int64_t` as any valid operation on the later will be valid on the former however, one is not derived from the other. In other words, subtyping is more about matching the *interface* of a type such that if one type supports all operations of another type, it is a subtype of the other type. This is more closely related to interface inheritance than virtual (traditional) inheritance found in OOP languages. Subclasses describes implementation based inheritance in which not only the signatures (interface) of a type are defined on another type but the implementation can be shared using a *virtual, method lookup table* or *vtable* and dynamic dispatch.

---

<!-- .slide: id="polymorphism/subtyping-polymorphism/abstract-classes" data-auto-animate -->

#### Abstract Classes

```cpp [1:]
struct Stringifiable {
    virtual auto to_string() -> std::string = 0;
};
```
<!-- .element: class="fragment" data-id="subtyping-poly-ex1" -->

notes: Abstract classes are introduced into C++ by making the member functions of a type purely `virtual`. This is done with the `virtual` keyword along the *pure specifier* `= 0` at the end of the method. The `virtual` keyword specifies that a method supports dynamic dispatch ie. the implementation of the method is looked up through a classes vtable at runtime (also known as late binding) while the *pure specifier* denotes that the implementation must be written by inheriting classes.

---

<!-- .slide: id="polymorphism/subtyping-polymorphism/abstract-classes/inheritance" data-auto-animate -->

##### Inheritance

```cpp [1: 5|10-13|34|35|36|37|39-41|18|20-22|24-26|43|44|46-48]
struct Stringifiable {
    virtual auto to_string() -> std::string = 0;
};

struct A : public Stringifiable {

    A(int n)
    : n { n } { }

    auto to_string() -> std::string {
        return std::format("A {{ {} }}", n);
    }

private:
    int n;
};

struct B : public A {

    B(int n, float f)
    : A { n }, f { f } 
    { }

    auto to_string() -> std::string {
        return std::format("B {{ {}, {} }}", A::to_string(), f);
    }

private:
    float f;
};

auto main() -> int {

    auto a = A { 123 };
    // Stringifiable s{};   // Error!
    Stringifiable& r = a;   // References work
    Stringifiable* p = &a;  // and pointers!
    
    fmt::println("a: {}", a.to_string());
    fmt::println("r: {}", r.to_string());
    fmt::println("p: {}", p->to_string());

    auto b = B { 456, 3.14f };
    p = &b;  // Pointers can be rebound

    fmt::println("b: {}", b.to_string());
    fmt::println("r: {}", r.to_string());
    fmt::println("p: {}", p->to_string());

    return 0;
}
```
<!-- .element: data-id="subtyping-poly-ex1" -->

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/ee5fv7hPd">https://godbolt.org/z/ee5fv7hPd</a></span>

notes: To inherit from a class you specify a `:` followed a class name(s), each prefixed with an access specifier (which defaults to `private`). Yes C++ supports multiple inheritance. It is also possible to inherit from a non-abstract classes as well which won't generate a vtable or require dynamic dispatch but rather will just compose the type together. We can call the superclass constructor(s) in the member initialiser list of the subclass's constructors in order to properly construct the superclass class(es). The superclass constructor(s) must appear before our subclass members in the initialiser list. To override a method you simply have to define a method of the same signature (return type, parameters, cv-quals etc.). You can keep the method pure-virtual by using the pure specifier again. You may also explicitly denote you are overriding a method by specifying it is `virtual` (again) and adding the `override` keyword at the end of the function signature however, it is important to note that you cannot use the trailing-return-type syntax with the `override` keyword.<br><br>Construction of objects from concrete classes works like any other class or type construction with the only difference being that member invocations on virtual methods are dynamically dispatched instead of statically. Dynamic dispatch means that the address of the correct method implementation is looked up using the vtable of a class at runtime while static dispatch means there is only a single possible implementation of a method and it is known at compile time thus the compiler can bake the methods address directly at the call site.<br><br>Abstract classes cannot be constructed directly but you can create pointers and references to abstract classes such that they can be bound to a subclass object. Again, method invocation is looked up using the class's vtable.<br><br>Note that you cannot have a template member function that is also virtual as it would be impossible for the compiler to determine the number of slots the vtable would need for each parameterization of the member.

---

<!-- .slide: id="polymorphism/subtyping-polymorphism/access-specifiers" -->

#### Access Specifiers

| Specifiers   | Same Class | Derived Class | Outside Class |
|--------------|------------|---------------|---------------|
| `public`     | Yes        | Yes           | Yes           |
| `private`    | Yes        | No            | No            |
| `protected`  | Yes        | Yes           | No            |

notes: Here's a second look at the access specifiers table. Note that while we can control the access to regions or each member of a class, we can also control the access to inherited class. The rules work in the same way, based on the precedence of each access type. `public` is the lowest and means the that the inheritance is public, ie. the superclass can be publicly access (based on it's own access rules) from the subclass. `protected` is in the middle and means that the superclass can be seen by subclasses of the inheriting class but the superclass is private to the outside would from and subclasses. `private` means the superclass is completely hidden from the outside when accessed through the subclass.

---

<!-- .slide: id="polymorphism/subtyping-polymorphism/access-specifiers" -->

#### Access Specifiers

<!-- diagram with three members of the different access types showing public can interact with outside in derived, protected can interact inside in derived and private is purely in base class -->

---

<!-- .slide: id="polymorphism/subtyping-polymorphism/abstract-classes/virtual-tables" -->

#### Virtual Tables

<!-- diagram of the above classes' data layout and pointer to vtable and little animation showing how calling gets resolved -->

notes: So what is this virtual table that I keep speaking of. A vtable is a map between a class method calls and the implementation for said type. Each class (that has virtual members) gets one vtable with class objects storing a pointer to said vtable. Where the vtable is stored is up to the compiler but sometimes it is stored statically as part of the programs encoded data, sometimes it is placed somewhere within the class itself, usually before or after the region of memory allocated for data members if it is inlined into the class itself. Why do we need a vtable? Declaring a method `virtual` means that it's implementation *could* be defined in another class, namely in a subclass or superclass. Thus any class within the inheritance hierarchy needs a way to find and load the correct method implementation, it is this information that is stored in the vtable. When a class calls a virtual method, the vtable gets loaded into the CPU and queried for the address of the correct implementation for that type.

---

<!-- .slide: id="polymorphism/subtyping-polymorphism/the-cost-of-virtual-tables" -->

##### The Cost of Virtual Tables

notes: As we've seen demonstrated, the use of dynamic dispatch for subclass polymorphism adds a lot more information to a type and even more runtime work in terms of lookup of methods. This is a pretty high cost for abstraction meaning that virtual functions and dynamic dispatch are solemnly used in C++ as the performance cost is too high in the situations you would use C++ as well as being to unpredictable. Instead, simple type composition (ie. non virtual or abstract classes) using the inheritance syntax, templates or a combination of the two are chosen in favour. This tradeoff usually means longer compile times but much quicker runtimes. There is even an entire design pattern in C++ can *Curiously Recurring Template Pattern* which uses the inheritance mechanism on templated superclasses to create interfaces in which static or regular dispatch is used. An entire sublibrary of the C++ standard library (Ranges) uses this exact technique as a fundamental pillar of it's abstraction model.

<!-- --- -->

<!-- .slide: id="polymorphism/subtyping-polymorphism/the-diamond-problem" -->

<!-- #### The Diamond Problem -->

<!-- Virtual inheritance -->
