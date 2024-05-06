---
sidebar_position: 2
---

# Lifecycle  
Explain how Go-Sail works and its lifecycle.  

## How it works  
The start and stop of Go-Sail will follow a certain sequence, and specific things will be done at different stages.

## Go-Sail's Lifecycle

### Initial configuration  
First, Go-Sail will inject the necessary configuration into the program stack for subsequent use.  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail/config"
)

var conf = &config.Config{}
```  
The `config.Config{}` configuration contains complete configuration items required by Go-Sail, and you can set the contents as needed.  
Let's take the configuration of the http service as an example. You can set the port that the service listens to, set whether the service runs in debug mode, whether to enable swagger documentation, etc.  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail/config"
)

var (
    conf = &config.Config{
        // highlight-start
        HttpServer: config.HttpServerConf{
            Debug: true,
            Addr:  ":8000",
        },
        // highlight-end
    }
)

func main() {
    sail.
        // highlight-start
        WakeupHttp("go-sail", conf).
        // highlight-end
        Hook(registerRoutes, nil, nil).
        Launch()
}
```  

### Set startup items  
Secondly, you need to set the necessary startup items, which will determine how some functions of Go-Sail will work.  
#### Api option (optional)  
```go title="main.go" showLineNumbers  
import (
    // highlight-start
    "github.com/keepchen/go-sail/v3/constants"
    "github.com/keepchen/go-sail/v3/http/api"
    // highlight-end
    "github.com/keepchen/go-sail/v3/sail/config"
)

var (
    conf = &config.Config{
        HttpServer: config.HttpServerConf{
            Debug: true,
            Addr:  ":8000",
        },
    }
)

func main() {
    sail.
        WakeupHttp("go-sail", conf).
        // highlight-start
        SetupApiOption(&api.Option{
                Timezone:         constants.DefaultTimeZone,
                ErrNoneCode:      constants.ErrNone,
                ErrNoneCodeMsg:   "SUCCESS",
                ForceHttpCode200: true,
            }).
        // highlight-end
        Hook(registerRoutes, nil, nil).
        Launch()
}
```  

#### Enable websocket (optional)  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail/config"
)

var (
    conf = &config.Config{
        HttpServer: config.HttpServerConf{
            Debug: true,
            Addr:  ":8000",
        },
    }
)

func main() {
    sail.
        WakeupHttp("go-sail", conf).
        // highlight-start
        EnableWebsocket(nil, nil, nil).
        // highlight-end
        Hook(registerRoutes, nil, nil).
        Launch()
}
```  

#### Hook functions (required)  
```go title="main.go" showLineNumbers  
import (
    // highlight-start
    "github.com/gin-gonic/gin"
    // highlight-end
    "github.com/keepchen/go-sail/v3/sail"
    "github.com/keepchen/go-sail/v3/sail/config"
)

var (
    conf = &config.Config{
        LoggerConf: logger.Conf{
            Filename: "logs/running.log",
        },
        HttpServer: config.HttpServerConf{
            Debug: true,
            Addr:  ":8000",
        },
    }
    // highlight-start
    registerRoutes = func(ginEngine *gin.Engine) {
        ginEngine.GET("/hello", func(c *gin.Context){
            c.String(http.StatusOK, "%s", "hello, world!")
        })
    }
    beforeFunc = func() {
        fmt.Println("call user function [before] to do something...")
    }
    afterFunc = func() {
        fmt.Println("call user function [after] to do something...")
    }
    // highlight-end
)

func main() {
    sail.WakeupHttp("go-sail", conf).
        // highlight-start
        Hook(registerRoutes, beforeFunc, afterFunc).
        // highlight-end
        Launch()
}
```  
### Launch  
When the `Launch()` function is called, Go-Sail will perform startup naming and start serving you.  
Go-Sail will execute in **sequence**:  
- **Execute `beforeFunc` (optional)**  
You can perform certain operations within this function. This function is called before the service starts.  
    :::danger  
    At this stage, the component has not been initialized yet, so no component instance can be called within this function, otherwise it will panic.
    :::
- **Startup components**  
At this stage, Go-Sail will start the responding component or service based on the content of the configuration file. For example: initialize the log library, initialize the database connection, initialize the redis connection, etc.  

- **Initial routing service (gin)**  
At this stage, Go-Sail will initialize the gin engine to prepare for subsequent route registration.  

- **Register websocket service**  
At this stage, Go-Sail will initiate a websocket connection and register it with the routing engine. Of course, this stage is optional.  

- **Start pprof**  
Only enabled if specified in the configuration file.  

- **Start Prometheus metric collection**  
Only enabled if specified in the configuration file.  

- **Start the Swagger document service**    
Only enabled if specified in the configuration file.  

- **Start the routing service and listen for http requests**  

- **Print overview information to the terminal**  
Maybe like this:  
![screenshot](/img/launch.png)

- **Execute `afterFunc` (optional)**  
    :::tip  
    At this stage, the relevant components have been initialized, and you can perform the functions you want in this function, such as initializing the database table structure, table data, scheduled tasks, etc.
    :::

- **Monitor system signals**  
At this stage, Go-Sail will continue to listen for system signals and will not start the exit operation until it receives the exit signal.  

- **Shutdown components**  
After receiving the exit signal, Go-Sail will shut down the previously started components or services in sequence.  

- **The entire process exits**  

## Request lifecycle  
The request life cycle represents the entire process of an http request from arriving at the service node, to the intermediate processing, and finally to the final response. It describes the entire process and events that occurred.  

### Routing middleware  
When a request reaches the service node, it will first pass through a series of routing middleware, and the context of the request will be processed in these middleware.  

#### Log trace  
First, when the request reaches the service node, it will be monitored and captured by the gin engine. Then, Go-Sail will take over the request to a routing middleware called LogTrace. In this middleware, some necessary log tracking information will be injected. into the context of the request for subsequent access.  
```go title="github.com/keepchen/go-sail/sail/httpserver/gin.go" showLineNumbers  
import (
    // highlight-start
    "github.com/keepchen/go-sail/v3/http/middleware"
    // highlight-end
    "github.com/gin-gonic/gin"
)

func InitGinEngine(conf config.HttpServerConf) *gin.Engine {
    var r *gin.Engine

    ...

    // highlight-start
    r.Use(middleware.LogTrace())
    // highlight-end

    ...
}
```  
#### Prometheus exporter  
The Prometheus exporter middleware will inject some numerical indicators about the request response into the request context. This middleware will be used in conjunction with the Prometheus service.  
:::tip  
This middleware is optional, you can enable or disable it according to your needs by specifying the configuration file.
:::  
```go title="github.com/keepchen/go-sail/sail/httpserver/gin.go" showLineNumbers  
import (
    // highlight-start
    "github.com/keepchen/go-sail/v3/http/middleware"
    // highlight-end
    "github.com/gin-gonic/gin"
)

func InitGinEngine(conf config.HttpServerConf) *gin.Engine {
    var r *gin.Engine

    ...

    // highlight-start
    if conf.Prometheus.Enable {
        r.Use(middleware.PrometheusExporter())
    }
    // highlight-end

    ...
}
```  
#### Other middlewares  
In addition, the request will also go through some other middleware, which are specified by the developer, such as Cors, Gzip, etc. Of course, these middleware are also optional and are fully specified by the developer.  

### Handler  
After passing through a series of routing middleware, the request will reach the routing processing function, which is usually the specific business processing code.  
Generally, there are the following processing steps:  
#### Parameter binding  
Bind request parameters to go code.  
#### Parameter verification  
Verify that the request parameters meet the conditions.  
#### Business processing  
Process business logic, such as querying qualified records in the database based on request parameters, etc.  
#### Response  
After processing the business logic, respond the processing results to the client (the party that initiated the request).  

### Ending  
At this point, the entire process from reaching the service node, to processing, and finally to the response is complete.  

