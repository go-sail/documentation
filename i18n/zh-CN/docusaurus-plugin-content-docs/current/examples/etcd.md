---
sidebar_position: 9
---  
# Etcd  
这个章节将介绍Etcd组件如何使用。  
## 简介  
Etcd组件对是对`go.etcd.io/etcd`库的简单封装，允许开发者忽略连接细节并提供简单的监听关键功能。  
当Go-Sail启动时，如果启用Etcd组件，它将自动初始化Etcd组件。之后开发者就可以直接通过`sail`关键字来调用它。  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    // highlight-start
    instance := sail.GetEtcdInstance()
    // highlight-end
}
```  
## 配置  
上一章我们学习了Etcd的详细配置，如下：  
```go title="github.com/keepchen/go-sail/lib/etcd/conf.go" showLineNumbers  
type Conf struct {
    Enable    bool        `yaml:"enable" toml:"enable" json:"enable" default:"false"` //是否启用
    Endpoints []string    `yaml:"endpoints" toml:"endpoints" json:"endpoints"`        //地址列表,如: localhost:2379
    Username  string      `yaml:"username" toml:"username" json:"username"`           //账号
    Password  string      `yaml:"password" toml:"password" json:"password"`           //密码
    Timeout   int         `yaml:"timeout" toml:"timeout" json:"timeout"`              //连接超时时间（毫秒）默认10000ms
    Tls       *tls.Config `yaml:"-" toml:"-" json:"-"`                                //tls配置
}
```  
## 使用方法  
### 监听Key  
```go title="main.go" showLineNumbers  
var fn = func(k, v []byte) {
    fmt.Printf("key: %s changed: %s\n", string(k), string(v))
}

func main() {
    // highlight-start
    sail.GetEtcdInstance().Watch(key, fn)
    // highlight-end
}
```  
### 通过前缀监听Key  
```go title="main.go" showLineNumbers  
var fn = func(k, v []byte) {
    fmt.Printf("key: %s changed: %s\n", string(k), string(v))
}

func main() {
    // highlight-start
    sail.GetEtcdInstance().WatchWithPrefix(key, fn)
    // highlight-end
}
```  
### 其他  
更多原生调用方法请查看[go.etcd.io/etcd/client/v3](https://pkg.go.dev/go.etcd.io/etcd/client/v3)的官方文档。  

