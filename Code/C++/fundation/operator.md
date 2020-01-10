# [Why override operator()?](https://stackoverflow.com/questions/317450/why-override-operator)



One of the primary goal when overloading operator() is to create a functor. A functor acts just like a function, but it has the advantages that it is stateful, meaning it can keep data reflecting its state between calls.

Here is a simple functor example :

```
struct Accumulator
{
    int counter = 0;
    int operator()(int i) { return counter += i; }
}
...
Accumulator acc;
cout << acc(10) << endl; //prints "10"
cout << acc(20) << endl; //prints "30"
```

Functors are heavily used with generic programming. Many STL algorithms are written in a very general way, so that you can plug-in your own function/functor into the algorithm. For example, the algorithm std::for_each allows you to apply an operation on each element of a range. It could be implemented something like that :

```
template <typename InputIterator, typename Functor>
void for_each(InputIterator first, InputIterator last, Functor f)
{
    while (first != last) f(*first++);
}
```

You see that this algorithm is very generic since it is parametrized by a function. By using the operator(), this function lets you use either a functor or a function pointer. Here's an example showing both possibilities :

```
void print(int i) { std::cout << i << std::endl; }
...    
std::vector<int> vec;
// Fill vec

// Using a functor
Accumulator acc;
std::for_each(vec.begin(), vec.end(), acc);
// acc.counter contains the sum of all elements of the vector

// Using a function pointer
std::for_each(vec.begin(), vec.end(), print); // prints all elements
```

------

Concerning your question about operator() overloading, well yes it is possible. You can perfectly write a functor that has several parentheses operator, as long as you respect the basic rules of method overloading (e.g. overloading only on the return type is not possible).