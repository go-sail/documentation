---
sidebar_position: 12
---  
# Http  
本章介绍HTTP相关请求响应的整个流程。  
## 简介  
在http请求响应的过程中，从请求到达服务节点，到流程处理，最后到响应给请求发起者，每一步都会根据需要进行处理。  

## 请求  
### 路由中间件  
路由中间件往往在整个请求调用链中起到过滤、拦截、上下文管理的作用，通常用于身份认证等，一般位于请求到达后的第一个位置，处理函数之前。  

现在，让我们实现一个简单的身份验证中间件。  
```go  title="examples/pkg/app/user/http/middleware/authcheck.go" showLineNumbers  
import (
    "github.com/gin-gonic/gin"
    "github.com/keepchen/go-sail/v3/constants"
)
func AuthCheck() gin.HandlerFunc {
    return func(c *gin.Context) {
        authorization := c.GetHeader("Authorization")
        if len(authorization) == 0 {
            sail.Response(c).Builder(constants.ErrAuthorizationTokenInvalid, nil).Send()
            return
        }

        uid := parseUserIDFromAuthorization(authorization)
        c.Set("userID", uid)
        c.Next()
    }
}

func parseUserIDFromAuthorization(authorization string) int64 {
    // TODO
    return int64(123)
}
```  
将其注册到路由中。  
```go  title="examples/pkg/app/user/http/routes/routes.go" showLineNumbers  
import (
    "github.com/gin-gonic/gin"
    "examples/pkg/app/user/http/middleware"
    "examples/pkg/app/user/http/handler"
)

func RegisterRoutes(r *gin.Engine) {
    userGroup := r.Group("/user", middleware.AuthCheck())
    {
        userGroup.GET("info", handler.UserInfo)
    }
}
```  
### 实体  
请求实体定义了请求参数的数据结构，将为后续的参数处理提供明确的数据类型。因此，我们建议明确定义它。同时我们还定义了它的扩展验证方法来验证内部数据是否符合要求。  
```go title="examples/pkg/app/user/http/vo/request/userinforeqvo.go" showLineNumbers  
import (
    sailConstants "github.com/keepchen/go-sail/v3/constants"
)
type UserInfoReqVO struct {
    ShowDetail     bool `json:"showDetail" form:"showDetail" validate:"required" format:"bool" `
    WithWalletInfo bool `json:"withWalletInfo" form:"withWalletInfo" validate:"required" format:"bool" `
}

func (v *UserInfoReqVO) Validator() (sailConstants.ICodeType, error) {
    return sailConstants.ErrNone, nil
}
```  
## 业务逻辑 
### 参数绑定  
```go title="examples/pkg/app/user/http/handler/user.go" showLineNumbers  
import (
    "go.uber.org/zap"
    "github.com/gin-gonic/gin"
    sailConstants "github.com/keepchen/go-sail/v3/constants"
    "github.com/keepchen/go-sail/v3/sail"
    "examples/pkg/app/user/http/vo/request"
)

func UserInfo(c *gin.Context) {
    var (
        ...
        form      request.UserInfoReqVO
        loggerSvc = c.MustGet("logger").(*zap.Logger)
        userID = c.MustGet("userID").(int64)
        ...
    )
    // highlight-start
    if err := c.ShouldBind(&form); err != nil {
    // highlight-end
        sail.Response(c).Failure400(sailConstants.ErrRequestParamsInvalid)
        return
    }
    if code, err := form.Validator(); err != nil {
        loggerSvc.Warn("[UserInfo] form field validate failed", zap.Errors("errors", []error{err}))
        sail.Response(c).Failure400(code)
        return
    }
    ...
}
```
### 验证参数  
```go title="examples/pkg/app/user/http/handler/user.go" showLineNumbers  
import (
    "go.uber.org/zap"
    "github.com/gin-gonic/gin"
    sailConstants "github.com/keepchen/go-sail/v3/constants"
    "github.com/keepchen/go-sail/v3/sail"
    "examples/pkg/app/user/http/vo/request"
)

func UserInfo(c *gin.Context) {
    var (
        ...
        form      request.UserInfoReqVO
        loggerSvc = c.MustGet("logger").(*zap.Logger)
        userID = c.MustGet("userID").(int64)
        ...
    )
    if err := c.ShouldBind(&form); err != nil {
        sail.Response(c).Failure400(sailConstants.ErrRequestParamsInvalid)
        return
    }
    // highlight-start
    if code, err := form.Validator(); err != nil {
    // highlight-end
        loggerSvc.Warn("[UserInfo] form field validate failed", zap.Errors("errors", []error{err}))
        sail.Response(c).Failure400(code)
        return
    }
    ...
}
```  
### 处理函数  
```go title="examples/pkg/app/user/http/handler/user.go" showLineNumbers  
import (
    "go.uber.org/zap"
    "github.com/gin-gonic/gin"
    sailConstants "github.com/keepchen/go-sail/v3/constants"
    "github.com/keepchen/go-sail/v3/sail"
    "examples/pkg/app/user/http/vo/request"
)

func UserInfo(c *gin.Context) {
    var (
        ...
        form      request.UserInfoReqVO
        loggerSvc = c.MustGet("logger").(*zap.Logger)
        userID = c.MustGet("userID").(int64)
        ...
    )
    if err := c.ShouldBind(&form); err != nil {
        sail.Response(c).Failure400(sailConstants.ErrRequestParamsInvalid)
        return
    }
    if code, err := form.Validator(); err != nil {
        loggerSvc.Warn("[UserInfo] form field validate failed", zap.Errors("errors", []error{err}))
        sail.Response(c).Failure400(code)
        return
    }

    // highlight-start
    var user User
    sail.GetDBR().Model(&User{}).Where("id = ?", userID).First(&user)
    // highlight-end
    ...
}
```  
## 响应  
### 实体  
响应实体定义了返回数据的数据结构，将为后续响应处理提供明确的数据类型。 因此，我们建议明确定义它。  
```go title="examples/pkg/app/user/http/vo/response/userinfoackvo.go" showLineNumbers  
import (
    sailConstants "github.com/keepchen/go-sail/v3/constants"
)

type UserInfoAckVO struct {
    UserID   int64  `json:"userId" format:"number" validate:"required" example:"123"`
    Email    string `json:"email" format:"string" validate:"required" example:"go-sail@example.com"`
    Nickname string `json:"nickname" format:"string" validate:"required" example:"go-sail"`
}
```  
### 响应  
```go title="examples/pkg/app/user/http/handler/user.go" showLineNumbers  
import (
    "go.uber.org/zap"
    
    sailConstants "github.com/keepchen/go-sail/v3/constants"
    "github.com/keepchen/go-sail/v3/sail"
    "examples/pkg/app/user/http/vo/request"
    "examples/pkg/app/user/http/vo/response"
)

func UserInfo(c *gin.Context) {
    var (
        ...
        form      request.UserInfoReqVO
        // highlight-start
        resp      resp.UserInfoAckVO
        // highlight-end
        loggerSvc = c.MustGet("logger").(*zap.Logger)
        userID = c.MustGet("userID").(int64)
        ...
    )
    if err := c.ShouldBind(&form); err != nil {
        sail.Response(c).Failure400(sailConstants.ErrRequestParamsInvalid)
        return
    }
    if code, err := form.Validator(); err != nil {
        loggerSvc.Warn("[UserInfo] form field validate failed", zap.Errors("errors", []error{err}))
        sail.Response(c).Failure400(code)
        return
    }

    var user User
    sail.GetDBR().Model(&User{}).Where("id = ?", userID).First(&user)
    ...
    // highlight-start
    resp.UserID = user.ID
    resp.Email = user.Email
    resp.Nickname = user.Nickname

    sail.Response(c).Data(resp)
    // highlight-end
}
```  
## 语法糖  
语法糖提供了更丰富的响应函数。  
```go showLineNumbers  
sail.Response(c).Data(nil)

sail.Response(c).Success()

sail.Response(c).Failure()

sail.Response(c).Failure("failed")

sail.Response(c).Failure400()

sail.Response(c).Failure500()

sail.Response(c).SimpleAssemble(constants.XXX, anyValue, "SUCCESS").Send()
```  

## Api选项  
API选项允许开发人员自定义响应者的行为。  
```go title="github.com/keepchen/go-sail/http/api/option.go" showLineNumbers  
type Option struct {
    ErrNoneCode constants.ICodeType
    ErrNoneCodeMsg string
    EmptyDataStruct int
    ForceHttpCode200 bool
    Timezone string
    DetectAcceptLanguage bool
}
```  
简单解释一下各个字段的用途：  
- **ErrNoneCode**  
    该字段将重新定义“无错误”错误代码。Go-Sail的默认“无错误”错误代码为 0。  
- **ErrNoneCodeMsg**  
    该字段将重新定义“无错误”错误消息。Go-Sail的默认“无错误”错误消息是“SUCCESS”。  
- **EmptyDataStruct**  
    当该字段为空时，将重新定义响应正文中“data”字段的类型。  
- **ForceHttpCode200**  
    无论错误代码是什么，该字段都会将HTTP状态代码重新定义为200。  
- **Timezone**  
    该字段将重新定义服务节点的时区。  
- **DetectAcceptLanguage**  
    该字段将指示是否解析请求头中的“Accept-Language”信息。如果为true，则响应消息将启用多语言检测并返回相应语言的错误消息（如果存在）。  

该选项在框架启动时设置：  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/constants"
    // highlight-start
    "github.com/keepchen/go-sail/v3/http/api"
    // highlight-end
    "github.com/keepchen/go-sail/v3/sail/config"
)

var (
    conf = &config.Config{}
    option = &api.Option{
        Timezone:         constants.DefaultTimeZone,
        ErrNoneCode:      constants.ErrNone,
        ErrNoneCodeMsg:   "SUCCESS",
        ForceHttpCode200: true,
    }
)

func main() {
    sail.
        WakeupHttp("go-sail", conf).
        // highlight-start
        SetupApiOption(option).
        // highlight-end
        Hook(registerRoutes, nil, nil).
        Launch()
}
```  
## 错误码  
Go-Sail框架仅提供少量错误代码常量。在业务开发过程中，开发者需要注册自己的错误码和错误消息，Go-Sail提供的错误码注册功能支持国际化。开发者需要按照规范进行注册才能生效。  
### 注册  
下面给出一个代码示例：  
```go title="examples/pkg/constants/errors.go" showLineNumbers  
import (
    "sync"
    sailConstants "github.com/keepchen/go-sail/v3/constants"
)

const (
    ErrNone                        = sailConstants.CodeType(200)    //没有错误，占位
    ErrStatusGatewayTimeoutTimeOut = sailConstants.CodeType(504)    //超时
    ErrInternalSeverError          = sailConstants.CodeType(999999) //服务器内部错误
    ErrRequestParamsInvalid        = sailConstants.CodeType(100000) //请求参数有误
    ErrAuthorizationTokenInvalid   = sailConstants.CodeType(100001) //令牌已失效
    SliderValidationFailed         = sailConstants.CodeType(8026)   //谷歌验证失败
)

// 错误码信息表
//
// READONLY for concurrency safety
var codeMsgMap = map[sailConstants.LanguageCode]map[sailConstants.ICodeType]string{
    //英语
    sailConstants.LanguageEnglish: {
        ErrNone:                        "SUCCESS",
        ErrStatusGatewayTimeoutTimeOut: "Timeout",
        ErrInternalSeverError:          "Internal server error",
        ErrRequestParamsInvalid:        "Bad request parameters",
        ErrAuthorizationTokenInvalid:   "Token invalid",
        SliderValidationFailed:         "Slider validation failed",
    },
    //更多语言...
}

var once sync.Once

func init() {
    once.Do(func() {
        go func() {
            time.Sleep(time.Second * 5)
            for code, msg := range codeMsgMap {
                // highlight-start
                sailConstants.RegisterCode(code, msg)
                // highlight-end
            }
        }()
    })
}
```  
### 使用方法  
现在您可以使用它们来响应：  
```go title="examples/pkg/app/user/http/handler/user.go" showLineNumbers  
import (
    "github.com/gin-gonic/gin"
    "github.com/keepchen/go-sail/v3/sail"
    // highlight-start
    "examples/pkg/constants"
    // highlight-end
)

func UserInfo(c *gin.Context) {
    ...
    // highlight-start
    sail.Response(c).Failure200(constants.SliderValidationFailed)
    // highlight-end
}
```

