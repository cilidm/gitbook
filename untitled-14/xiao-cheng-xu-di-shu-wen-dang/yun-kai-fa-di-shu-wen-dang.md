# 云开发-地鼠文档

Tencent Cloud Base [文档](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/reference-http-api/)

## 使用说明 <a id="d8ctv"></a>

**初始化配置**

```text
wc := wechat.NewWechat()
//使用memcache保存access_token，也可选择redis或自定义cache
memCache:=cache.NewMemcache("127.0.0.1:11211")

//配置小程序参数
config := &wechat.Config{
    AppID:     "your app id",
    AppSecret: "your app secret",
    Cache:     memCache,
}
miniprogram := wc.GetMiniProgram(cfg)
wcTcb := miniprogram.GetTcb()
```

## 举例 <a id="6asoq2"></a>

### 触发云函数 <a id="bfn5ly"></a>

```text
res, err := wcTcb.InvokeCloudFunction("test-xxxx", "add", `{"a":1,"b":2}`)
if err != nil {
    panic(err)
}
```

更多使用方法参考[GODOC](https://godoc.org/github.com/silenceper/wechat/v2/miniprogram/tcb)

文档更新时间: 2020-08-19 10:32   作者：kuteng

