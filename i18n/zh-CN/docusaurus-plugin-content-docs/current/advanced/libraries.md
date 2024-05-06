---
sidebar_position: 2
---

# 类库  
## 简介  
库包从一开始就设计为可导入和独立运行。 因此，您可以通过“go get”命令轻松导入和使用它们。
它们的源代码地址位于[github.com/keepchen/go-sail/v3/lib](https://github.com/keepchen/go-sail/tree/main/lib)。  
以redis为例，可以使用go get命令将redis库安装到本地，然后在代码文件的import指令处引入。    
```bash showLineNumbers  
go get -u github.com/keepchen/go-sail/v3/lib/redis
```  
## 使用方法  
### Redis  
#### 单机  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/lib/redis"
)

func main() {
    ...
    var conf = redis.Conf{}
    redis.InitRedis(conf)
    ...

    otherFunc()
}

func otherFunc() {
    ...
    result, err := redis.GetInstance().Get(ctx, key).Result()
    ...
}
```  
#### 集群  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/lib/redis"
)

func main() {
    ...
    var conf = redis.ClusterConf{}
    redis.InitRedisCluster(conf)
    ...

    otherFunc()
}


func otherFunc() {
    ...
    result, err := redis.GetClusterInstance().Get(ctx, key).Result()
    ...
}
```  
#### 初始化新连接  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/lib/redis"
)

func main() {
    ...
    var conf = redis.ClusterConf{}
    clusterInstance := redis.NewCluster(conf)
    ...

    clusterInstance.Get(ctx, key).Result()
}
```  