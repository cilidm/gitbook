# 第九十六天-地鼠文档

## 1.下面的代码输出什么？ <a id="5r7ep4"></a>

```text
type Point struct{ x, y int }

func main() {
    s := []Point{
        {1, 2},
        {3, 4},
    }
    for _, p := range s {
        p.x, p.y = p.y, p.x
    }
    fmt.Println(s)
}
```

参考答案及解析：输出 \[{1 2} {3 4}\]。知识点：for range 循环。range 循环的时候，获取到的元素值是副本，就比如这里的 p。修复代码示例：

```text
type Point struct{ x, y int }

func main() {
    s := []*Point{
        &Point{1, 2},
        &Point{3, 4},
    }
    for _, p := range s {
        p.x, p.y = p.y, p.x
    }
    fmt.Println(*s[0])
    fmt.Println(*s[1])
}
```

## 2.下面的代码有什么隐患？ <a id="g3tqvu"></a>

```text
func get() []byte {
    raw := make([]byte, 10000)
    fmt.Println(len(raw), cap(raw), &raw[0])
    return raw[:3]
}

func main() {
    data := get()
    fmt.Println(len(data), cap(data), &data[0])
}
```

参考答案及解析：get\(\) 函数返回的切片与原切片公用底层数组，如果在调用函数里面（这里是 main\(\) 函数）修改返回的切片，将会影响到原切片。为了避免掉入陷阱，可以如下修改：

```text
func get() []byte {
    raw := make([]byte, 10000)
    fmt.Println(len(raw), cap(raw), &raw[0])
    res := make([]byte, 3)
    copy(res, raw[:3])
    return res
}

func main() {
    data := get()
    fmt.Println(len(data), cap(data), &data[0])
}
```

文档更新时间: 2021-04-20 19:07   作者：kuteng

