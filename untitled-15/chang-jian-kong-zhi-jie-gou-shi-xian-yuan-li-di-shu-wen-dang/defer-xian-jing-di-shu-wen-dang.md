# defer 陷阱-地鼠文档

## 前言 <a id="64yz0e"></a>

项目中，有时为了让程序更健壮，也即不`panic`，我们或许会使用`recover()`来接收异常并处理。

比如以下代码：

```text
func NoPanic() {
    if err := recover(); err != nil {
        fmt.Println("Recover success...")
    }
}

func Dived(n int) {
    defer NoPanic()

    fmt.Println(1/n)
}
```

`func NoPanic()` 会自动接收异常，并打印相关日志，算是一个通用的异常处理函数。

业务处理函数中只要使用了`defer NoPanic()`，那么就不会再有`panic`发生。

关于是否应该使用recover接收异常，以及什么场景下使用等问题不在本节讨论范围内。  
本节关注的是这种用法的一个变体，曾经出现在笔者经历的一个真实项目，在该变体下，recover再也无法接收异常。

## recover使用误区 <a id="837k60"></a>

在项目中，有众多的数据库更新操作，正常的更新操作需要提交，而失败的就需要回滚，如果异常分支比较多，  
就会有很多重复的回滚代码，所以有人尝试了一个做法：即在defer中判断是否出现异常，有异常则回滚，否则提交。

简化代码如下所示：

```text
func IsPanic() bool {
    if err := recover(); err != nil {
        fmt.Println("Recover success...")
        return true
    }

    return false
}

func UpdateTable() {
    
    defer func() {
        if IsPanic() {
            
        } else {
            
        }
    }()

    
}
```

`func IsPanic() bool` 用来接收异常，返回值用来说明是否发生了异常。`func UpdateTable()`函数中，使用defer来判断最终应该提交还是回滚。

上面代码初步看起来还算合理，但是此处的`IsPanic()`再也不会返回`true`，不是`IsPanic()`函数的问题，而是其调用的位置不对。

## recover 失效的条件 <a id="46wqvo"></a>

上面代码`IsPanic()`失效了，其原因是违反了recover的一个限制，导致recover\(\)失效（永远返回`nil`）。

以下三个条件会让recover\(\)返回`nil`:

1. panic时指定的参数为`nil`；（一般panic语句如`panic("xxx failed...")`）
2. 当前协程没有发生panic；
3. recover没有被defer方法直接调用；

前两条都比较容易理解，上述例子正是匹配第3个条件。

本例中，recover\(\) 调用栈为“defer （匿名）函数” –&gt; IsPanic\(\) –&gt; recover\(\)。也就是说，recover并没有被defer方法直接调用。符合第3个条件，所以recover\(\) 永远返回nil。

> 赠人玫瑰手留余香，如果觉得不错请给个赞~
>
> 本篇文章已归档到GitHub项目，求星~ [点我即达](https://github.com/RainbowMango/GoExpertProgramming)

文档更新时间: 2020-08-08 17:58   作者：kuteng

