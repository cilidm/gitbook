# uuid

## 1. uuid <a id="uuid"></a>

### 1.1.1. 什么是uuid? <a id="&#x4EC0;&#x4E48;&#x662F;uuid"></a>

uuid是Universally Unique Identifier的缩写，即通用唯一识别码。

uuid的目的是让分布式系统中的所有元素，都能有唯一的辨识资讯，而不需要透过中央控制端来做辨识资讯的指定。如此一来，每个人都可以建立不与其它人冲突的 uuid。

A universally unique identifier \(UUID\) is a 128-bit number used to identify information in computer systems.

例如java中生成uuid：

```text
package com.mytest;
import java.util.UUID;
public class UTest {
    public static void main(String[] args) {
        UUID uuid = UUID.randomUUID();
        System.out.println(uuid);
}}
```

c++中生成uuid：

```text
#pragma comment(lib, "rpcrt4.lib")
#include <windows.h>
#include <iostream>

using namespace std;

int main()
{
    UUID uuid;
    UuidCreate(&uuid);
    char *str;
    UuidToStringA(&uuid, (RPC_CSTR*)&str);
    cout<<str<<endl;
    RpcStringFreeA((RPC_CSTR*)&str);
    return 0;
}
```

go生成uuid：

目前，golang中的uuid还没有纳入标准库，我们使用github上的开源库：

```text
    go get -u github.com/satori/go.uuid
```

使用：

```text
package main

import (
    "fmt"

    uuid "github.com/satori/go.uuid"
)

func main() {
    
    u1, _ := uuid.NewV4()
    fmt.Printf("UUIDv4: %s\n", u1)

    
    u2, err := uuid.FromString("f5394eef-e576-4709-9e4b-a7c231bd34a4")
    if err != nil {
        fmt.Printf("Something gone wrong: %s", err)
        return
    }
    fmt.Printf("Successfully parsed: %s", u2)
}
```

uuid在websocket中使用

这里就是一个简单的使用而已，在websocket中为每一个连接的客户端分配一个uuid。

golang中可以使用github.com/gorilla/websocket为我们提供的websocket开发包。

声明一个客户端结构体：

```text
    type Client struct {
        id     string
        socket *websocket.Conn
        send   chan []byte
    }
```

使用：

```text
    client := &Client{id: uuid.NewV4().String(), socket: conn, send: make(chan []byte)}
```

