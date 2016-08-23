# Main : [Hi](http://iris-go.com/hi/)

## 第一個範例：

```go
package main

import "github.com/kataras/iris"

func main() {
    iris.Get("/hi", func(ctx *iris.Context) {
        ctx.Write("Hi %s", "iris")
    })
    iris.Listen(":8080")
}

//The same

package main

import "github.com/kataras/iris"

func main() {
    api := iris.New()
    api.Get("/hi", hi)
    api.Listen(":8080")
}

func hi(ctx *iris.Context){
   ctx.Write("Hi %s", "iris")
}
```

# **說明**

這邊兩者使用的Listen在背後做的事情都是一樣的，因為預設會建立一個Default Framework，使用New 也是新建一個Framework蓋掉替換掉而已，直接看Code。

```go
func Listen(addr string) {
    Default.Listen(addr)
}

func (s *Framework) Listen(addr string) {
    //ListenTo後面會說明
    s.Must(s.ListenTo(config.Server{ListeningAddr: addr}))
}
```

那這邊要設定的內容有兩個，一個是iris本身的設定，另一個則是使用的網路設定。

### 首先是iris本身
```go
iris.Config().DisableBanner = true
//or
api := iris.New(config.Iris{DisableBanner: true})
```

其他可設定模組請參考 github.com\kataras\iris\config\iris.go

```go
Iris struct {
    // 判斷URL結尾
    // for example: /home/ and /home
    // Default is false
    DisablePathCorrection bool

    // URL Escape
    // routes:"/posts/:id"
    // for example: /posts/123 and /posts%2f123
    // https://github.com/kataras/iris/issues/135
    // Default is false
    DisablePathEscape bool

    // 開啟專案時會看到的Banner
    // Default is false
    DisableBanner bool

    // 效能測試的位置
    // for example if '/debug/pprof'
    // http://yourdomain:PORT/debug/pprof/
    // http://yourdomain:PORT/debug/pprof/cmdline
    // 也可使用子網域 if 'debug.'
    // http://debug.yourdomain:PORT/
    // http://debug.yourdomain:PORT/cmdline
    // Default is empty
    ProfilePath string

    // 關閉HTML的樣版引擎
    // default is false
    DisableTemplateEngines bool

    // 每個request重建一次樣版
    // default is false
    IsDevelopment bool

    //編碼
    // defaults to "UTF-8"
    Charset string

    // 壓縮
    // defaults to false
    Gzip bool

    // Sessions contains the configs for sessions
    Sessions Sessions

    // Websocket contains the configs for Websocket's server integration
    Websocket *Websocket

    // Tester contains the configs for the test framework, so far we have only one because all test framework's configs are setted by the iris itself
    // You can find example on the https://github.com/kataras/iris/glob/master/context_test.go
    Tester Tester
}
```

### 網路設定

Listen有以下方法
```go
//只有使用PORT，其他預設
func Listen(addr string)

//設定PORT以及安全性相關的檔案（https）
func ListenTLS(addr string, certFile string, keyFile string)

//1.開啟兩個web server，且自動將80 PORT轉addr port
//2.預設使用letsencrypt.org
func ListenTLSAuto(addr string)

//自行定義網路設定，並且會有回傳錯誤
func ListenTo(cfg config.Server) error

//使用socket file
func ListenUNIX(addr string, mode os.FileMode)
```

再來看看設定有哪些，設定檔在 github.com\kataras\iris\config\server.go

```go
type Server struct {
	// ListenningAddr the addr that server listens to
	ListeningAddr string
	CertFile      string
	KeyFile       string

    // 使用ListenTo手動設定
    // 或可以使用新的ListenTLSAuto
	AutoTLS bool
	// Mode this is for unix only
	Mode os.FileMode

	// By default request body size is 8MB.
	MaxRequestBodySize int

	// Default buffer size is used if not set.
	ReadBufferSize int

	// Default buffer size is used if not set.
	WriteBufferSize int

	// By default request read timeout is unlimited.
	ReadTimeout time.Duration

	// By default response write timeout is unlimited.
	WriteTimeout time.Duration

	// RedirectTo, defaults to empty
	RedirectTo string

	// Virtual If this server is not really listens to a real host, it mostly used in order to achieve testing without system modifications
	Virtual bool
	// VListeningAddr, can be used for both virtual = true or false,
	// if it's setted to not empty, then the server's Host() will return this addr instead of the ListeningAddr.
	// server's Host() is used inside global template helper funcs
	// set it when you are sure you know what it does.
	//
	// Default is empty ""
	VListeningAddr string
	// VScheme if setted to not empty value then all template's helper funcs prepends that as the url scheme instead of the real scheme
	// server's .Scheme returns VScheme if  not empty && differs from real scheme
	//
	// Default is empty ""
	VScheme string
	// Name the server's name, defaults to "iris".
	// You're free to change it, but I will trust you to don't, this is the only setting whose somebody, like me, can see if iris web framework is used
	Name string
}
```


### 補充

iris可以做到，但是少用的功能

ListenTLSAuto剛剛有說，他是一個開兩個並且導向的新方法，他可以使用現有的方法來拆解

內建有一個AddServer，所以我們可以多個port的間聽

```go
iris.AddServer(config.Server{ListeningAddr: ":80", RedirectTo: "https://" + host})

iris.ListenTLS("127.0.0.1:443", "mycert.cert", "mykey.key")
```

另外也可以在一個專案裡使用多個Framework間聽不同的PORT
```go
server1 := iris.New()
server2 := iris.New()
go server1.Listen(":8080")
server2.Listen(":1993")
```