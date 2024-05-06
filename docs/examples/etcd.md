---
sidebar_position: 9
---  
# Etcd  
This chapter contains etcd usage examples.  
## Introduction  
The Etcd component pair is a simple encapsulation of the `go.etcd.io/etcd` library, allowing developers to ignore connection details and providing a simple listening key function.  
When Go-Sail starts, it will automatically initialize the Etcd component if enabled. After that, developers can call it directly through the `sail` keyword.  
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
## Configuration  
In the previous chapter, we learned about the detailed configuration of Redis (standalone and cluster), which is as follows:  
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
## Usage  
### Watch key  
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
### Watch key with prefix  
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
### Others  
For more native calling methods, please view the official documentation of [go.etcd.io/etcd/client/v3](https://pkg.go.dev/go.etcd.io/etcd/client/v3).  

