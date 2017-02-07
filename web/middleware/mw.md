# Middleware

Web的middleware就是當Request進來後，我們在執行他應該做的事情「前」及「後」插入我們需要的動作，所以叫Middleware。

常用的Middleware：
* Logger
* Auth
* Execusive time
* CORS

接下來就直接使用程式碼說明

```go
//使用mux的原因是讓各個路徑可以各自加所需的mw
mux := http.NewServeMux()
//將原本的handler加上處理，就是前面所說的middleware
commonMux := Time(mux)

func Time(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		next.ServeHTTP(w, r)
		elapsed := time.Since(start)
		log.Printf("%s took %s", r.URL, elapsed)
	})
}
//最後的執行
http.ListenAndServe(config.Port, commonMux)
```

以上的程式碼就會將每個method的執行時間記錄下來

下面這兩個連結是別人製作的middleware的框架

https://github.com/urfave/negroni

https://github.com/justinas/alice