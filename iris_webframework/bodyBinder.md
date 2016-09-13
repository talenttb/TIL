# [Body Binder](http://iris-go.com/body_binder/)

我們使用API再對應資料時，如果要每個自己來處理會多很多有點雜的程式碼，所以使用這種方式可以大量減少一般在做對應的事情

那對應的物件就三種
* JSON(ReadJSON)
* XML(ReadXML)
* Form(ReadForm)

使用方式其實也很簡單，上述的括弧就是iris提供的方法名稱

然後將我們自定義的物件放進去即可

```go
type Company struct {
   Public     bool      `form:"public"`
   Name       string`json:"myName"`
   Location   struct {
     Country  string
     City     string
   }
   Products   []struct {
     Name string
     Type string
   }
   Founders   []string
   Employees  int64
}

func MyHandler(c *iris.Context) {
  if err := c.ReadJSON(&Company{}); err != nil {
    panic(err.Error())
  }
}

```

這邊Parse JSON時，form或json的annotation都是可以的，所以也可以在後面將接的名稱換掉

但這邊呼叫的是內建的json.NewDecoder，這個方法在Go1.6以前有人發issue認為處理太慢，如果有需要可以直接另外處理，程式也不難

另外範例使用time.Time還有url.URL我在JSON轉換的時候會一直出錯，在Time的格式也將[Go所支援的格式](https://golang.org/pkg/time/#pkg-constants)放下去測，結果一樣轉失敗

目前就手動轉吧

這邊一樣可以使用struct extend的方式，他一樣也會幫你做轉型的動作