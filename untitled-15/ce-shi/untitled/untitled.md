# 性能测试-地鼠文档

## 源代码目录结构 <a id="ok0w7"></a>

我们在gotest包中创建两个文件，目录结构如下所示：

```text
[GoExpert]
|
   |
      |
      |
```

其中`benchmark.go`为源代码文件，`benchmark_test.go`为测试文件。

## 源代码文件 <a id="7tkl2w"></a>

源代码文件`benchmark.go`中包含`MakeSliceWithoutAlloc()`和`MakeSliceWithPreAlloc()`两个方法，如下所示：

```text
package gotest


func MakeSliceWithoutAlloc() []int {
    var newSlice []int

    for i := 0; i < 100000; i++ {
        newSlice = append(newSlice, i)
    }

    return newSlice
}


func MakeSliceWithPreAlloc() []int {
    var newSlice []int

    newSlice = make([]int, 0, 100000)
    for i := 0; i < 100000; i++ {
        newSlice = append(newSlice, i)
    }

    return newSlice
}
```

两个方法都会构造一个容量为100000的切片，所不同的是`MakeSliceWithPreAlloc()`会预先分配内存，而`MakeSliceWithoutAlloc()`不预先分配内存，二者理论上存在性能差异，本次就来测试一下二者的性能差异。

## 测试文件 <a id="4vjv9t"></a>

测试文件`benchmark_test.go`中包含两个测试方法，用于测试源代码中两个方法的性能，测试文件如下所示：

```text
package gotest_test

import (
    "testing"
    "gotest"
)

func BenchmarkMakeSliceWithoutAlloc(b *testing.B) {
    for i := 0; i < b.N; i++ {
        gotest.MakeSliceWithoutAlloc()
    }
}

func BenchmarkMakeSliceWithPreAlloc(b *testing.B) {
    for i := 0; i < b.N; i++ {
        gotest.MakeSliceWithPreAlloc()
    }
}
```

性能测试函数命名规则为”BenchmarkXxx”，其中”Xxx”为自定义的标识，需要以大写字母开始，通常为待测函数。

testing.B提供了一系列的用于辅助性能测试的方法或成员，比如本例中的`b.N`表示循环执行的次数，而N值不用程序员特别关心，按照官方说法，N值是动态调整的，直到可靠地算出程序执行时间后才会停止，具体执行次数会在执行结束后打印出来。

## 执行测试 <a id="a5hnow"></a>

命令行下，使用`go test -bench=.`命令即可启动性能测试，如下所示：

```text
E:\OpenSource\GitHub\RainbowMango\GoExpertProgrammingSourceCode\GoExpert\src\gotest>go test -bench=.
BenchmarkMakeSliceWithoutAlloc-4            2000           1103822 ns/op
BenchmarkMakeSliceWithPreAlloc-4            5000            328944 ns/op
PASS
ok      gotest  4.445s
```

其中`-bench`为go test的flag，该flag指示go test进行性能测试，即执行测试文件中符合”BenchmarkXxx”规则的方法。  
紧跟flag的为flag的参数，本例表示执行当前所有的性能测试。

通过输出可以直观的看出，`BenchmarkMakeSliceWithoutAlloc`执行了2000次，平均每次`1103822`纳秒，`BenchmarkMakeSliceWithPreAlloc`执行了5000次，平均每次`328944`纳秒。

从测试结果上看，虽然构造切片很快，但通过给切片预分配内存，性能还可以进一步提升，符合预期。关于原理分析，请参考Slice相关章节。

## 总结 <a id="clj0wo"></a>

从上面的例子可以看出，编写并执行性能测试是非常简单的，只需要遵循一些规则：

* 文件名必须以“\_test.go”结尾；
* 函数名必须以“BenchmarkXxx”开始；
* 使用命令“go test -bench=.”即可开始性能测试；

文档更新时间: 2020-08-08 22:00   作者：kuteng

