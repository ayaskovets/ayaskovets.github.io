---
layout: post
date: 2021-5-25
title: Applications of scoped enums (part 2)
excerpt: Some abstractions to mimic pattern matching in plain old C++
---

## Example three (C++14): pattern matching

The enumeration could also be a means to pass runtime info to to type level with minimal overhead.
<br>
The both previous examples are focused on compile-time values, but with by adding a couple of runtime checks we can achieve more by taking some inspiration from a common pattern functional programming idea of implementing functions in terms of pattern matching.

### atype.hpp:
```c++
/* let's pretend that this is a type we want to match on */
using type = int;

/* pattern matching is basically determining the state of a variable
   we'll need a list of possible states we are going to handle */
enum class int_state
{
    positive,
    negative,

    other
};

/* helper utility to associate types with state enums */
template<typename T>
struct get_state_t;

/* associate int with int_state */
template<>
struct get_state_t<int> { using type = int_state; };

/* convenience routine */
template<typename T>
using state_t = typename get_state_t<T>::type;

/* mapper from type value to its state */
template<typename T>
auto stateof(const T& value) -> state_t<T>;
```

### **atype.cpp**:
```c++
/* definition of the state mapper for our type */
template<>
auto stateof<int>(const int &value) -> state_t<int>
{
    return value > 0 ? int_state::positive
         : value < 0 ? int_state::negative
         : int_state::other;
}
```

### example3.hpp:
```c++
/* an interface visible to client code */
auto magic(int) -> int;
```

### Somewhere in example3.cpp:
```c++
/* the real magic implementation */
template<int_state>
auto true_magic(int i) -> int;

/* implement the function in terms of pattern matching */
template<>
auto true_magic<int_state::positive>(int i) -> int
{
    return i;
}
template<>
auto true_magic<int_state::negative>(int i) -> int
{
    return -i;
}
template<>
auto true_magic<int_state::other>(int i) -> int
{
    return 0;
}

/* interface implementation via simple switch
   that can be auto-generated, for each unhandled case
   compiler generates a warning */
auto magic(int i) -> int
{
    switch (stateof(i))
    {
    case int_state::positive: return true_magic<int_state::positive>(i);
    case int_state::negative: return true_magic<int_state::negative>(i);
    default: return true_magic<int_state::other>(i);
    }
}
```

That's for sure a lot of boilerplate for such a simple function but the overall pattern is very simple - by doing a few runtime checks we can acquire additional information about runtime values and carry this information to type level. This is especially useful when objects are immutable because the state can be calculated on construction.

All of the examples (except the one with **static_assert**) do not require modern language features and by sacrificing some type-safety can be used even with C++03-supporting compiler.
