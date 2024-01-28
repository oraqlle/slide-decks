<!-- .slide: id="polymorphism" -->

## Polymorphism

notes: So far we have discussed a lot on how to define our own types and manage resources through types. We are now going to look deeper at some of C++ abstraction mechanisms for types. The main abstraction we are going to look at polymorphism. Polymorphism means something can take many different shapes and change between those shapes. We will look at two forms of polymorphism C++ offers.

===

<!-- .slide: id="polymorphism/parametric-polymorphism/1" data-auto-animate -->

### Parametric Polymorphism

```cpp
                   struct int_array { /* ... */ };
                   struct float_array { /* ... */ };
                   struct A_array { /* ... */ };
```
<!-- .element: class="fragment" data-id="parametric-poly-ex1" -->

notes: Parametric polymorphism allows us to define types and functions in terms of (other) types and functions giving rise to generic definitions for types and functions. Take for example a container type like an array. Instead of defining a different array type for each type of element the array could hold; thus making users define their own array type for their own element types, we could define a *generic* array type that takes some element of type `T` where the actual type of `T` is supplied as an argument or *parameter* to the array when we instantiate an object of the array type.

---

<!-- .slide: id="polymorphism/parametric-polymorphism/2" data-auto-animate -->

### Parametric Polymorphism

```cpp
           type_args(T)
           struct array { /* ... */ };

           auto a = array(int) { 1, 2 /* ... */ };
           auto b = array(float) { 1.2f, -0.02f /* ... */ };
           auto c = array(A) { 
               A { 'a', 1, 0.005f  }, 
               A { 'z', -3, 123.005f  } 
               /* ... */ 
           };
```
<!-- .element: data-id="parametric-poly-ex1" -->

notes: This would mean that `array` is polymorphic over the generic parameter `T`. A type or function doesn't have to be parameterized/polymorphic over just one argument by any number of arguments, even non-type parameters! Take our array example again, if we wanted to define a fixed size array we could create a second value parameter, making array parameterized over `T` and `int N`.

---

<!-- .slide: id="polymorphism/parametric-polymorphism/3" data-auto-animate -->

### Parametric Polymorphism

```cpp
           type_args(type: T, int N)
           struct array { /* ... */ };

           auto a = array(int, 5) { 1, 2, 3, 4, 5 };
           auto b = array(float, 3) { 1.2f, -0.02f, 12364.253f };
           auto c = array(A, 2) { 
               A { 'a', 1, 0.005f  }, 
               A { 'z', -3, 123.005f  }  
           };
```
<!-- .element: data-id="parametric-poly-ex1" -->

notes: This has a large amount of benefits as it allows us to create a generic blueprints for types and functions which than can be instantiated with more concrete types and values. This can be thought of in a similar fashion to regular functions. You describe some instructions of some values however, you don't yet know what those values will be so, specify parameters that represent these values such that when the function is *invoked* (akin to instantiation), the values are passed in. The difference here being that generic parameters are able to change the structure of our program while data parameters cannot.

---

<!-- .slide: id="polymorphism/parametric-polymorphism/templates" data-auto-animate -->

#### Templates

notes: C++ achieves parametric polymorphism using a language construct called *templates*. Template parameters can be type or value parameters. Value parameters must be a *constant expression* ie. a literal type. Templates allow you to specify arguments that the compiler will fill in at compile time when a template is instantiated. Each instantiation of a template function or class is it's own symbol or type respectively.

---

<!-- .slide: id="polymorphism/parametric-polymorphism/templates/functions/1" data-auto-animate -->

##### Function Template

```cpp
                template<typename T, typename U = T>
                auto add(T x, U y) -> decltype(x + y) {
                    return x + y;
                }
```
<!-- .element: data-id="parametric-template-ex1" -->

notes: A template is introduced using the `template` keyword above a class or function signature. Type parameters are introduced using the `typename` keyword and value parameter using the regular `type var_name` signature. You can also give templates a default fallback value which can be another template parameter of the same kind (as you can see with `U`). Here we have a function template `add<>()` which can take two arguments of either the same type or different types. We have also used a metaprogramming feature from C++; `decltype`, which tells the compiler to find the type of an expression. In this case it is the type of the expression `x + y`. This is useful when the types of two parameters is different but will be known to the compiler when the template is instantiated.

---

<!-- .slide: id="polymorphism/parametric-polymorphism/templates/functions/2" data-auto-animate -->

##### Function Template

```cpp
                   template<typename T, typename U = T>
                   auto add(T x, U y) -> decltype(x + y) {
                       return x + y;
                   }

                   add<int, float>(i, f);  // Explicit  
                   add<int>(i, i);         // Explicit + Default Second
                   add<int, char>(i, c);   // Explicit
                   add<>(i, d);            // Explicit template, deduced parameters
                   add(d, f);              // Deduced
                   add(c, f);              // Deduced
```
<!-- .element: data-id="parametric-template-ex1" -->

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/defc1dxcG">https://godbolt.org/z/defc1dxcG</a></span>

---

<!-- .slide: id="polymorphism/parametric-polymorphism/templates/classes/1" data-auto-animate -->

##### Class Template

```cpp
                         template<typename T>
                         struct B {
                             T t;
                         };
```
<!-- .element: data-id="parametric-template-ex2" -->

notes: Class templates work in a very similar manner with the major difference being that templates are usually used to specify the types of data members, not just function parameters. You can also create method templates freely within types with the only rule (outside regular template rules) is the template parameter names cannot collide.

---

<!-- .slide: id="polymorphism/parametric-polymorphism/templates/classes/2" data-auto-animate -->

##### Class Template

```cpp
    template<typename T>
    struct B {
        T t;
    };

    auto a = B<int> { 1 };        // Explicit
    auto const b = B<int> { 2 };  // Whole object constant, even if T is not
    auto c = B<int const> { 0 };  // B::t constant, others mutable
    auto d = B<float> { 3.14f };  // Different types work too!
    auto e = B { 'a' };           // Deduced type argument (T = char)
```
<!-- .element: data-id="parametric-template-ex2" -->

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/qWdeqj5c7">https://godbolt.org/z/qWdeqj5c7</a></span>

---

<!-- .slide: id="polymorphism/parametric-polymorphism/concepts-and-constraints/1" data-auto-animate -->

#### Concepts and Constraints

```cpp
                 template<typename T, typename U = T>
                 auto add(T x, U y) -> decltype(x + y) {
                     return x + y;
                 }

                 add(3, "abc"s);  // Compiler Error!
```

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/vKaYvd3KT">https://godbolt.org/z/vKaYvd3KT</a></span>

notes: You may notice from our *Function Template* example that we are adding objects of various types together but what if `+` was not defined between those types. **Error!**. The compiler is clever but it cannot just generate an `+` overload for us, what would that even look like for and `int` and a string (JavaScript might have some ideas|JavaScript has entered the chat)? No, it must already be defined which is not the case for `int` and `std::string`. How can we constrain `add<>()` such that it only allowed *arithmetic* types? C++ offers a type constraint mechanism as the `requires` clauses. This allow us to define; well, requirements of template arguments. We can use a requires-clause alongside some C++ meta-programming utilities.

---

<!-- .slide: id="polymorphism/parametric-polymorphism/concepts-and-constraints/2" data-auto-animate -->

#### Concepts and Constraints

```cpp
                 template<typename T, typename U = T>
                        requires std::is_arithmetic_v<T>
                              && std::is_arithmetic_v<U>
                 auto add(T x, U y) -> decltype(x + y) {
                     return x + y;
                 }

                 add(3, "abc"s);  // Compiler Error!
```

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/">https://godbolt.org/z/</a></span>

notes: This doesn't fix the issue but it does give a bit better insight from the compiler about what went wrong. We can formalise requirements into a symbol known as a *concept*. Concepts are named requirements and can be placed directly on template types.

---

<!-- .slide: id="polymorphism/parametric-polymorphism/concepts-and-constraints/3" data-auto-animate -->

#### Concepts and Constraints

```cpp
              template<typename T>
              concept Arithmetic = std::is_arithmetic_v<T>;
  
              template<Arithmetic T, Arithmetic U = T>
              auto add(T x, U y) -> decltype(x + y) {
                  return x + y;
              }
  
              add(3, "abc"s);  // Compiler Error!
```

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/9KEcPx63n">https://godbolt.org/z/9KEcPx63n</a></span>

notes: Concepts can also be used to constrain variables introduced with the `auto` keyword. This is useful in ensuring that the return type of a function meets that concept.

---

<!-- .slide: id="polymorphism/parametric-polymorphism/concepts-and-constraints/4" data-auto-animate -->

#### Concepts and Constraints

```cpp
              template<typename T>
              concept Arithmetic = std::is_arithmetic_v<T>;

              Arithmetic auto i = 3;           // Ok
              Arithmetic auto f = 3.14f;       // Ok
              Arithmetic auto c = 'z';         // Ok
              Arithmetic auto d = 13253.5676;  // Ok
              Arithmetic auto s = "abc"s;      // Error!
```

<span class="fragment" style="font-size: large;">See it on Godbolt ⚡: <a href="https://godbolt.org/z/E5Ye37ve8">https://godbolt.org/z/E5Ye37ve8</a></span>

notes: This doesn't fix the issue but it does give a bit better insight from the compiler about what went wrong. We can formalise requirements into a symbol known as a *concept*. Concepts are named requirements and can be placed directly on template types.
