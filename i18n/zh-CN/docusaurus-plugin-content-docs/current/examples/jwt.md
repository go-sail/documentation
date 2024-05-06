---
sidebar_position: 8
---  
# Jwt  
这个章节将介绍Jwt如何使用。  
## 简介  
Jwt组件封装了最常用的签名和验证令牌的方法。 Go-Sail启动时，会根据配置激活jwt配置项。  
## 配置  
```go title="github.com/keepchen/go-sail/lib/jwt/conf.go" showLineNumbers  
type Conf struct {
    Enable      bool   `yaml:"enable" toml:"enable" json:"enable"`                   //是否启用
    PublicKey   string `yaml:"public_key" toml:"public_key" json:"public_key"`       //公钥字符串或公钥文件地址
    PrivateKey  string `yaml:"private_key" toml:"private_key" json:"private_key"`    //私钥字符串或私钥文件地址
    Algorithm   string `yaml:"algorithm" toml:"algorithm" json:"algorithm"`          //加密算法: RS256 | RS512 | HS512
    HmacSecret  string `yaml:"hmac_secret" toml:"hmac_secret" json:"hmac_secret"`    //密钥
    TokenIssuer string `yaml:"token_issuer" toml:"token_issuer" json:"token_issuer"` //令牌颁发者
    privateKey  *rsa.PrivateKey
    publicKey   *rsa.PublicKey
}
```  
:::warning
需要注意的是，公钥和私钥只能配置其中之一，但需要与签名和解密配合使用。如果不配置公钥而使用验证令牌，会出现空指针异常。如果不配置私钥，而使用签名颁发令牌，会出现空指针异常。  
:::  

## 使用方法  
### 颁发令牌  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/lib/jwt"
)
var (
    conf = jwt.Conf{
        Algorithm:  string(jwt.SigningMethodRS512),
        PrivateKey: string(privateKey),
        PublicKey:  string(publicKey),
    }
    mapClaim = jwt.MapClaims{
        {
            "name":  "test",
            "aud":   "test-user",
            "exp":   time.Now().Add(time.Hour * 24).Unix(),
            "iat":   time.Now().Unix(),
            "iss":   "go-sail",
            "nbf":   time.Now().Add(-10 * time.Minute).Unix(),
            "sub":   "tester",
        },
    }
)

func main() {
    conf.Load() //this line is unnecessary after Go-Sail started, using *sailConfig.JwtConf instead.
    token, err := jwt.SignWithMap(mapClaim, conf)
}
```  
### 验证令牌
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/lib/jwt"
)
var (
    conf = jwt.Conf{
        Algorithm:  string(jwt.SigningMethodRS512),
        PrivateKey: string(privateKey),
        PublicKey:  string(publicKey),
    }
    token = "...."
)

func main() {
    conf.Load() //this line is unnecessary after Go-Sail started, using *sailConfig.JwtConf instead.
    claim, err := jwt.VerifyFromMap(token, conf)
}
```

