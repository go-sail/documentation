---
sidebar_position: 13
---  
# Toolkit  
This chapter contains toolkit usage examples.  
## Introduction  
Go-Sail provides a toolkit that contains some commonly used tool functions.  
## AES  
### Encode  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    encoded, err := utils.AesEncode(rawStr, key)
}
```  
### Decode  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    decoded, err := utils.AesDecode(encoded, key)
}
```  
## Base64  
### Encode  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    encoded, err := utils.Base64Encode(rawBytes)
}
```  
### Decode  
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
### Format Date
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
### Parse Date
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
## File  
### Save to destination 
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
### Get contents
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.FileGetContents("path/to/filename")
}
```  
### Put contents
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    err := utils.FilePutContents(content, "path/to/filename")
}
```  
### Append contents
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    err := utils.FileAppendContents(content, "path/to/filename")
}
```  
### Assert exist
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    ok := utils.FileExists("path/to/filename")

    ok, err := utils.FileExistsWithError("path/to/filename")
}
```  
### Get extension
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    ext := utils.FileExt("path/to/filename")
}
```  
### Read line by line  
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
### Get local ip  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    localIp, err := utils.GetLocalIP()
}
```  
## MD5  
### Encode
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    encoded := utils.MD5Encode(rawStr)
}
```  
## RedisLock  
:::tip  
The redis lock will be automatically renewed internally, and developers do not need to care about internal details.  
:::  
:::warning  
You must first initialize the connection using `redis.InitRedis` or `redis.InitRedisCluster`.  
:::  
### Try lock  
> Non-blocking  

```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    ok := utils.RedisTryLock(key)
}
```  
### Acquire lock  
> Blocking  

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
### Encrypt  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.RSAEncrypt(rawString, publicKey)
}
```  
### Decrypt    
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.RSADecrypt(rawString, publicKey)
}
```  
## Signal  
### Listen exist signal  
> Blocking  

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
### Encrypt  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.SM4ECBEncrypt(hexKey, rawStr)
}
```  
### Decrypt  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result, err := utils.SM4ECBDecrypt(hexKey, rawStr)
}
```  
## Strings  
### Wrap words  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.Wordwrap(rawStr, 64, "\n")
}
```  
### Wrap redis key  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.WrapRedisKey(appName, key)
}
```  
### Random letters  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.RandomLetters(length)
}
```  
### Random digital chars  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.RandomDigitalChars(length)
}
```  
### Random complex string  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.RandomComplexString(length)
}
```  
### Reverse  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.StringReverse(rawStr)
}
```  
### Shuffle  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.StringShuffle(rawStr)
}
```  
### Padding (left)  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.StringPaddingLeft(rawStr, padChar, length)
}
```  
### Padding (right)  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.StringPaddingRight(rawStr, padChar, length)
}
```  
### Padding (both)  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.StringPaddingBoth(rawStr, padChar, length)
}
```  
## Swagger  
### Print summary info comment  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    result := utils.PrintSwaggerSummaryInfo(param)
}
```  
### Print controller (handler) comment  
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
## Validator  
### Email  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    ok := utils.ValidateEmail(email)
}
```  
### Identity Card  
> P.R. China  

```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    ok := utils.ValidateIdentityCard(idCard)
}
```  
## Version  
### Print Software version  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    utils.PrintVersion(fields)
}
```  
## Webpush  
:::tip  
This toolkit approach is for [PWA](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps).
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
### Send Notification  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/utils"
)

func main() {
    privateKey, publicKey, err := utils.GenerateVAPIDKeys()
    err := utils.SendNotification(privateKey, publicKey, subscription, subscribeEmail, payload)
}
```  
