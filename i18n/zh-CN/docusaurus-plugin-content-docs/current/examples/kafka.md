---
sidebar_position: 10
---  
# Kafka  
这一章节将介绍Kafka组件如何使用。  
## 简介  
Kafka组件对是对segmentio/kafka-go库的简单封装，允许开发者忽略连接细节。  
当Go-Sail启动时，如果启用，它将自动初始化Kafka组件。之后开发者就可以直接通过`sail`关键字来调用它。  
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
## 配置  
在上一章中，我们学习了Kafka的详细配置，如下：  
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
## 使用方法  
### 其他  
更多原生调用方法请查看[segmentio/kafka-go](https://github.com/segmentio/kafka-go)的官方文档。  