---
sidebar_position: 5
---  
# Nacos  
This chapter contains nacos usage examples.  
## Introduction  
The nacos component is a secondary encapsulation of `nacos-group/nacos-sdk-go/v2`. This component encapsulates some of the most commonly used methods, such as obtaining configuration, monitoring configuration, registering services, uninstalling services, obtaining healthy instances, etc.  
## Usage  
### Initial client  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/lib/nacos"
)

func main() {
    // highlight-start
    nacos.InitClient("appName", "endpoints", "namespace id")
    // highlight-end
}
```  
### Get configuration  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/lib/nacos"
    sailConfig "github.com/keepchen/go-sail/v3/sail/config"
)

func main() {
    var conf = &sailConfig.Config{}
    // highlight-start
    err := nacos.GetConfig(group, dataID, conf, "yaml")
    // highlight-end
}
```  
### Listen configuration  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/lib/nacos"
    sailConfig "github.com/keepchen/go-sail/v3/sail/config"
)

func main() {
    var conf = &sailConfig.Config{}
    // highlight-start
    callback := func(namespace, group, dataId, data string) {
        err = nacos.ParseConfig([]byte(data), conf, "yaml")
        if err != nil {
            fmt.Printf("<Nacos> listen config {%s:%s} change,but can't be unmarshal: %s\n", group, dataId, err.Error())
            return
        }
    }
    // highlight-end

    // highlight-start
    //listen config if it changed
    err = nacos.ListenConfigWithCallback(group, dataID, callback)
    // highlight-end
    if err != nil {
        panic(err)
    }
}
```  
### Register service  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/lib/nacos"
)

func main() {
    ok, err := nacos.RegisterService(groupName, serviceName, ip, port, metadata)
}
```  
### Unregister service  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/lib/nacos"
)

func main() {
    ok, err := nacos.UnregisterService(groupName, serviceName, ip, port)
}
```  
### Get healthy instance
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/lib/nacos"
)

func main() {
    // highlight-start
    serviceUrl := nacos.GetHealthyInstanceUrl(group, serviceName, sail.GetLogger())
    // highlight-end
    if len(serviceUrl) == 0 {
        sail.GetLogger().Warn("no healthy instances")
        return ""
    }
}
```  

### Others  
For more native calling methods, please view the official documentation of [nacos-group/nacos-sdk-go/v2](https://github.com/nacos-group/nacos-sdk-go).  


