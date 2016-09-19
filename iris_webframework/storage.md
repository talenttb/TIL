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

