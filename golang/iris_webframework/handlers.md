# [Handlers](http://iris-go.com/handlers/) && [Name parameter](http://iris-go.com/named_parameters/)

這邊看到Handler可以使用很多方式來做

來分析一下這些方式到底有什麼不一樣

一開始看到的
```go
api := iris.New()
api.Get("/hi", hi)

func hi(ctx *iris.Context){
   ctx.Write("Hi %s", "iris")
}

//or

iris.Get("/hi", func(ctx *iris.Context) {
    ctx.Write("Hi %s", "iris")
})
```

上面的api或iris，其實都是*Framework struct*，而裡面有個屬性是[**muxAPI*](https://godoc.org/github.com/kataras/iris#MuxAPI)，這個東西就是建置我們網站或API的重要方法。

接著要新增Handlers有幾個方法
* MuxAPI.Handle(method string, registedPath string, handlers ...Handler) RouteNameFunc
* MuxAPI.HandleFunc(method string, registedPath string, handlersFn ...HandlerFunc) RouteNameFunc
* MuxAPI.API(path string, api HandlerAPI, middleware ...HandlerFunc) error
* 使用Handle但是是操作原生的Request/Response，官方不推薦，所以就暫不說明。

##### 其中Method如果是空的，他會自己幫你串所有的HTTP METHOD("GET", "POST", "PUT", "DELETE", "CONNECT", "HEAD", "PATCH", "OPTIONS", "TRACE")

再來是Handlers裡的說明

Handlers必須實做Handler interface，然後會回傳 RouteNameFunc。
```go
type Handler interface {
    Serve(*Context)
}
```
##### 會回傳 RouteNameFunc 的包含[Handle,HandleFunc,Get,Post,...,Static,StaticWeb,...]，這個可以設定路徑的名稱，稍後會提到。


```go
api.Handle("GET", "/hi", &MyHandler{})

type MyHandler struct {
}

func (m *MyHandler) Serve(ctx *iris.Context) {
	ctx.Write("Hi %s", "iris")
}
```

如果沒有要在struct放入什麼資料的話，其實使用HandlerFuncs是比較方便的。

接著是使用HandlerFunc

```go
func handlerFunc(c *iris.Context)  {
    c.Write("Hello")
}

iris.HandleFunc("GET","/hi",handlerFunc)
```

就是使用一個func，然後要接收Context，這個感覺跟最一開始提到的 *iris.Get({path},{func})* 很像，其實他們就是一樣的。

在github.com\kataras\iris\iris.go，1429行或是搜尋[func (api *muxAPI) Get]，即可找到。

```go
// Get registers a route for the Get http method
func (api *muxAPI) Get(path string, handlersFn ...HandlerFunc) RouteNameFunc {
	return api.HandleFunc(MethodGet, path, handlersFn...)
}
```

以上兩個方法理論上直接使用 **Handle** 這個API會比較快，因為HandleFunc會再包一層把Function轉成Handle。

再來是使用 **API** 這個方法。

這個他會幫你一次產出HTTP的所有方法，並使用Restful的方式，讓你可以快速的做出對應的路徑。

但其實這個方法官方不是很建議，除非你不在意route的效能，或者使用ROR很習慣的人。

```go
type UserAPI struct {
	*iris.Context
}

// GET /users
func (u UserAPI) Get() {
}

// GET /users/:param1 which its value passed to the id argument
func (u UserAPI) GetBy(id string) { // id equals to u.Param("param1")
}

// POST /users
func (u UserAPI) Post() {
	name := u.FormValue("name")
}

// PUT /users/:param1
func (u UserAPI) PutBy(id string) {
	name := u.FormValue("name")
	println(string(name))
	println("Put from /users/" + id)
}

// DELETE /users/:param1
func (u UserAPI) DeleteBy(id string) {
}

iris.API("/users", UserAPI{})
```

來看看這個API是怎麼自動做出來的

在github.com\kataras\iris\iris.go
```go

func (api *muxAPI) API(path string, restAPI HandlerAPI, middleware ...HandlerFunc) {

...

// GET, DELETE -> with url named parameters (/users/:id/:secondArgumentIfExists)
// POST, PUT -> with post values (form)

methodWithBy := strings.Title(strings.ToLower(methodName)) + "By"
if method, found := typ.MethodByName(methodWithBy); found {
```

他對所有的methodName(GET, POST, ...)都加上By，然後從我們傳進去的物件來找有沒有MethodName+"By"的名稱，然後再使用HandleFunc註冊進去，就有了自動化產生的效果了。

但這種方法就如同作者所述，很慢，因為要處理的事情很多。


接下來介紹一下要怎麼找所有的Routes path

1. 查詢所有的路徑
    ```go
    for _, v := range iris.Lookups() {
        fmt.Print(v.Name() + " - " + v.Method() + "\n")
    }
    ```

1. 查詢個別的路徑
    ```go
    iris.Lookup(routeName string) Route
    ```
##### [Route type](https://godoc.org/github.com/kataras/iris#Route)

另外還有結合Template的部分，就於之後再提（或補）

對應完路徑的處理事件後，接著我們要在Path裡設定變數，譬如剛剛API所使用的 *:id*

在Handle Path裡面，我們可以直接使用:id以及*action來放在路徑裡（不限個數），然後在Func裡面在做Param取值的動作。

##### Param預設空字串，ParamInt(ParamInt64)回傳Int, error

```go
// :為1對1
iris.Get("/profile/:fullname/friends/:friendID", func(c *iris.Context) {
    // Retrieve the parameters fullname and friendID
    fullname := c.Param("fullname")
    friendID, err := c.ParamInt("friendID")
})

//or
// *為萬用字元
// Path : /anything/new，action = /new
// Not Match: /anything , /anything/ , /something
iris.Get("/anything/*action", func(c *iris.Context) {
})
```



iris 這個專案的Route也是使用[fasthttp](https://github.com/valyala/fasthttp)