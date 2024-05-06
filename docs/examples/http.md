---
sidebar_position: 12
---  
# Http  
This chapter introduces the entire process of HTTP-related request responses.  
## Introduction  
In the process of http request response, from the request arriving at the service node, to the process processing, and finally to the response to the request initiator, each step will be processed as needed.  

## Request  
### Routing middleware  
Routing middleware often plays the role of filtering, interception, and context management in the entire request call chain, and is usually used for identity authentication, etc. It is generally located at the first position after the request arrives, before the processing function.  

Now, let's implement a simple authentication middleware.  
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
Register it with the route.  
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
### Entities  
The request entity defines the data structure of the request parameters and will provide clear data types for subsequent parameter processing. Therefore, we recommend defining it explicitly. At the same time, we also defined its extended verification method to verify whether the internal data meets the requirements.  
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
## Logic 
### Parameters binding  
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
### Validator  
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
### Handler  
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
## Response  
### Entities  
The response entity defines the data structure of the returned data and will provide clear data types for subsequent response processing. Therefore, we recommend defining it explicitly.  
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
### Response  
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
## Syntactic sugar  
Syntactic sugar provides richer response functions.  
```go showLineNumbers  
sail.Response(c).Data(nil)

sail.Response(c).Success()

sail.Response(c).Failure()

sail.Response(c).Failure("failed")

sail.Response(c).Failure400()

sail.Response(c).Failure500()

sail.Response(c).SimpleAssemble(constants.XXX, anyValue, "SUCCESS").Send()
```  

## Api Option  
API options allow developers to customize the behavior of responders.  
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
Briefly explain the purpose of each field:  
- **ErrNoneCode**  
    This field will redefine the 'no error' error code. Go-Sail's default 'no error' error code is 0.  
- **ErrNoneCodeMsg**  
    This field will redefine the 'no error' error message. Go-Sail's default 'no error' error message is 'SUCCESS'.  
- **EmptyDataStruct**  
    This field will redefine the type of the 'data' field in the response body when it is empty.  
- **ForceHttpCode200**  
    This field will redefine the HTTP status code to be 200 no matter what the error code is.  
- **Timezone**  
    This field will redefine the time zone of the service node.  
- **DetectAcceptLanguage**  
    This field will indicate whether to parse the 'Accept-Language' information in the request header. If true, the response message will enable multi-language detection and return an error message for the corresponding language (if one exists).   


This option is set when the framework starts:  
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
## Error code  
The Go-Sail framework only provides a small number of error code constants. During the business development process, developers need to register their own error codes and error messages, and the error code registration function provided by Go-Sail supports internationalization. Developers need to register in accordance with the specifications to take effect.  
### Register  
A code example is given below:  
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
### Usage  
Now you can use them in response:  
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

