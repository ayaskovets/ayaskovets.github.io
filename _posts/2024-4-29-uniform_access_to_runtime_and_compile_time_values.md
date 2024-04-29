---
layout: post
date: 2024-4-29
title: Uniform access to runtime and compile time values
excerpt: With just a few lines of code
---

While writing some of the C++20 [code](https://github.com/ayaskovets/matrix_views) related to types optional with compile-time non-type template parameters I stomped upon a rather minor but nonetheless interesting issue.

## Illustrative example

In C++20 we have **std::span** with **T** and **Extent** template parameters, where extent can be either some **size_t** value known at compile time or **std::dynamic_extent** which is basically **(size_t)(-1)** that represents some runtime value.

Suppose we have our own type-container with optionally provided compile-time size:

```c++
template <typename T, std::size_t Size>
class Container {
  ...
};
```

And some deduction guides for the type:

```c++
template <typename T, std::size_t Size>
Container(T (&)[Size]) -> Container<T, Size>;

template <typename T, std::size_t Size>
Container(std::vector<T>) -> Container<T, std::dynamic_extent>;
```

And a member function that returns the size of the container:

```c++
constexpr std::size_t size() {
  if constexpr (Size == std::dynamic_extent) {
    return size_; // some member value that is set in the constructor
  } else {
    return Size;
  }
}
```

This pattern of returning a value in case it is known at compile-time with **if constexpr** is pretty useful and simple but not very handy because **if constexpr** is not an expression that returns value and thus has to be wrapped into a function for each bit of the logic that requires reading the value. We can do better.

## Wrapping the value

A very simplified example of a wrapper that could simplify the code above is something like this:

```c++
template <typename T, T Dynamic, T Value = Dynamic>
class CompileRuntimeValue {
  constexpr static inline bool kStatic = Value != Dynamic;
  constexpr static inline bool kDynamic = !kStatic;

 public:
  constexpr CompileRuntimeValue()
    requires(kStatic)
  = default;

  constexpr CompileRuntimeValue()
    requires(kDynamic)
  = delete;

  constexpr CompileRuntimeValue(T value)
    requires(kDynamic)
  : value_(std::move(value)) {}

  constexpr T operator*() {
    if constexpr {
      return value_;
    } else {
      return Value;
    }
  }

 private:
  struct Empty{};
  [[no_unique_address]] std::conditional_t<kDynamic, T, Empty> value_;
};
```

By wrapping the value into a type with a specific indicator that the value is dynamic allows to write the **if constexpr** once for all of the uses of compile/run time value.

Any **CompileRuntimeValue** object could also be marked with [[no_unique_address]] to hint the compiler that the value stored in the type itself. And applying that to the **Container** class:

```c++
template <typename T, std::size_t Size>
class Container {
  ...
  constexpr std::size_t size() {
    return *size_;
  }
  ...

 private:
  // initialized only in 'runtime' constructors, takes no space when Size is a compile-time value
  [[no_unique_address]] CompileRuntimeValue<std::size_t, std::dynamic_extent, Size> size_;
};
```
