# Cookies & Sessions

## Cookies

Cookie是存在瀏覽器端，通常會隨著用戶端的請求一起帶過來的資料

iris提供的的操作就是如下

1. SetCookie(cookie *fasthttp.Cookie)
1. SetCookieKV(key, value string)
1. GetCookie(name string) string
1. RemoveCookie(name string)
1. VisitAllCookies(visitor func(key string, value string))

2,3基本上應該沒啥問題，2的時效為兩小時且只能用http

5作者有說內建的fasthttp有bug，所以才先寫這一個

主要來談談1,4

首先是刪除，行為如下
```go
// 1.將Response的刪除（使用的是array複製並扣除要移除的）
ctx.Response.Header.DelCookie(name)
// 2.將fasthttp的cookie拿出來並將指定的設為空值且過期
// 並且更新fasthttp裡pool的資料
cookie := fasthttp.AcquireCookie()
ctx.SetCookie(cookie)
fasthttp.ReleaseCookie(cookie)
// 3.保險起見也將Request的刪除
ctx.Request.Header.DelCookie(name)
```

一個刪除也做了很多事情

再來談談SetCookie，這是可自訂最多東西的，使用fasthttp.Cookie

```go
type Cookie struct {
	noCopy noCopy

	key    []byte
	value  []byte
	expire time.Time
	domain []byte
	path   []byte

	httpOnly bool
	secure   bool

	bufKV argsKV
	buf   []byte
}
```

如果我們需要設定subDomain或是長時效等等額外設定，就必須使用這個物件了


## Sessions

記得下載
```go
go get -u github.com/kataras/go-sessions
```

在iris裡，Session是被包成一個物件，所以我們再使用時，需要從context取得session然後在使用以下方法操作，實做在 */sessions.go* 這個檔案裡。
```go
Session interface {
	ID() string
	Get(string) interface{}
	GetString(key string) string
	GetInt(key string) int
	GetAll() map[string]interface{}
	VisitAll(cb func(k string, v interface{}))
	Set(string, interface{})
	Delete(string)
	Clear()
}
```

其實再下一層是**sessionsManager**，這裡面有一個provider，所以我們可以使用本機或是iris提供的redis來存放sessions。

##### *Redis的設定參考範例使用UseSessionDB即可

我們在*ctx.Session().Set()* 後，前端的Cookie Key 是**irissessionid**

更改方式一樣從最外層iris.config下設定sessions相關屬性

```go
const (
	// DefaultCookieName the secret cookie's name for sessions
	DefaultCookieName = "irissessionid"
	// DefaultSessionGcDuration  is the default Session Manager's GCDuration , which is 2 hours
	DefaultSessionGcDuration = time.Duration(2) * time.Hour
	// DefaultCookieLength is the default Session Manager's CookieLength, which is 32
	DefaultCookieLength = 32
)

// DefaultSessionsConfiguration the default configs for Sessions
func DefaultSessionsConfiguration() SessionsConfiguration {
	return SessionsConfiguration{
		Cookie:                      DefaultCookieName,
		CookieLength:                DefaultCookieLength,
		DecodeCookie:                false,
		Expires:                     0,
		GcDuration:                  DefaultSessionGcDuration,
		DisableSubdomainPersistence: false,
		DisableAutoGC:               true,
	}
}
```

另外一樣搭配*iris.Party*就可以設定subdomain。