# [Static Files](http://iris-go.com/static_files/)

這邊提供了幾個主要靜態的API方法，來一一介紹
1. **Static**(relative string, systemPath string, stripSlashes int) RouteNameFunc
1. **StaticContent**(reqPath string, cType string, content []byte) RouteNameFunc
1. **StaticFS**(reqPath string, systemPath string, stripSlashes int) RouteNameFunc
1. **StaticHandler**(systemPath string, stripSlashes int, compress bool, generateIndexPages bool, indexNames []string) HandlerFunc
1. **StaticServe**(systemPath string, requestPath ...string) RouteNameFunc
1. **StaticWeb**(reqPath string, systemPath string, stripSlashes int) RouteNameFunc
1. **ServeFile**(filename string, gzipCompression bool) error

先從常用的**StaticWeb**以及參數裡有*stripSlashes*的開始吧

stripSlashes，就是把Slash切掉然後取第n個來當我們的url root path，以下就是兩個相同的例子。

```go
// file tree:
// static -|
//         | index.html
// main.go
// main.exe

// url:/index.html
iris.StaticWeb("/static", "./static", 1)
// or
iris.StaticWeb("/", "./static", 0)
```
然後第二個參數是相對於exe執行檔的目錄。

也是一樣的，差在Static不會自己找index page。

**Static**與**StaticFS**使用方式都與**StaticWeb**相同，差異會於下圖說明。

**StaticServe**第一個參數是直接對應系統路徑，第二個參數是URL的路徑（如果沒有，路徑跟前面一樣），其實就跟StaticWeb差不多，寫法不相同，差異如下圖。

```go
//兩者相同
iris.StaticServe("./static/mywebpage","/webpage")
iris.StaticWeb("/webpage","./static/mywebpage", 1)
```

|method   |index   |cache(10s)   |zip   |
|:-:|:-:|:-:|:-:|
|Static   |N   |N   |N|
|StaticWeb   |Y   |N   |N|
|StaticServe   |Y   |N   |Y   |
|StaticFS   |Y   |Y   |Y|

**ServeFile**這個是指定單獨的檔案來做輸出，如果配合Route的全符合路徑，然後手動檢查路徑是否存在，配合此方法就是全動態的（StaticServe就是多改一個路徑，然後呼叫此方法）。

最後要來看最關鍵的方法**StaticHandler**，為什麼說最關鍵呢，因為上述的方法最後都是呼叫此方法

再看一次此方法的參數，然後配合上表即可明白。
```go
StaticHandler(systemPath string, stripSlashes int, compress bool, generateIndexPages bool, indexNames []string)
```

**StaticContent**配合**websocket**時一起介紹。

補充：

iris提供了一個叫**Favicon**的API，其用法與StaticServe相同，如果第二個參數沒寫，則是直接對應到/favicon.ico。