# Go的类型断言解析

susufufu · 2017-10-20 08:06:27 · 1261 次点击 · 预计阅读时间 1 分钟 · 7分钟之前 开始浏览    

这是一个创建于 2017-10-20 08:06:27 的文章，其中的信息可能已经有所发展或是发生改变。

经常地我们对一个接口值的动态类型是不确定的，如方法的形参为接口类型时，此时就需要检验它是否符合我们需要的类型。
类型断言是一个使用在接口值上的操作。

如果对Golang的接口和接口值的概念不熟悉，看这里：[Go的接口总结](http://www.cnblogs.com/susufufu/p/7353312.html)
**断言类型的语法：**x.(T)，这里x表示一个接口的类型，T表示一个类型（也可为接口类型）。
一个类型断言检查一个接口对象x的动态类型是否和断言的类型T匹配。

类型断言分两种情况：
**第一种**，如果断言的类型T是一个具体类型，类型断言x.(T)就检查x的动态类型是否和T的类型相同。

- 如果这个检查成功了，类型断言的结果是一个类型为T的对象，该对象的值为接口变量x的动态值。换句话说，具体类型的类型断言从它的操作对象中获得具体的值。
- 如果检查失败，接下来这个操作会抛出panic，除非用两个变量来接收检查结果，如：f, ok := w.(*os.File)

**第二种**，如果断言的类型T是一个接口类型，类型断言x.(T)检查x的动态类型是否满足T接口。

- 如果这个检查成功，则检查结果的接口值的动态类型和动态值不变，但是该接口值的类型被转换为接口类型T。换句话说，对一个接口类型的类型断言改变了类型的表述方式，改变了可以获取的方法集合（通常更大），但是它保护了接口值内部的动态类型和值的部分。
- 如果检查失败，接下来这个操作会抛出panic，除非用两个变量来接收检查结果，如：f, ok := w.(io.ReadWriter)

**注意：**

- 如果断言的操作对象x是一个nil接口值，那么不论被断言的类型T是什么这个类型断言都会失败。
- 我们几乎不需要对一个更少限制性的接口类型（更少的方法集合）做断言，因为它表现的就像赋值操作一样，除了对于nil接口值的情况。

**示例代码：**

　　

```
//===接口=====
type Tester interface {
	getName()string
}
type Tester2 interface {
	printName()
}
//===Person类型====
type Person struct {
	name string
}
func (p Person)getName() string {
	return p.name
}
func (p Person) printName() {
	fmt.Println(p.name)
}
//============
func main() {
	var t Tester
	t = Person{"xiaohua"}
	check(t)
}
func check(t Tester)  {
    //第一种情况
	if f, ok1 := t.(Person);ok1 {
		fmt.Printf("%T\n%s\n",f,f.getName())
	}
    //第二种情况
	if t, ok2 := t.(Tester2);ok2 {  //重用变量名t（无需重新声明）
		check2(t) //若类型断言为true，则新的t被转型为Tester2接口类型，但其动态类型和动态值不变
	}
}
func check2(t Tester2)  {
	t.printName()
}
```

 执行结果：

main.Person
xiaohua
xiaohua