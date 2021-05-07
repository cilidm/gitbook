# Cookie介绍

* [**1.** Cookie介绍](cookie-jie-shao.md#cookie介绍)
  * * [**1.1.1.** Cookie的用途](cookie-jie-shao.md#cookie的用途)

## 1. Cookie介绍 <a id="cookie&#x4ECB;&#x7ECD;"></a>

* HTTP是无状态协议，服务器不能记录浏览器的访问状态，也就是说服务器不能区分两次请求是否由同一个客户端发出
* Cookie就是解决HTTP协议无状态的方案之一，中文是小甜饼的意思
* Cookie实际上就是服务器保存在浏览器上的一段信息。浏览器有了Cookie之后，每次向服务器发送请求时都会同时将该信息发送给服务器，服务器收到请求后，就可以根据该信息处理请求
* Cookie由服务器创建，并发送给浏览器，最终由浏览器保存

### 1.1.1. Cookie的用途 <a id="cookie&#x7684;&#x7528;&#x9014;"></a>

* 测试服务端发送cookie给客户端，客户端请求时携带cookie
