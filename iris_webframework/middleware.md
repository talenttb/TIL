# [Middleware](http://iris-go.com/middleware/)

什麼是Middleware？

這是一種中介層，直接舉例比較清楚

如果我要記錄每個Request的LOG，我必須要在Route時紀錄，但其實都是做同樣的一件事情，我們就在中間記錄完，在交給Route要做的事情就好。

或者是認證，有些需要認證過才能使用的方法，如果每個方法都要一直去檢視，這樣就會非常麻煩。

主要的應用大多在認證、LOG、i18n，JWT，Recovery，secure。

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

每個Route都可以有很多的Middleware，而且他在執行時是按照五們宣告的順序執行。

其中，還有一個API叫做DoneFunc，這個就是所有的做完會執行的，而且我們必須要在前一個Middleware或是Handle本身去呼叫Next()，最後才會執行，目前看起來沒有那麼直覺。

同樣在github.com\kataras\iris\http.go，其實就是把middleware這個Handler的陣列取出，然後把context傳入來處理事情。

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


**明天來看Party(http://iris-go.com/party/)**