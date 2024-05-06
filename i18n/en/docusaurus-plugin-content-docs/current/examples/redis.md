---
sidebar_position: 4
---  
# Redis  
This chapter contains redis usage examples.  
## Introduction  
The redis component is a secondary encapsulation of `go-redis/redis/v8`. This component only encapsulates the connection processing and log processing of the redis, and the rest of the contents are native calls.  
Since the `go-redis/redis/v8` library provides two different client instances for standalone mode and cluster mode, the redis component also provides two client instances.   
When Go-Sail starts, it will automatically initialize the Redis component if enabled. After that, developers can call it directly through the sail keyword.  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    // highlight-start
    standalone := sail.GetRedis()
    cluster := sail.GetRedisCluster()
    // highlight-end
}
```  
## Configuration  
In the previous chapter, we learned about the detailed configuration of Redis (standalone and cluster), which is as follows:  
### Standalone  
```go title="github.com/keepchen/go-sail/lib/redis/conf.go" showLineNumbers  
type Conf struct {
    Endpoint  `yaml:"endpoint" toml:"endpoint" json:"endpoint"`
    Enable    bool `yaml:"enable" toml:"enable" json:"enable" default:"false"` //是否启用
    Database  int  `yaml:"database" toml:"database" json:"database"`           //数据库名
    SSLEnable bool `yaml:"ssl_enable" toml:"ssl_enable" json:"ssl_enable"`     //是否启用ssl
}

type Endpoint struct {
    Host     string `yaml:"host" toml:"host" json:"host" default:"localhost"` //主机地址
    Port     int    `yaml:"port" toml:"port" json:"port" default:"6379"`      //端口
    Username string `yaml:"username" toml:"username" json:"username"`         //用户名
    Password string `yaml:"password" toml:"password" json:"password"`         //密码
}
```  
:::warning  
It should be noted that this part of the configuration does not support hot update.  
:::  
### Cluster  
```go title="github.com/keepchen/go-sail/lib/redis/conf.go" showLineNumbers  
type ClusterConf struct {
    Enable    bool       `yaml:"enable" toml:"enable" json:"enable" default:"false"` //是否启用
    SSLEnable bool       `yaml:"ssl_enable" toml:"ssl_enable" json:"ssl_enable"`     //是否启用ssl
    Endpoints []Endpoint `yaml:"endpoints" toml:"endpoints" json:"endpoints"`        //连接地址列表
}

type Endpoint struct {
    Host     string `yaml:"host" toml:"host" json:"host" default:"localhost"` //主机地址
    Port     int    `yaml:"port" toml:"port" json:"port" default:"6379"`      //端口
    Username string `yaml:"username" toml:"username" json:"username"`         //用户名
    Password string `yaml:"password" toml:"password" json:"password"`         //密码
}
```  
:::warning  
It should be noted that this part of the configuration does not support hot update.  
:::  

## Usage  
### Standalone  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    // highlight-start
    standalone := sail.GetRedis()
    // highlight-end
    result, err := standalone.Get(ctx, key).Result()
}
```  
### Cluster  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    // highlight-start
    cluster := sail.GetRedisCluster()
    // highlight-end
    result, err := cluster.Get(ctx, key).Result()
}
```  

### Others  
For more native calling methods, please view the official documentation of [redis/go-redis/v8](https://github.com/redis/go-redis).  


