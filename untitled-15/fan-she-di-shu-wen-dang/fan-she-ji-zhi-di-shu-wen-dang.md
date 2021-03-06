# 反射机制-地鼠文档

## 1. 前言 <a id="c08jg2"></a>

个人觉得，反射讲得最透彻的还是官方博客。官方博客略显晦涩，多读几遍就慢慢理解了。  
本文既是学习笔记，也是总结。  
官方博客地址：[https://blog.golang.org/laws-of-reflection](https://blog.golang.org/laws-of-reflection)

## 2. 反射概念 <a id="5ppzze"></a>

官方对此有个非常简明的介绍，两句话耐人寻味：

1. 反射提供一种让程序检查自身结构的能力
2. 反射是困惑的源泉

第1条，再精确点的描述是“反射是一种检查interface变量的底层类型和值的机制”。  
第2条，很有喜感的自嘲，不过往后看就笑不出来了，因为你很可能产生困惑.

想深入了解反射，必须深入理解类型和接口概念。下面开始复习一下这些基础概念。

### 2.1 关于静态类型 <a id="dgkko8"></a>

你肯定知道Go是静态类型语言，比如”int”、”float32”、”\[\]byte”等等。每个变量都有一个静态类型，且在编译时就确定了。  
那么考虑一下如下一种类型声明:

```text
type Myint int

var i int
var j Myint
```

Q: i 和j 类型相同吗？  
A：i 和j类型是不同的。 二者拥有不同的静态类型，没有类型转换的话是不可以互相赋值的，尽管二者底层类型是一样的。

### 2.2 特殊的静态类型interface <a id="c41flw"></a>

interface类型是一种特殊的类型，它代表方法集合。 它可以存放任何实现了其方法的值。

经常被拿来举例的是io包里的这两个接口类型：

```text

type Reader interface {
    Read(p []byte) (n int, err error)
}


type Writer interface {
    Write(p []byte) (n int, err error)
}
```

任何类型，比如某struct，只要实现了其中的Read\(\)方法就被认为是实现了Reader接口，只要实现了Write\(\)方法，就被认为是实现了Writer接口，不过方法参数和返回值要跟接口声明的一致。

接口类型的变量可以存储任何实现该接口的值。

### 2.3 特殊的interface类型 <a id="6p5nmk"></a>

最特殊的interface类型为空interface类型，即`interface {}`，前面说了，interface用来表示一组方法集合，所有实现该方法集合的类型都被认为是实现了该接口。那么空interface类型的方法集合为空，也就是说所有类型都可以认为是实现了该接口。

一个类型实现空interface并不重要，重要的是一个空interface类型变量可以存放所有值，记住是所有值，这才是最最重要的。 这也是有些人认为Go是动态类型的原因，这是个错觉。

### 2.4 interface类型是如何表示的 <a id="e4lmpd"></a>

前面讲了，interface类型的变量可以存放任何实现了该接口的值。还是以上面的`io.Reader`为例进行说明，`io.Reader`是一个接口类型，`os.OpenFile()`方法返回一个`File`结构体类型变量，该结构体类型实现了`io.Reader`的方法，那么`io.Reader`类型变量就可以用来接收该返回值。如下所示：

```text
var r io.Reader
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
if err != nil {
    return nil, err
}
r = tty
```

那么问题来了。  
Q： r的类型是什么？  
A: r的类型始终是`io.Reader`interface类型，无论其存储什么值。

Q：那`File`类型体现在哪里？  
A：r保存了一个\(value, type\)对来表示其所存储值的信息。 value即为r所持有元素的值，type即为所持有元素的底层类型

Q：如何将r转换成另一个类型结构体变量？比如转换成`io.Writer`  
A：使用类型断言，如`w = r.(io.Writer)`. 意思是如果r所持有的元素如果同样实现了io.Writer接口,那么就把值传递给w。

## 3. 反射三定律 <a id="25li8u"></a>

前面之所以讲类型，是为了引出interface，之所以讲interface是想说interface类型有个\(value，type\)对，而反射就是检查interface的这个\(value, type\)对的。具体一点说就是Go提供一组方法提取interface的value，提供另一组方法提取interface的type.

官方提供了三条定律来说明反射，比较清晰，下面也按照这三定律来总结。

反射包里有两个接口类型要先了解一下.

* `reflect.Type` 提供一组接口处理interface的类型，即（value, type）中的type
* `reflect.Value`提供一组接口处理interface的值,即\(value, type\)中的value

下面会提到反射对象，所谓反射对象即反射包里提供的两种类型的对象。

* `reflect.Type` 类型对象
* `reflect.Value`类型对象

### 3.1 反射第一定律：反射可以将interface类型变量转换成反射对象 <a id="gdj19h"></a>

下面示例，看看是如何通过反射获取一个变量的值和类型的：

```text
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x float64 = 3.4
    t := reflect.TypeOf(x)  
    fmt.Println("type:", t)

    v := reflect.ValueOf(x) 
    fmt.Println("value:", v)
}
```

程序输出如下：

```text
type: float64
value: 3.4
```

注意：反射是针对interface类型变量的，其中`TypeOf()`和`ValueOf()`接受的参数都是`interface{}`类型的，也即x值是被转成了interface传入的。

除了`reflect.TypeOf()`和`reflect.ValueOf()`，还有其他很多方法可以操作，本文先不过多介绍，否则一不小心会会引起困惑。

### 3.2 反射第二定律：反射可以将反射对象还原成interface对象 <a id="bdcv6n"></a>

之所以叫’反射’，反射对象与interface对象是可以互相转化的。看以下例子：

```text
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x float64 = 3.4

    v := reflect.ValueOf(x) 

    var y float64 = v.Interface().(float64)
    fmt.Println("value:", y)
}
```

对象x转换成反射对象v，v又通过Interface\(\)接口转换成interface对象，interface对象通过.\(float64\)类型断言获取float64类型的值。

### 3.3 反射第三定律：反射对象可修改，value值必须是可设置的 <a id="aoyr0"></a>

通过反射可以将interface类型变量转换成反射对象，可以使用该反射对象设置其持有的值。在介绍何谓反射对象可修改前，先看一下失败的例子：

```text
package main

import (
    "reflect"
)

func main() {
    var x float64 = 3.4
    v := reflect.ValueOf(x)
    v.SetFloat(7.1) 
}
```

如下代码，通过反射对象v设置新值，会出现panic。报错如下：

```text
panic: reflect: reflect.Value.SetFloat using unaddressable value
```

错误原因即是v是不可修改的。

反射对象是否可修改取决于其所存储的值，回想一下函数传参时是传值还是传址就不难理解上例中为何失败了。

上例中，传入reflect.ValueOf\(\)函数的其实是x的值，而非x本身。即通过v修改其值是无法影响x的，也即是无效的修改，所以golang会报错。

想到此处，即可明白，如果构建v时使用x的地址就可实现修改了，但此时v代表的是指针地址，我们要设置的是指针所指向的内容，也即我们想要修改的是`*v`。 那怎么通过v修改x的值呢？

`reflect.Value`提供了`Elem()`方法，可以获得指针向指向的`value`。看如下代码：

```text
package main

import (
"reflect"
    "fmt"
)

func main() {
    var x float64 = 3.4
    v := reflect.ValueOf(&x)
    v.Elem().SetFloat(7.1)
    fmt.Println("x :", v.Elem().Interface())
}
```

输出为：

```text
x : 7.1
```

### 4. 总结 <a id="dve2jf"></a>

结合官方博客及本文，至少可以对反射理解个大概，还有很多方法没有涉及。  
对反射的深入理解，个人觉得还需要继续看的内容：

* 参考业界，尤其是开源框架中是如何使用反射的
* 研究反射实现原理，探究其性能优化的手段

文档更新时间: 2020-08-08 21:59   作者：kuteng

