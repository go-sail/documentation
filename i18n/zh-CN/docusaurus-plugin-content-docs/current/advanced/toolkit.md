---
sidebar_position: 3
---

# 工具包  
## 简介  
在大多数情况下，工具包也可以单独使用。 因此，您可以通过“go get”命令轻松导入和使用它们。  
它们的源代码地址位于[github.com/keepchen/go-sail/v3/utils](https://github.com/keepchen/go-sail/tree/main/utils)。  
例如，您可以使用`go get`命令将工具包安装到本地，然后在代码文件的`import`指令处引入。  
```bash showLineNumbers  
go get -u github.com/keepchen/go-sail/v3/utils
```  
## 使用方法  
### RSA  
#### 加密  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.RSAEncrypt(rawString, publicKey)
}
```  
#### 解密    
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.RSADecrypt(rawString, publicKey)
}
```  
## 特例  
### Redis锁  
工具包中，redis锁是一种特例，需要依赖redis库。因此，在使用之前，需要提前实例化redis连接，可以是单机的，也可以是集群的。  
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