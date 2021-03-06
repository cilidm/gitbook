# 数组Array

## 1. 数组Array <a id="&#x6570;&#x7EC4;array"></a>

Golang Array和以往认知的数组有很大不同。

```text
    1. 数组：是同一种数据类型的固定长度的序列。
    2. 数组定义：var a [len]int，比如：var a [5]int，数组长度必须是常量，且是类型的组成部分。一旦定义，长度不能变。
    3. 长度是数组类型的一部分，因此，var a[5] int和var a[10]int是不同的类型。
    4. 数组可以通过下标进行访问，下标是从0开始，最后一个元素下标是：len-1
    for i := 0; i < len(a); i++ {
    }
    for index, v := range a {
    }
    5. 访问越界，如果下标在数组合法范围之外，则触发访问越界，会panic
    6. 数组是值类型，赋值和传参会复制整个数组，而不是指针。因此改变副本的值，不会改变本身的值。
    7.支持 "=="、"!=" 操作符，因为内存总是被初始化过的。
    8.指针数组 [n]*T，数组指针 *[n]T。
```

### 1.1.1. 数组初始化： <a id="&#x6570;&#x7EC4;&#x521D;&#x59CB;&#x5316;&#xFF1A;"></a>

#### 一维数组： <a id="&#x4E00;&#x7EF4;&#x6570;&#x7EC4;&#xFF1A;"></a>

```text
    全局：
    var arr0 [5]int = [5]int{1, 2, 3}
    var arr1 = [5]int{1, 2, 3, 4, 5}
    var arr2 = [...]int{1, 2, 3, 4, 5, 6}
    var str = [5]string{3: "hello world", 4: "tom"}
    局部：
    a := [3]int{1, 2}           // 未初始化元素值为 0。
    b := [...]int{1, 2, 3, 4}   // 通过初始化值确定数组长度。
    c := [5]int{2: 100, 4: 200} // 使用索引号初始化元素。
    d := [...]struct {
        name string
        age  uint8
    }{
        {"user1", 10}, // 可省略元素类型。
        {"user2", 20}, // 别忘了最后一行的逗号。
    }
```

代码：

```text
package main

import (
    "fmt"
)

var arr0 [5]int = [5]int{1, 2, 3}
var arr1 = [5]int{1, 2, 3, 4, 5}
var arr2 = [...]int{1, 2, 3, 4, 5, 6}
var str = [5]string{3: "hello world", 4: "tom"}

func main() {
    a := [3]int{1, 2}           
    b := [...]int{1, 2, 3, 4}   
    c := [5]int{2: 100, 4: 200} 
    d := [...]struct {
        name string
        age  uint8
    }{
        {"user1", 10}, 
        {"user2", 20}, 
    }
    fmt.Println(arr0, arr1, arr2, str)
    fmt.Println(a, b, c, d)
}
```

输出结果:

```text
[1 2 3 0 0] [1 2 3 4 5] [1 2 3 4 5 6] [   hello world tom]
[1 2 0] [1 2 3 4] [0 0 100 0 200] [{user1 10} {user2 20}]
```

#### 多维数组 <a id="&#x591A;&#x7EF4;&#x6570;&#x7EC4;"></a>

```text
    全局
    var arr0 [5][3]int
    var arr1 [2][3]int = [...][3]int{{1, 2, 3}, {7, 8, 9}}
    局部：
    a := [2][3]int{{1, 2, 3}, {4, 5, 6}}
    b := [...][2]int{{1, 1}, {2, 2}, {3, 3}} // 第 2 纬度不能用 "..."。
```

代码：

```text
package main

import (
    "fmt"
)

var arr0 [5][3]int
var arr1 [2][3]int = [...][3]int{{1, 2, 3}, {7, 8, 9}}

func main() {
    a := [2][3]int{{1, 2, 3}, {4, 5, 6}}
    b := [...][2]int{{1, 1}, {2, 2}, {3, 3}} 
    fmt.Println(arr0, arr1)
    fmt.Println(a, b)
}
```

输出结果：

```text
    [[0 0 0] [0 0 0] [0 0 0] [0 0 0] [0 0 0]] [[1 2 3] [7 8 9]]
    [[1 2 3] [4 5 6]] [[1 1] [2 2] [3 3]]
```

值拷贝行为会造成性能问题，通常会建议使用 slice，或数组指针。

```text
package main

import (
    "fmt"
)

func test(x [2]int) {
    fmt.Printf("x: %p\n", &x)
    x[1] = 1000
}

func main() {
    a := [2]int{}
    fmt.Printf("a: %p\n", &a)

    test(a)
    fmt.Println(a)
}
```

输出结果:

```text
    a: 0xc42007c010
    x: 0xc42007c030
    [0 0]
```

内置函数 len 和 cap 都返回数组长度 \(元素数量\)。

```text
package main

func main() {
    a := [2]int{}
    println(len(a), cap(a)) 
}
```

输出结果：

```text
2 2
```

#### 多维数组遍历： <a id="&#x591A;&#x7EF4;&#x6570;&#x7EC4;&#x904D;&#x5386;&#xFF1A;"></a>

```text
package main

import (
    "fmt"
)

func main() {

    var f [2][3]int = [...][3]int{{1, 2, 3}, {7, 8, 9}}

    for k1, v1 := range f {
        for k2, v2 := range v1 {
            fmt.Printf("(%d,%d)=%d ", k1, k2, v2)
        }
        fmt.Println()
    }
}
```

输出结果：

```text
    (0,0)=1 (0,1)=2 (0,2)=3 
    (1,0)=7 (1,1)=8 (1,2)=9
```

### 1.1.2. 数组拷贝和传参 <a id="&#x6570;&#x7EC4;&#x62F7;&#x8D1D;&#x548C;&#x4F20;&#x53C2;"></a>

```text
package main

import "fmt"

func printArr(arr *[5]int) {
    arr[0] = 10
    for i, v := range arr {
        fmt.Println(i, v)
    }
}

func main() {
    var arr1 [5]int
    printArr(&arr1)
    fmt.Println(arr1)
    arr2 := [...]int{2, 4, 6, 8, 10}
    printArr(&arr2)
    fmt.Println(arr2)
}
```

### 1.1.3. 数组练习 <a id="&#x6570;&#x7EC4;&#x7EC3;&#x4E60;"></a>

#### 求数组所有元素之和 <a id="&#x6C42;&#x6570;&#x7EC4;&#x6240;&#x6709;&#x5143;&#x7D20;&#x4E4B;&#x548C;"></a>

```text
package main

import (
    "fmt"
    "math/rand"
    "time"
)


func sumArr(a [10]int) int {
    var sum int = 0
    for i := 0; i < len(a); i++ {
        sum += a[i]
    }
    return sum
}

func main() {
    
    
    
    rand.Seed(time.Now().Unix())

    var b [10]int
    for i := 0; i < len(b); i++ {
        
        b[i] = rand.Intn(1000)
    }
    sum := sumArr(b)
    fmt.Printf("sum=%d\n", sum)
}
```

#### 找出数组中和为给定值的两个元素的下标，例如数组\[1,3,5,8,7\]，找出两个元素之和等于8的下标分别是（0，4）和（1，2） <a id="&#x627E;&#x51FA;&#x6570;&#x7EC4;&#x4E2D;&#x548C;&#x4E3A;&#x7ED9;&#x5B9A;&#x503C;&#x7684;&#x4E24;&#x4E2A;&#x5143;&#x7D20;&#x7684;&#x4E0B;&#x6807;&#xFF0C;&#x4F8B;&#x5982;&#x6570;&#x7EC4;13587&#xFF0C;&#x627E;&#x51FA;&#x4E24;&#x4E2A;&#x5143;&#x7D20;&#x4E4B;&#x548C;&#x7B49;&#x4E8E;8&#x7684;&#x4E0B;&#x6807;&#x5206;&#x522B;&#x662F;&#xFF08;0&#xFF0C;4&#xFF09;&#x548C;&#xFF08;1&#xFF0C;2&#xFF09;"></a>

```text
package main

import "fmt"





func myTest(a [5]int, target int) {
    
    for i := 0; i < len(a); i++ {
        other := target - a[i]
        
        for j := i + 1; j < len(a); j++ {
            if a[j] == other {
                fmt.Printf("(%d,%d)\n", i, j)
            }
        }
    }
}

func main() {
    b := [5]int{1, 3, 5, 8, 7}
    myTest(b, 8)
}
```

