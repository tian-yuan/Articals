# <T> T的含义

在读java源代码的时候，我们经常会看到类似这样的定义：

```
// 摘自RestTemplate.java
public <T> T getForObject(String url, Class<T> responseType, Object... urlVariables) throws RestClientException
```

那么这个`<T> T`是什么含义呢？
第二个`T`很好理解，表示返回值类型；
而第一个`<T>`的作用是声明这个方法是个*泛型方法*。

看javadoc的关于`泛型方法`的解释：

> Generic methods are methods that introduce their own type parameters. This is similar to declaring a generic type, but the type parameter's scope is limited to the method where it is declared. Static and non-static generic methods are allowed, as well as generic class constructors.



大概含义是，泛型方法声明了本方法含有`类型参数`（比如`T`）。

有个好方法帮助理解，把这个`<T>`去掉，会发生什么呢？

![img](https://upload-images.jianshu.io/upload_images/54256-45573697e8c77aab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/262)

会报*找不到T的定义*的错误！
原来，编译器并不会识别这个`T`是泛型类型的占位符，还可能有其他的类名也叫`T`（`T`并不是保留字，没人规定不可以）！
这个`<T>`就会告诉编译器，现在声明`T`是一个范型类型的占位符，而不是其他东东（比如类名）！

现在是不是恍然大悟了呢？

再回想下我们平时经常写的带有泛型的类：

```
class Base<T> {}
```

这里的`<T>`, 作用也一样，声明`T`是个泛型类型的占位符。这么定义的类，叫做`泛型类`。

类比下`c++`关于模板的定义：

```
template< typename T>
void T get(T a);
```

> Java的这个`<T>`就相当于`template<template T>`，只是理解起来没有那么直观~

