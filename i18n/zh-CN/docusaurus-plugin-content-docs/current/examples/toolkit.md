---
sidebar_position: 13
---  
# 工具包  
这个章节将介绍工具包如何使用。  
## Introduction  
Go-Sail提供了一个工具包，包含一些常用的工具功能。  
## AES  
### 编码  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    encoded, err := utils.AesEncode(rawStr, key)
}
```  
### 解码  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    decoded, err := utils.AesDecode(encoded, key)
}
```  
## Base64  
### 编码  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    encoded, err := utils.Base64Encode(rawBytes)
}
```  
### 解码  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    decoded, err := utils.Base64Decode(encoded)
}
```  
## CRC  
### Checksum 32
```go title="main.go"  showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.Crc32Checksum(rawBytes, table)
}
```  
### Checksum 64
```go title="main.go"  showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.Crc64Checksum(rawBytes, table)
}
```  
### Checksum ECMA
```go title="main.go"  showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.Crc64ChecksumECMA(rawBytes)
}
```  
### Checksum IEEE
```go title="main.go"  showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.Crc32ChecksumIEEE(rawBytes)
}
```  
## Datetime  
### 格式化日期
```go title="main.go" showLineNumbers  
import (
    "time"
    "github.com/keepchen/go-sail/v3/constants"
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    now := time.Now()
    result := utils.FormatDate(now, utils.YYYYMMDDHHMMSS)

    result := utils.FormatDate(now, utils.YYYY_MM_DD_HH_MM_SS_SSS)
}
```  
### 解析日期
```go title="main.go" showLineNumbers  
import (
    "time"
    "github.com/keepchen/go-sail/v3/constants"
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    loc, _ := time.LoadLocation(constants.DefaultTimeZone)
    date := "2024-05-01 10:00:00"
    timeObj, err := utils.ParseDate(date, string(utils.YYYY_MM_DD_HH_MM_SS_SSS), loc)
}
```  
## 文件  
### 保存到目的地  
> from gin.Context  

```go title="examples/pkg/app/user/http/handler/user.go" showLineNumbers  
import (
    "github.com/gin-gonic/gin"
    "github.com/keepchen/go-sail/v3/utils"
)

func UserInfo(c *gin.Context) {
    ...
    fileheader, _ := c.FormFile("filename")
    err := utils.SaveFile2Dst(filehader, "path/to/filename")
    ...
}
```  
### 获取内容  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.FileGetContents("path/to/filename")
}
```  
### 写入内容  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    err := utils.FilePutContents(content, "path/to/filename")
}
```  
### 追加内容  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    err := utils.FileAppendContents(content, "path/to/filename")
}
```  
### 断言是否存在
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    ok := utils.FileExists("path/to/filename")

    ok, err := utils.FileExistsWithError("path/to/filename")
}
```  
### 获取扩展名
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    ext := utils.FileExt("path/to/filename")
}
```  
### 逐行读取  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    readCh, err := utils.FileGetContentsReadLine("path/to/filename")
    for content := range readCh {
        fmt.Println(content)
    }
}
```  
## IP  
### 获取本地ip  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    localIp, err := utils.GetLocalIP()
}
```  
## MD5  
### 编码
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    encoded := utils.MD5Encode(rawStr)
}
```  
## Redis锁  
:::tip  
redis锁会在内部自动更新，开发者无需关心内部细节。  
:::  
:::warning  
您必须首先使用“redis.InitRedis”或“redis.InitRedisCluster”初始化连接。  
:::  
### Try lock  
> 非阻塞式的  

```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    ok := utils.RedisTryLock(key)
}
```  
### Acquire lock  
> 阻塞式的  

```go title="main.go" showLineNumbers  
import (
    "context"
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    ctx, cancel := context.WithDeadline(context.Background())
    go func(){
        for range ctx.Deadline() {
            cancel()
        }
    }()
    utils.RedisLock(ctx, key)
}
```  
### Unlock  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    utils.RedisUnlock(key)
}
```  
## RSA  
### 加密  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.RSAEncrypt(rawString, publicKey)
}
```  
### 解密    
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.RSADecrypt(rawString, publicKey)
}
```  
## 信号  
### 监听系统信号  
> 阻塞式的  

```go title="main.go" showLineNumbers  
import (
    "sync"
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    ...
    wg := &sync.WaitGroup{}
    ...
    result, err := utils.ListeningExitSignal(wg)
}
```  
## SM4  
### 加密  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.SM4ECBEncrypt(hexKey, rawStr)
}
```  
### 解密  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.SM4ECBDecrypt(hexKey, rawStr)
}
```  
## 字符串  
### 包装字符  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.Wordwrap(rawStr, 64, "\n")
}
```  
### 包装Redis键  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.WrapRedisKey(appName, key)
}
```  
### 随机字母串  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.RandomLetters(length)
}
```  
### 随机数字串  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.RandomDigitalChars(length)
}
```  
### 随机复合字符  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.RandomComplexString(length)
}
```  
### 反转  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.StringReverse(rawStr)
}
```  
### 打乱  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.StringShuffle(rawStr)
}
```  
### 填充（左侧）  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.StringPaddingLeft(rawStr, padChar, length)
}
```  
### 填充（右侧）  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.StringPaddingRight(rawStr, padChar, length)
}
```  
### 填充（两侧）  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.StringPaddingBoth(rawStr, padChar, length)
}
```  
## Swagger  
### 打印概览注释  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.PrintSwaggerSummaryInfo(param)
}
```  
### 打印处理函数注释  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.PrintSwaggerControllerInfo(param)
}
```  
## Time  
> New time instance with timezone  

```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/pkg/constants"
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.NewTimeWithTimeZone(constants.DefaultTimeZone).Now().Date()

    result := utils.NewTimeWithTimeZone(constants.DefaultTimeZone).Now().Time()

    result := utils.NewTimeWithTimeZone(constants.DefaultTimeZone).Now().DateTime()
}
```  
## 验证器  
### 邮箱号  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    ok := utils.ValidateEmail(email)
}
```  
### 身份证  
> 适用于中国  

```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    ok := utils.ValidateIdentityCard(idCard)
}
```  
## 版本  
### 打印软件版本  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    utils.PrintVersion(fields)
}
```  
## Web推送  
:::tip  
此工具包方法适用于[PWA](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps)。
:::  
### VAP ID Keys  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    privateKey, publicKey, err := utils.GenerateVAPIDKeys()
}
```  
### 发送推送通知  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    privateKey, publicKey, err := utils.GenerateVAPIDKeys()
    err := utils.SendNotification(privateKey, publicKey, subscription, subscribeEmail, payload)
}
```  
