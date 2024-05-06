---
sidebar_position: 4
---  
# Redis  
这个章节将介绍Redis的使用方法。  
## 简介  
redis组件是`go-redis/redis/v8`的二次封装。 该组件只封装了redis的连接处理和日志处理，其余内容均为原生调用。  
由于`go-redis/redis/v8`库为独立模式和集群模式提供了两种不同的客户端实例，因此redis组件也提供了两种客户端实例。  
当Go-Sail启动时，如果启用，它将自动初始化redis组件。之后开发者可以直接通过`sail`关键字来调用。  
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
## 配置  
上一章我们学习了Redis的详细配置（单机和集群），如下：  
### 单机  
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
需要注意的是，这部分配置不支持热更新。  
:::  
### 集群  
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
需要注意的是，这部分配置不支持热更新。  
:::  

## 使用方法  
### 单机  
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
### 集群  
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

### 其他  
更多原生调用方法请查看[redis/go-redis/v8](https://github.com/redis/go-redis)的官方文档。  


