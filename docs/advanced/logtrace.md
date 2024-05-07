---
sidebar_position: 6
---

# Log tracing  
This chapter introduces log tracing throughout the entire call chain.  
## Introduction  
We introduced the data structure of Go-Sail with unified response in the chapter [Unified Response](../concepts/http-toolkit.md). One of the fields is `requestId`, which specifies the purpose of this request. A unique identifier through which the entire calling process can be traced. So how is this done?  
Next we will learn more about the principles.    
In the [Lifecycle](../concepts/lifecycle.md) chapter, we introduced that when Go-Sail is started, a routing middleware named `LogTrace` will be injected. When the request arrives, Injects a series of content into the request context. Including request identifier, entry timestamp, etc.    
:::tip  
You can view the specific source code [here](https://github.com/keepchen/go-sail/blob/main/http/middleware/logtrace.go).  
:::  
Among them, LogTrace injects a logger instance with a request identifier into the context, which can be used by subsequent call chains.    
## Usage  
For example, the processing function mentioned in the previous chapter:  
```go title="examples/pkg/app/user/handler/userinfo.go" showLineNumbers  
package handler

import (
    "errors"

    "github.com/gin-gonic/gin"
    "github.com/keepchen/go-sail/v3/constants"
    "github.com/keepchen/go-sail/v3/sail"
    "github.com/keepchen/go-sail/v3/examples/pkg/app/user/http/vo/request"
    "github.com/keepchen/go-sail/v3/examples/pkg/app/user/http/vo/response"
    userSvc "examples/pkg/common/db/service/user"
    "github.com/keepchen/go-sail/v3/lib/logger"
    "go.uber.org/zap"
    "gorm.io/gorm"
)

func GetUserInfoSvc(c *gin.Context) {
    var (
        ...
        // highlight-start
        loggerSvc = c.MustGet("logger").(*zap.Logger)
        // highlight-end
        ...
    )
    if err := c.ShouldBind(&form); err != nil {
        sail.Response(c).Assemble(constants.ErrRequestParamsInvalid, nil).Send()
        return
    }

    if errorCode, err := form.Validator(); err != nil {
        sail.Response(c).Assemble(errorCode, nil, err.Error()).Send()
        return
    }

    // highlight-start
    user, sqlErr := userSvc.NewUserSvcImpl(sail.GetDBR(), sail.GetDBW(), loggerSvc).GetUser(form.UserID)
    // highlight-end
    if sqlErr != nil && errors.Is(sqlErr, gorm.ErrRecordNotFound) {
        sail.Response(c).Assemble(constants.ErrRequestParamsInvalid, nil, "user not found").Send()
        return
    }

    ...

    sail.Response(c).Assemble(constants.ErrNone, resp).Send()
}
```  
At this time, the request log with the unique identifier has been passed from the routing middleware to the processing function, and then passed to the database operation layer by the `NewUserSvcImpl` method. You may have thought that by passing this logger, you can easily connect the call chain.  
That's right! That's it.  
Combined with the exporter provided by the logger component, you can easily output the log to anywhere you want, such as ELK, and then obtain the complete log information by requesting a unique identifier query in the ELK query interface.    
:::tip  
If your service is in a microservice environment, you can also chain calls between different services by passing this unique request ID.  
At this time, do you think of telemetry tools such as **zipkin** and **skywalking**?   
:::