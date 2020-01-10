#### true_type

```
/// integral_constant
  template<typename _Tp, _Tp __v>
    struct integral_constant
    {
      static constexpr _Tp                  value = __v;
      typedef _Tp                           value_type;
      typedef integral_constant<_Tp, __v>   type;
      constexpr operator value_type() const noexcept { return value; }
#if __cplusplus > 201103L
#define __cpp_lib_integral_constant_callable 201304
      constexpr value_type operator()() const noexcept { return value; }
#endif
    };
  template<typename _Tp, _Tp __v>
    constexpr _Tp integral_constant<_Tp, __v>::value;
  /// The type used as a compile-time boolean with true value.
  typedef integral_constant<bool, true>     true_type;
  /// The type used as a compile-time boolean with false value.
  typedef integral_constant<bool, false>    false_type;
```

#### value_type

Well the *function* `value_type()` is not really a function with that name. Indeed, the definition of `integral_constant` looks like this:

```cpp
template <class T, T v>
struct integral_constant {
    // ...
    typedef T value_type;
    constexpr operator value_type() const noexcept { return value; }
};
```

Notice that `value_type` is actually a `typedef` for the template parameter `T` (`int`, in OP's example). In addition there's a conversion operator that converts from `integral_constant<T, v>`to `T`. It's implicitly called like this

```cpp
int i = one_o; // same as int i = 1;
```

To call it explicitly you need to use the `operator` keyword and the right type like this:

```cpp
one_o.operator int();
```

which, thanks to the `typedef` member, is also equivalent to

```cpp
one_o.operator one_t::value_type();
```



#### is_array

```
template<class T>
struct is_array : std::false_type {};
 
template<class T>
struct is_array<T[]> : std::true_type {};
 
template<class T, std::size_t N>
struct is_array<T[N]> : std::true_type {};
```



#### example

```
#include <iostream>
#include <type_traits>
 
class A {};
 
int main() 
{
    std::cout << std::boolalpha;
    std::cout << std::is_array<A>::value << '\n';
    std::cout << std::is_array<A[]>::value << '\n';
    std::cout << std::is_array<A[3]>::value << '\n';
    std::cout << std::is_array<float>::value << '\n';
    std::cout << std::is_array<int>::value << '\n';
    std::cout << std::is_array<int[]>::value << '\n';
    std::cout << std::is_array<int[3]>::value << '\n';
}
```

主要靠类型推导

#### is_bounded_array

```
template<class T>
struct is_bounded_array: std::false_type {};
 
template<class T, std::size_t N>
struct is_bounded_array<T[N]> : std::true_type {};
```

