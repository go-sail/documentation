---
sidebar_position: 3
---

# Toolkit  
## Introduction  
In most cases, toolkits can also be used individually. Therefore, you can easily import and use them through the `go get` command.  
Their source code addresses are located at [github.com/keepchen/v3/utils](https://github.com/keepchen/go-sail/tree/main/utils).  
For example, you can use the `go get` command to install the toolkit locally, and then introduce it at the `import` instruction of the code file.  
```bash showLineNumbers  
go get -u github.com/keepchen/go-sail/v3/utils
```  
## Usage  
### RSA  
#### Encrypt  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.RSAEncrypt(rawString, publicKey)
}
```  
#### Decrypt    
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.RSADecrypt(rawString, publicKey)
}
```  
## Special case  
### Redis lock  
In the toolkit, redis lock is a special case and needs to depend on the redis library. Therefore, before using it, you need to instantiate the redis connection in advance, either standalone or cluster.  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/lib/redis"
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    ...
    // highlight-start
    var conf = redis.ClusterConf{}
    redis.InitRedisCluster(conf)
    // highlight-end
    ...

    otherFunc()
}

func otherFunc() {
    ...
    ok := utils.TryLock(key)
    ...
}
```  