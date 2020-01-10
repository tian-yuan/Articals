#### std::enable_if 的使用



```
#include <iostream>
#include <type_traits>
 
class E { public: template<class T> E(T&&) { } };

class A {};
class B : public A {};
class C {};
class D { public: operator C() { return c; }  C c; };

template<typename T, 
    typename = typename std::enable_if<std::is_convertible<A*, B*>::value>::type>
void func(T value) {
    std::cout << "value : " << value << std::endl;
}
 
int main() 
{
    bool b2a = std::is_convertible<B*, A*>::value;
    bool a2b = std::is_convertible<A*, B*>::value;
    bool b2c = std::is_convertible<B*, C*>::value;
    bool d2c = std::is_convertible<D, C>::value;
 
    // A Perfect Forwarding constructor make the class 'convert' from everything
 
    bool everything2e = std::is_convertible<A, E>::value; //< B, C, D, etc
 
    std::cout << std::boolalpha;
 
    std::cout << b2a << '\n';
    std::cout << a2b << '\n';
    std::cout << b2c << '\n';
    std::cout << d2c << '\n';
    std::cout << '\n';
    std::cout << everything2e << '\n';
}

```

由于 A* 不能转化为 B*，所以上面编译会失败：

```
main.cpp:12:77: error: 'type' in 'struct std::enable_if<false, void>' does not name a type
     typename = typename std::enable_if<std::is_convertible<A*, B*>::value>::type>
```



做如下修改即可编译成功：



```
template<typename T, 
    typename = typename std::enable_if<std::is_convertible<A*, B*>::value>::type>
void func(T value) {
    std::cout << "value : " << value << std::endl;
}
====>
template<typename T, 
    typename = typename std::enable_if<std::is_convertible<B*, A*>::value>::type>
void func(T value) {
    std::cout << "value : " << value << std::endl;
}
```

