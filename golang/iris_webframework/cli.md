# CLI

我們在下載專案時，iris會帶一個cli helper來協助我們

> $ go get -u github.com/kataras/iris/iris

**github.com\kataras\iris\iris\main.go**

---
README.md(*github.com\kataras\iris\iris\README.md*)已經圖文並冒，這邊提一下重點就好。


目前有兩個Command [create, run]，並且在#41跟#44把*action**傳進去。

建立專案就看個人選擇吧，先看看好用的run

這個run是runAndWatchCmd，所以我們可以換一下

從
> go run main.go

改成
> iris run main.go

這樣修改專案就可以存檔，自動重啟（訊息如下）

```
A change has been detected, reloading now...ready!

//or

A change has been detected, reloading now...{error message}
```

這個run是怎麼作用的呢？

是使用了[Rizla]這個專案，預設每三秒會監控一次（不能再短了）。

在此同時，我們就先解決了[cli]跟[rizla]。

接下來就把重心在放回iris。

最後就是LOG，是使用他們[自己做的](https://github.com/iris-contrib/color)LOG。

主要是加上顏色，以及設定啟用\停用，API與一般LOG相同。

# 參考

action*

    是一個接收flag參數並回傳error的方法
    這邊使用了Go的First Class的特性