---
sidebar_position: 2
---

# Libraries  
## Introduction  
Library packages are designed from the outset to be imported and run independently. Therefore, you can easily import and use them through the `go get` command.  
Their source code addresses are located at [github.com/keepchen/v3/lib](https://github.com/keepchen/go-sail/tree/main/lib).  

Taking redis as an example, you can use the `go get` command to install the redis library locally, and then introduce it at the `import` instruction of the code file.  
```bash showLineNumbers  
go get -u github.com/keepchen/go-sail/v3/lib/redis
```  
## Usage  
### Redis  
#### Standalone
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
#### Cluster  
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
#### Initialize new instance  
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