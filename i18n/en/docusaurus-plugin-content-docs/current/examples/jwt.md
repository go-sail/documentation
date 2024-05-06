---
sidebar_position: 8
---  
# Jwt  
This chapter contains jwt usage examples.  
## Introduction  
The jwt component encapsulates the most commonly used methods of signing and verifying tokens. When Go-Sail starts, the jwt configuration items will be activated according to the configuration.  
## Configuration  
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
It should be noted that only one of the public key and the private key can be configured, but it needs to be used in conjunction with signing and decryption. If the public key is not configured but the verification token method is used, a null pointer exception will occur. If the private key is not configured but the signing and issuing token method is used, a null pointer exception will occur.  
:::  

## Usage  
### Issue token  
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
### Verify token
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

