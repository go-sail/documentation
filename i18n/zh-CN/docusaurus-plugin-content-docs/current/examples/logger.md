---
sidebar_position: 2
---  
# 日志  
这个章节将介绍日志库项如何使用。  

## 简介  
Logger组件是对`uber-go/zap`日志库的二次封装，丰富和增强了业务功能。  
当 Go-Sail启动时，如果启用，它将自动初始化日志组件。之后开发者就可以直接通过`sail`关键字来调用它。  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    // highlight-start
    logger := sail.GetLogger()
    // highlight-end
}
```  

## 配置  
上一章我们学习了日志库的详细配置，如下：  
```go title="github.com/keepchen/go-sail/lib/logger/conf.go" showLineNumbers  
type Conf struct {
    ConsoleOutput bool     `yaml:"console_output" toml:"console_output" json:"console_output" default:"false"` //是否同时输出到终端
    Env           string   `yaml:"env" toml:"env" json:"env" default:"prod"`                                   //日志环境，prod：生产环境，dev：开发环境
    Level         string   `yaml:"level" toml:"level" json:"level" default:"info"`                             //日志级别，debug，info，warn，error
    Modules       []string `yaml:"modules" toml:"modules" json:"modules"`                                      //模块名称（日志记录到不同的文件中）
    Filename      string   `yaml:"filename" toml:"filename" json:"filename" default:"logs/running.log"`        //日志文件名称
    MaxSize       int      `yaml:"max_size" toml:"max_size" json:"max_size" default:"100"`                     //日志大小限制，单位MB
    MaxBackups    int      `yaml:"max_backups" toml:"max_backups" json:"max_backups" default:"10"`             //最大历史文件保留数量
    Compress      bool     `yaml:"compress" toml:"compress" json:"compress" default:"true"`                    //是否压缩历史日志文件
    Exporter      Exporter `yaml:"exporter" toml:"exporter" json:"exporter"`                                   //导出器
}

type Exporter struct {
    Provider string        `yaml:"provider" toml:"provider" json:"provider" default:""` //导出器，目前支持redis、redis-cluster、nats和kafka，为空表示不启用
    Redis    ProviderRedis `yaml:"redis" toml:"redis" json:"redis"`
    Nats     ProviderNats  `yaml:"nats" toml:"nats" json:"nats"`
    Kafka    ProviderKafka `yaml:"kafka" toml:"kafka" json:"kafka"`
}

type ProviderRedis struct {
    ListKey         string            `yaml:"list_key" toml:"list_key" json:"list_key"`                            //redis列表名称
    ConnConf        redis.Conf        `yaml:"conn_conf" toml:"conn_conf" json:"conn_conf"`                         //redis连接配置（单机）
    ClusterConnConf redis.ClusterConf `yaml:"cluster_conn_conf" toml:"cluster_conn_conf" json:"cluster_conn_conf"` //redis连接配置（集群）
}

type ProviderNats struct {
    Subject  string    `yaml:"subject" toml:"subject" json:"subject"`       //nats的发布主题
    ConnConf nats.Conf `yaml:"conn_conf" toml:"conn_conf" json:"conn_conf"` //nats连接配置
}

type ProviderKafka struct {
    Topic    string     `yaml:"topic" toml:"topic" json:"topic"`             //kafka的发布主题
    ConnConf kafka.Conf `yaml:"conn_conf" toml:"conn_conf" json:"conn_conf"` //kafka连接配置
}
```  
现在让我们详细解释一下各个字段的含义和作用。  
- ConsoleOutput  
写入日志文件时是否将日志信息输出到终端。      
- Env  
指定运行环境。 目前尚未使用，可以忽略。  
- Level  
    指定打印日志级别，支持以下级别（从低到高）：  
    - debug  
    - info  
    - warn  
    - error  
    - dpanic  
    - panic  
    - fatal  
- Modules  
    指定日志文件模块，通俗地说，就是按照文件名来分割日志。 当日志较多且需要按功能划分时，此配置很有用。  

    它将与“文件名”字段结合使用。 假设“Filename”字段值为“running.log”，“Modules”的值之一为“schedule”，则schedule的文件名为“running_schedule.log”。  

    您可以通过`sail.GetLogger()`方法指定模块的日志输入，如：`sail.GetLogger("schedule")`。  
- Filename  
    指定日志文件名，可以包含路径。  
- MaxSize  
    指定的文件大小单位为MB。当文件大小达到设定值时，日志轮转将开始。轮转的日志将被压缩并打包。  
- MaxBackups  
    指定历史文件的保留数量。  
- Compress  
    是否压缩历史文件。  
- Exporter  
    设置导出器。  
## 导出器  
导出者的作用是将日志转移到其他地方。 一般来说，导出器应该在内部异步工作以提供最佳性能。  
这是可选的，如果不启用，日志只会输出到本地文件。  
### Redis列表(单实例)  
`Exporter.Provider`选项的值需要设置为`redis`。另外还需要设置相应的redis连接信息。  
### Redis单实例(集群)  
`Exporter.Provider`选项的值需要设置为`redis-cluster`。另外还需要设置相应的redis连接信息。  
### Nats主题  
`Exporter.Provider`选项的值需要设置为`nats`。另外还需要设置相应的nat连接信息。  
### Kafka主题  
`Exporter.Provider` 选项的值需要设置为 `kafka`。 另外还需要设置对应的kafka连接信息。  
### 其他  
上述导出器是 Go-Sail 的内置实现。 如果您想使用其他导出器，您可以自己实现“zapcore.WriteSyncer”。  
## 使用方法  
### 常规的  
```go title="main.go"  showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    sail.GetLogger().Error("looks like something went wrong", 
        zap.Errors("errors", []error{err}))
}
```  
### 指定模块  
```go title="main.go"  showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    sail.GetLogger("schedule").Error("looks like something went wrong", 
        zap.Errors("errors", []error{err}))
}
```  
### 序列化字段  
```go title="main.go"  showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/lib/logger"
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    sail.GetLogger("schedule").Error("looks like something went wrong", 
        zap.String("value", logger.MarshalInterfaceValue(anyValue)),
        zap.Errors("errors", []error{err}))
}
```  
### 其他  
更多原生调用方法请查看[uber-go/zap](https://github.com/uber-go/zap)的官方文档。  