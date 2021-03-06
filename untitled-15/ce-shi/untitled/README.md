# 快速开始-地鼠文档

使用Go自身的测试系统使用起来非常简单，只需要添加很少的代码就可以快速开始测试。

目前Go测试系统支持单元测试、性能测试和示例测试。

## 单元测试 <a id="r6tn0"></a>

单元测试是指对软件中的最小可测试单元进行检查和验证，比如对一个函数的测试。

## 性能测试 <a id="89jm7a"></a>

性能测试，也称基准测试，可以测试一段程序的性能，可以得到时间消耗、内存使用情况的报告。

## 示例测试 <a id="7rtz6e"></a>

示例测试，广泛应用于Go源码和各种开源框架中，用于展示某个包或某个方法的用法。

比如，Go标准库中，mail包展示如何从一个字符串解析出邮件列表的用法，非常直观易懂。

源码位于`src/net/mail/example_test.go`中：

```text
func ExampleParseAddressList() {
    const list = "Alice , Bob , Eve "
    emails, err := mail.ParseAddressList(list)
    if err != nil {
        log.Fatal(err)
    }

    for _, v := range emails {
        fmt.Println(v.Name, v.Address)
    }

    
    
    
    
}
```

本节，我们通过简单的例子，快速体验一下如何使用Go的测试系统进行测试。

文档更新时间: 2020-08-08 22:00   作者：kuteng

