# Http

## 1. Http <a id="http"></a>

Go语言内置的net/http包十分的优秀，提供了HTTP客户端和服务端的实现。

### 1.1.1. net/http介绍 <a id="nethttp&#x4ECB;&#x7ECD;"></a>

Go语言内置的net/http包提供了HTTP客户端和服务端的实现。

### 1.1.2. HTTP协议 <a id="http&#x534F;&#x8BAE;"></a>

超文本传输协议（HTTP，HyperText Transfer Protocol\)是互联网上应用最为广泛的一种网络传输协议，所有的WWW文件都必须遵守这个标准。设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法。

### 1.1.3. HTTP客户端 <a id="http&#x5BA2;&#x6237;&#x7AEF;"></a>

基本的HTTP/HTTPS请求 Get、Head、Post和PostForm函数发出HTTP/HTTPS请求。

```text
resp, err := http.Get("http://5lmh.com/")
...
resp, err := http.Post("http://5lmh.com/upload", "image/jpeg", &buf)
...
resp, err := http.PostForm("http://5lmh.com/form",
    url.Values{"key": {"Value"}, "id": {"123"}})
```

程序在使用完response后必须关闭回复的主体。

```text
resp, err := http.Get("http://5lmh.com/")
if err != nil {
    
}
defer resp.Body.Close()
body, err := ioutil.ReadAll(resp.Body)

```

### 1.1.4. GET请求示例 <a id="get&#x8BF7;&#x6C42;&#x793A;&#x4F8B;"></a>

使用net/http包编写一个简单的发送HTTP请求的Client端，代码如下：

```text
package main

import (
    "fmt"
    "io/ioutil"
    "net/http"
)

func main() {
    resp, err := http.Get("https://www.5lmh.com/")
    if err != nil {
        fmt.Println("get failed, err:", err)
        return
    }
    defer resp.Body.Close()
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println("read from resp.Body failed,err:", err)
        return
    }
    fmt.Print(string(body))
}
```

将上面的代码保存之后编译成可执行文件，执行之后就能在终端打印liwenzhou.com网站首页的内容了，我们的浏览器其实就是一个发送和接收HTTP协议数据的客户端，我们平时通过浏览器访问网页其实就是从网站的服务器接收HTTP数据，然后浏览器会按照HTML、CSS等规则将网页渲染展示出来。

### 1.1.5. 带参数的GET请求示例 <a id="&#x5E26;&#x53C2;&#x6570;&#x7684;get&#x8BF7;&#x6C42;&#x793A;&#x4F8B;"></a>

关于GET请求的参数需要使用Go语言内置的net/url这个标准库来处理。

```text
func main() {
    apiUrl := "http://127.0.0.1:9090/get"
    
    data := url.Values{}
    data.Set("name", "枯藤")
    data.Set("age", "18")
    u, err := url.ParseRequestURI(apiUrl)
    if err != nil {
        fmt.Printf("parse url requestUrl failed,err:%v\n", err)
    }
    u.RawQuery = data.Encode() 
    fmt.Println(u.String())
    resp, err := http.Get(u.String())
    if err != nil {
        fmt.Println("post failed, err:%v\n", err)
        return
    }
    defer resp.Body.Close()
    b, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println("get resp failed,err:%v\n", err)
        return
    }
    fmt.Println(string(b))
}
```

对应的Server端HandlerFunc如下：

```text
func getHandler(w http.ResponseWriter, r *http.Request) {
    defer r.Body.Close()
    data := r.URL.Query()
    fmt.Println(data.Get("name"))
    fmt.Println(data.Get("age"))
    answer := `{"status": "ok"}`
    w.Write([]byte(answer))
}
```

### 1.1.6. Post请求示例 <a id="post&#x8BF7;&#x6C42;&#x793A;&#x4F8B;"></a>

上面演示了使用net/http包发送GET请求的示例，发送POST请求的示例代码如下：

```text
package main

import (
    "fmt"
    "io/ioutil"
    "net/http"
    "strings"
)



func main() {
    url := "http://127.0.0.1:9090/post"
    
    
    
    
    contentType := "application/json"
    data := `{"name":"枯藤","age":18}`
    resp, err := http.Post(url, contentType, strings.NewReader(data))
    if err != nil {
        fmt.Println("post failed, err:%v\n", err)
        return
    }
    defer resp.Body.Close()
    b, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println("get resp failed,err:%v\n", err)
        return
    }
    fmt.Println(string(b))
}
```

对应的Server端HandlerFunc如下：

```text
func postHandler(w http.ResponseWriter, r *http.Request) {
    defer r.Body.Close()
    
    r.ParseForm()
    fmt.Println(r.PostForm) 
    fmt.Println(r.PostForm.Get("name"), r.PostForm.Get("age"))
    
    b, err := ioutil.ReadAll(r.Body)
    if err != nil {
        fmt.Println("read request.Body failed, err:%v\n", err)
        return
    }
    fmt.Println(string(b))
    answer := `{"status": "ok"}`
    w.Write([]byte(answer))
}
```

### 1.1.7. 自定义Client <a id="&#x81EA;&#x5B9A;&#x4E49;client"></a>

要管理HTTP客户端的头域、重定向策略和其他设置，创建一个Client：

```text
client := &http.Client{
    CheckRedirect: redirectPolicyFunc,
}
resp, err := client.Get("http://5lmh.com")

req, err := http.NewRequest("GET", "http://5lmh.com", nil)

req.Header.Add("If-None-Match", `W/"wyzzy"`)
resp, err := client.Do(req)

```

### 1.1.8. 自定义Transport <a id="&#x81EA;&#x5B9A;&#x4E49;transport"></a>

要管理代理、TLS配置、keep-alive、压缩和其他设置，创建一个Transport：

```text
tr := &http.Transport{
    TLSClientConfig:    &tls.Config{RootCAs: pool},
    DisableCompression: true,
}
client := &http.Client{Transport: tr}
resp, err := client.Get("https://5lmh.com")
```

Client和Transport类型都可以安全的被多个go程同时使用。出于效率考虑，应该一次建立、尽量重用。

### 1.1.9. 服务端 <a id="&#x670D;&#x52A1;&#x7AEF;"></a>

#### 默认的Server <a id="&#x9ED8;&#x8BA4;&#x7684;server"></a>

ListenAndServe使用指定的监听地址和处理器启动一个HTTP服务端。处理器参数通常是nil，这表示采用包变量DefaultServeMux作为处理器。

Handle和HandleFunc函数可以向DefaultServeMux添加处理器。

```text
http.Handle("/foo", fooHandler)
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})
log.Fatal(http.ListenAndServe(":8080", nil))
```

#### 默认的Server示例 <a id="&#x9ED8;&#x8BA4;&#x7684;server&#x793A;&#x4F8B;"></a>

使用Go语言中的net/http包来编写一个简单的接收HTTP请求的Server端示例，net/http包是对net包的进一步封装，专门用来处理HTTP协议的数据。具体的代码如下：

```text


func sayHello(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Hello 枯藤！")
}

func main() {
    http.HandleFunc("/", sayHello)
    err := http.ListenAndServe(":9090", nil)
    if err != nil {
        fmt.Printf("http server failed, err:%v\n", err)
        return
    }
}
```

将上面的代码编译之后执行，打开你电脑上的浏览器在地址栏输入127.0.0.1:9090回车，此时就能够看到 `Hello 枯藤！`

#### 自定义Server <a id="&#x81EA;&#x5B9A;&#x4E49;server"></a>

要管理服务端的行为，可以创建一个自定义的Server：

```text
s := &http.Server{
    Addr:           ":8080",
    Handler:        myHandler,
    ReadTimeout:    10 * time.Second,
    WriteTimeout:   10 * time.Second,
    MaxHeaderBytes: 1 << 20,
}
log.Fatal(s.ListenAndServe())
```

