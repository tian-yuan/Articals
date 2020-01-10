### Folly likely



```
#if __GNUC__
#define FOLLY_DETAIL_BUILTIN_EXPECT(b, t) (__builtin_expect(b, t))
#else
#define FOLLY_DETAIL_BUILTIN_EXPECT(b, t) b
#endif

#define FOLLY_LIKELY(x) FOLLY_DETAIL_BUILTIN_EXPECT((x), 1)
#define FOLLY_UNLIKELY(x) FOLLY_DETAIL_BUILTIN_EXPECT((x), 0)

//  Un-namespaced annotations

#undef LIKELY
#undef UNLIKELY

#if defined(__GNUC__)
#define LIKELY(x) (__builtin_expect((x), 1))
#define UNLIKELY(x) (__builtin_expect((x), 0))
#else
#define LIKELY(x) (x)
#define UNLIKELY(x) (x)
#endif
```

这个指令是gcc引入的，作用是**允许程序员将最有可能执行的分支告诉编译器**。这个指令的写法为：`__builtin_expect(EXP, N)`。
意思是：EXP==N的概率很大。

`__builtin_expect()` 是 GCC (version >= 2.96）提供给程序员使用的，目的是将“分支转移”的信息提供给编译器，这样编译器可以对代码进行优化，以减少指令跳转带来的性能下降。
 `__builtin_expect((x),1)`表示 x 的值为真的可能性更大；
 `__builtin_expect((x),0)`表示 x 的值为假的可能性更大。
 也就是说，使用`likely()`，执行 if 后面的语句的机会更大，使用 `unlikely()`，执行 else 后面的语句的机会更大。通过这种方式，编译器在编译过程中，会将可能性更大的代码紧跟着前面的代码，从而减少指令跳转带来的性能上的下降。