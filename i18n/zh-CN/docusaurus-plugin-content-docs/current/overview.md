---
sidebar_position: 1
---

# Overview

A lightweight progressive Web Framework written in Go.

## Getting Started

### Installation

> Requirement: [Go](https://go.dev/dl/) version **1.19** or above.  

```bash  
go get -u github.com/keepchen/go-sail/v3
```

### Launch your app  
- Copy the following code to the `main.go` file
```go title="main.go"
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
- Run command `go run main.go`  
- Open browser and access url: [http://localhost:8000/hello](http://localhost:8000/hello)  
- Screenshot:  
![/img/launch.png](/img/launch.png)  
- Your app is ready, Enjoy it. :)
