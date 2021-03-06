# 单元测试-地鼠文档

## 源代码目录结构 <a id="g89usa"></a>

我们在gotest包中创建两个文件，目录结构如下所示：

```text
[GoExpert]
|
   |
      |
      |
```

其中`unit.go`为源代码文件，`unit_test.go`为测试文件。要保证测试文件以“\_test.go”结尾。

## 源代码文件 <a id="dgsxgu"></a>

源代码文件`unit.go`中包含一个`Add()`方法，如下所示：

```text
package gotest


func Add(a int, b int) int {
    return a + b
}
```

`Add()`方法仅提供两数加法，实际项目中不可能出现类似的方法，此处仅供单元测试示例。

## 测试文件 <a id="8ergxk"></a>

测试文件`unit_test.go`中包含一个测试方法`TestAdd()`，如下所示：

```text
package gotest_test

import (
    "testing"
    "gotest"
)

func TestAdd(t *testing.T) {
    var a = 1
    var b = 2
    var expected = 3

    actual := gotest.Add(a, b)
    if actual != expected {
        t.Errorf("Add(%d, %d) = %d; expected: %d", a, b, actual, expected)
    }
}
```

通过package语句可以看到，测试文件属于“gotest\_test”包，测试文件也可以跟源文件在同一个包，但常见的做法是创建一个包专用于测试，这样可以使测试文件和源文件隔离。GO源代码以及其他知名的开源框架通常会创建测试包，而且规则是在原包名上加上”\_test”。

测试函数命名规则为”TestXxx”，其中“Test”为单元测试的固定开头，go test只会执行以此为开头的方法。紧跟“Test”是以首字母大写的单词，用于识别待测试函数。

测试函数参数并不是必须要使用的，但”testing.T”提供了丰富的方法帮助控制测试流程。

t.Errorf\(\)用于标记测试失败，标记失败还有几个方法，在介绍testing.T结构时再详细介绍。

## 执行测试 <a id="804s65"></a>

命令行下，使用`go test`命令即可启动单元测试，如下所示：

```text
E:\OpenSource\GitHub\RainbowMango\GoExpertProgrammingSourceCode\GoExpert\src\gotest>go test
PASS
ok      gotest  0.378s

E:\OpenSource\GitHub\RainbowMango\GoExpertProgrammingSourceCode\GoExpert\src\gotest>
```

通过打印可知，测试通过，花费时间为0.378s。

## 总结 <a id="e35755"></a>

从上面可以看出，编写一个单元测试并执行是非常方便的，只需要遵循一定的规则：

* 测试文件名必须以”\_test.go”结尾；
* 测试函数名必须以“TestXxx”开始；
* 命令行下使用”go test”即可启动测试；

文档更新时间: 2020-08-08 22:00   作者：kuteng

