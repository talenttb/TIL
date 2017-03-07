# 註記在ＧＲＰＣ尚未使用的功能或細節

## 細節

protoc 參數

	-I 参数：指定import路径，可以指定多个-I参数，编译时按顺序查找，不指定时默认查找当前目录

	--go_out ：golang编译支持，支持以下参数
	plugins=plugin1+plugin2 - 指定插件，目前只支持grpc，即：plugins=grpc

	M 参数 - 指定导入的.proto文件路径编译后对应的golang包名(不指定本参数默认就是.proto文件中import语句的路径)

	import_prefix=xxx - 为所有import路径添加前缀，主要用于编译子目录内的多个proto文件，这个参数按理说很有用，尤其适用替代一些情况时的M参数，但是实际使用时有个蛋疼的问题导致并不能达到我们预想的效果，自己尝试看看吧

	import_path=foo/bar - 用于指定未声明package或go_package的文件的包名，最右面的斜线前的字符会被忽略

	末尾 :编译文件路径 .proto文件路径(支持通配符)

## 功能

* 保留字段及標識
```
message Foo {
    reserved 2, 15, 9 to 11;
    reserved "foo", "bar";
}
```

* Middleware
```
var interceptor grpc.UnaryServerInterceptor
```

* Trace
```
func main {

    go startTrace()

    grpclog.Println("Listen on " + Address)
    s.Serve(listen)
}

func startTrace() {
    trace.AuthRequest = func(req *http.Request) (any, sensitive bool) {
        return true, true
    }
    go http.ListenAndServe(":50051", nil)
    grpclog.Println("Trace listen on 50051")
}
```


[參考](https://segmentfault.com/a/1190000007880647)