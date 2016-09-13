# 小章節的都放在這邊

## [Gzip](http://iris-go.com/gzip/)

Gzip就是壓縮我們的文件，可以幫使用者節省流量

但另一個面向就是我們自己的伺服器會消耗CPU做壓縮這件事情

所以有時候需要觀察一下，看看是不是一定要做這件事情

在iris加上gzip其實也很簡單

再一開始可以設定

```go
iris.Config.Gzip = true
```
或者在每個方法自行處理
```go
//1.在Render時
iris.RenderOptions{"gzip": false}

//2.使用內建方法
ctx.Response.WriteGzip(...)
```

第二個方法還有第二個參數版本的**WriteGzipLevel**，第二個參數為壓縮等級，共有下列幾種

```go
// Level is the desired compression level:
//
//     * CompressNoCompression
//     * CompressBestSpeed
//     * CompressBestCompression
//     * CompressDefaultCompression
```


