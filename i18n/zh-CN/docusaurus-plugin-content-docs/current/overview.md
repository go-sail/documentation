---
sidebar_position: 1
---

# 概览

一个轻量的渐进式golang web框架。

## 准备开始  

### 安装  

> Requirement: [Go](https://go.dev/dl/) version **1.19** or above.  

```bash  showLineNumbers  
go get -u github.com/keepchen/go-sail/v3
```

### 启动你的服务  
- 将下面的代码拷贝到`main.go`文件中  
```go title="main.go" showLineNumbers  
import (
    "github.com/gin-gonic/gin"
    "github.com/keepchen/go-sail/v3/sail"
    "github.com/keepchen/go-sail/v3/sail/config"
)

var (
    conf = &config.Config{}
    registerRoutes = func(ginEngine *gin.Engine) {
        ginEngine.GET("/hello", func(c *gin.Context){
            c.String(http.StatusOK, "%s", "hello, world!")
        })
    }
)

func main() {
    sail.WakeupHttp("go-sail", conf).Hook(registerRoutes, nil, nil).Launch()
}
```  
- 运行命令`go run main.go`  
- 打开浏览器并访问： [localhost:8080/hello](http://localhost:8080/hello)  
- 屏幕截图:  
![screenshot](/img/launch.png)  
- 你的服务已经准备就绪，祝你用的开心。 :)
