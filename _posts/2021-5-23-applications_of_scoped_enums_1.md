---
layout: post
date: 2021-5-23
title: Applications of scoped enums (part 1)
excerpt: 'Overloading' return type of a function
---

## For starters

**enum class** is a great feature of C++11 and has many uses. Some of which are:

- type-safe enumerations;
- ability to share the same name between multiple **enum class**'es contrary to plain C enums;
- a finite set of values that can abstract away anything.

Here and below we are going to focus on the latter which is basically the main reason behind enumeration types in general. The idea is that **enum class** being type-checked represents a bare bones set of values - *a type*.
<br>
Beyond that templates have a nice capability of being specialized not only on types but on compile time variables which **enum class** constants are. Other than enums we can use any intergal values but the fact that enum values can be given explicit names comes in handy. With a couple of abstractions we'll try to implement Rust-like enums with attached data.

## Example one (C++14): 'overloading' return type

The enumeration could mean a way for client code to specify an additional parameter to a function which in turn can be specialized on that parameter.
<br>
The simplest example here is a common construction or access point for multiple types - a factory, application settings storage etc.

### example1.hpp:
```c++
/* explicit list of values of our so called type */
enum class overloader
{
    caseA,
    caseB
};

/* set of types which associate with the overloader values
   the key point is that a single type can associate with multiple enum values */
struct typeA
{
    void a();
};
struct typeB
{
    void b();
};

/* mapping between the values and types of overloader */
template<overloader>
struct get_return_t;

/* concrete type mappings */
template<>
struct get_return_t<overloader::caseA> { using type = typeA; };
template<>
struct get_return_t<overloader::caseB> { using type = typeB; };

/* convenience routine */
template<overloader O>
using return_t = typename get_return_t<O>::type;

/* this way the interface is clealy defined and extendable in exactly one place - the enum */
template<overloader O>
auto get() -> return_t<O>;
```

### Somewhere in example1.cpp:
```c++
void typeA::a() {}
void typeB::b() {}

/* the definitions are modular because of the template specialization */
template<>
auto get<overloader::caseA>() -> return_t<overloader::caseA>
{
    auto value = typeA{};
    // ...
    return value;
}
template<>
auto get<overloader::caseB>() -> return_t<overloader::caseB>
{
    auto value = typeB{};
    // ...
    return value;
}
```

### Client code
```c++
/* only an 'overloader' value could be passed */
auto valueA = get<overloader::caseA>();
valueA.a();
auto valueB = get<overloader::caseB>();
valueB.b();
```

In C++14 there is auto return type deduction for functions but that forces us to write the definition in header.

## Example two (C++14): implementing CRTP / a family of mixins

The enumeration could also mean a way to list a family of implementation or mixin types.

### example2.hpp:
```c++
/* for std::is_same */
#include <utility>

/* list of scenarios */
enum class family
{
    caseA,
    caseB
};

/* type specifier for interface methods */
template<family, class>
struct get_method_t;

/* explicit user-defined types for interface methods */
template<class C>
struct get_method_t<family::caseA, C> { using type = int(C::*)(int, char); };
template<class C>
struct get_method_t<family::caseB, C> { using type = void(C::*)(double); };

/* convenience routine */
template<family F, class C>
using method_t = typename get_method_t<F, C>::type;

/* implementation class */
template<family F>
struct implementation;

/* implementation types specifications */
template<>
struct implementation<family::caseA>
{
    auto foo(int, char) -> int;
    /* caseA-specific methods */
};
template<>
struct implementation<family::caseB>
{
    auto foo(double) -> void;
    /* caseB-specific methods */
};

/* common interface for implementation types */
template<family F>
struct interface : implementation<F>
{
    /* enforce method 'foo' in implementation<F> with the specified signature */
    static_assert(std::is_same_v<
        decltype(&implementation<F>::foo),
        method_t<F, implementation<F>>>);
};
```

### Somewhere in example2.cpp:
```c++
auto implementation<family::caseA>::foo(int, char) -> int
{
    /* caseA implementation details */
}
auto implementation<family::caseB>::foo(double) -> void
{
    /* caseB implementation details */
}
```

### Client code
~~~ c++
auto valueA = interface<family::caseA>{};
valueA.foo(42, 'e');

auto valueB = interface<family::caseB>{};
valueB.foo(42.0);
~~~

The common benefit of all these applications is simplification of the interface for client code by using a small number of key entities all defined in one place. Most of these things can be implemented using the regular CRTP, inheritance etc. But in some cases putting a collection of something under an enumeration can help express restrictions in your code.

For more see [part 2]({{ site.baseurl }}/applications_of_scoped_enums_2)
