# [Middleware](http://iris-go.com/middleware/) && Party(http://iris-go.com/party/) && [subdomains](http://iris-go.com/subdomains/)

什麼是Middleware？

這是一種中介層，直接舉例比較清楚

如果我要記錄每個Request的LOG，我必須要在Route時紀錄，但其實都是做同樣的一件事情，我們就在中間記錄完，在交給Route要做的事情就好。

或者是認證，有些需要認證過才能使用的方法，如果每個方法都要一直去檢視，這樣就會非常麻煩。

主要的應用大多在認證、LOG、i18n，JWT，Recovery，secure。

或者參考[官方提供的Middleware](https://github.com/iris-contrib/middleware)

範例一開始就給我們看了幾個基本的Middleware使用方式，其實跟Handle差不多（因為他們也是要從context去取資料然後做一些共同的事情）。

在github.com\kataras\iris\http.go，type就有定義Middleware。
```go
Middleware []Handler
```

Middleware使用方式：
* Use(UseFunc)
* Handle最後的參數

與Handle一樣，兩種方式，只要能處理context的方法即可。

```go
// Then declare which middleware to use (custom or not)
iris.Use(myMiddleware{})
iris.UseFunc(func(ctx *iris.Context){})

// Now declare routes
iris.Get("/myroute", func(c *iris.Context) {
    // do stuff
})
iris.Get("/secondroute", myMiddlewareFunc, myRouteHandlerfunc)

// myMiddleware will be like that

type myMiddleware struct {
  // your 'stateless' fields here
}

func (m *myMiddleware) Serve(ctx *iris.Context){
  // ...
}
```

每個Route都可以有很多的Middleware，而且他在執行時是按照五們宣告的順序執行，同樣在github.com\kataras\iris\http.go，其實就是把middleware這個Handler的陣列取出（Next方法），然後把context傳入來處理事情。

其中，還有一個API叫做DoneFunc，這個就是所有的做完會執行的，而且我們必須要在前一個Middleware或是Handle本身去呼叫Next()，最後才會執行，目前看起來沒有那麼直覺。

**如果Middleware沒有往後跑，全部檢查有沒有Next，官網範例有些也沒有加，需要自己補。**

```go
// Next calls all the next handler from the middleware stack, it used inside a middleware
func (ctx *Context) Next() {
	//set position to the next
	ctx.pos++
	midLen := uint8(len(ctx.middleware))
	//run the next
	if ctx.pos < midLen {
		ctx.middleware[ctx.pos].Serve(ctx)
	}
}
```

在範例裡有一段註解，Next放在第一行，可以繼續執行下一個Middleware，稍後來測試一下。
```go
c.Next()// process the request first, we don't want to have delays
```



接這看看範例最後一個API *Party*。

這個API其實就是幫助我們串上層的路徑，譬如"/api/posts"、"/api/posts/:id"、等，我們當然可以每個路徑都寫一個Route，但是數量一多就會變的難以維護，如果使用這種路徑管理的方式則會好維護很多。

而且Party也可以針對性的下Middleware，譬如/Admin下全部都需要加一個登入權限，其他的則依據需求時。

Party後可以在接Party，因為他的回傳值是一個MuxAPI（處理Handle的事情，在handlers也有提過。

```go
func (api *muxAPI) Party(relativePath string, handlersFn ...HandlerFunc) MuxAPI {
```

範例產生的路徑
* GET /admin/(/admin)
* GET /admin/dashboard
* DELETE /admin/delete/:userId

```go
admin := api.Party("/admin", func(ctx *iris.Context) {
    ctx.Write("Middleware for all party's routes!")
    ctx.Next()
})

{
    // add a silly middleware
    admin.UseFunc(func(c *iris.Context) {
        //your authentication logic here...
        println("from ", c.PathString())
        authorized := true
        if authorized {
            c.Next()
        } else {
            c.Text(401, c.PathString()+" is not authorized for you")
        }

    })
    admin.Get("/", func(c *iris.Context) {
        c.Write("from /admin/ or /admin if you pathcorrection on")
    })
    admin.Get("/dashboard", func(c *iris.Context) {
        c.Write("/admin/dashboard")
    })
    admin.Delete("/delete/:userId", func(c *iris.Context) {
        c.Write("admin/delete/%s", c.Param("userId"))
    })
}
```

Party除了設定上層路徑之外，我們如果加".(dot)"，就會變成SubDomain（也算合理，畢竟Hostname也算是路徑）。

在Subdomain碰到了一些問題，就是在Listen時，好像不是每個Listen都有辦法處理，譬如ListenTLSAuto。

目前猜測有可能是ACME的問題。