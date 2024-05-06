---
sidebar_position: 10
---  
# Kafka  
This chapter contains kafka usage examples.  
## Introduction  
The Kafka component pair is a simple encapsulation of the `segmentio/kafka-go` library, allowing developers to ignore connection details.  
When Go-Sail starts, it will automatically initialize the Kafka component if enabled. After that, developers can call it directly through the `sail` keyword.  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    // highlight-start
    connections := sail.GetKafkaConnections()
    // highlight-end

    // highlight-start
    instance := sail.GetKafkaInstance()
    // highlight-end

    // highlight-start
    reader := sail.GetKafkaReader()
    // highlight-end

    // highlight-start
    writer := sail.GetKafkaWriter()
    // highlight-end
}
```  
## Configuration  
In the previous chapter, we learned about the detailed configuration of Kafka, which is as follows:  
```go title="github.com/keepchen/go-sail/lib/kafka/conf.go" showLineNumbers  
type Conf struct {
    Enable       bool        `yaml:"enable" toml:"enable" json:"enable" default:"false"`   //是否启用
    Endpoints    []string    `yaml:"endpoints" toml:"endpoints" json:"endpoints"`          //地址列表,如: localhost:9092
    SASLAuthType string      `yaml:"SASLAuthType" toml:"SASLAuthType" json:"SASLAuthType"` //认证加密方式：plain、sha256、sha512
    Username     string      `yaml:"username" toml:"username" json:"username"`             //账号
    Password     string      `yaml:"password" toml:"password" json:"password"`             //密码
    Timeout      int         `yaml:"timeout" toml:"timeout" json:"timeout"`                //连接超时时间（毫秒）默认10000ms
    Tls          *tls.Config `yaml:"-" toml:"-" json:"-"`                                  //tls配置//tls配置
}
```  
## Usage  
### Others  
For more native calling methods, please view the official documentation of [segmentio/kafka-go](https://github.com/segmentio/kafka-go).  

