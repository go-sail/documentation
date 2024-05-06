---
sidebar_position: 6
---

# Common toolkit 
Explain what Common toolkit is.  

### Introduction  
The common toolkit provides a relatively rich set of tool methods, which can provide developers with daily tool calls. 

### Features  
- **AES**  
- **Base64**  
- **CRC**  
- **Datetime**  
- **File**  
- **Heap**  
- **IP**  
- **MD5**  
- **Redis lock**  
- **RSA**  
- **Singal**  
- **SM4**  
- **String**  
- **Time**  
- **Validator**  
- **Version**  
- **Webpush**

### Usage  
```go title="main.go" showLineNumbers  
import (
  "github.com/keepchen/go-sail/v3/utils"
)

func main() {
  result, err := utils.AesEncode(rawStr, key)
  result := utils.MD5Encode(rawStr)
}
```  
:::tip   
We will introduce its purpose and usage in detail in the [subsequent chapters](../examples/toolkit.md).
:::  