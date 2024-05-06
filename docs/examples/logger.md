---
sidebar_position: 2
---  
# Logger  
This chapter contains logger usage examples.  

## Introduction  
The Logger component is a secondary encapsulation of the `uber-go/zap` log library, which enriches and enhances business functions.  
When Go-Sail starts, it will automatically initialize the Logger component if enabled. After that, developers can call it directly through the `sail` keyword.  
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

## Configuration  
In the previous chapter, we learned about the detailed configuration of Logger, which is as follows:  
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
Now let us explain in detail the meaning and function of each field.  
- ConsoleOutput  
    Whether to output log information to the terminal while writing the log file.  
- Env  
    Specify the operating environment. It is not currently used and can be ignored.  
- Level  
    Specify the printing log level, the following levels are supported (from low to high):  
    - debug  
    - info  
    - warn  
    - error  
    - dpanic  
    - panic  
    - fatal  
- Modules  
    Specifying the log file module, in layman's terms, means splitting the logs by file name. This configuration is useful when there are many logs and need to be divided by function.  

    It will be used in conjunction with the `Filename` field. Assume that the `Filename` field value is `running.log` and one of the values ​​of `Modules` is `schedule`, then the file name of schedule is `running_schedule.log`.  

    You can specify the log input to the module through the `sail.GetLogger()` method, such as: `sail.GetLogger("schedule")`.  
- Filename  
    Specify the log file name, which can include the file path.  
- MaxSize  
    The specified file size unit is MB. When the file size reaches the set value, log rotation will start. The rotated logs will be compressed and packaged.  
- MaxBackups  
    Specify the number of historical files to keep.  
- Compress  
    Whether to compress history files.  
- Exporter  
    Specify the exporter.  
## Exporter  
The role of the exporter is to transfer the logs to other places. Generally speaking, exporters should work asynchronously internally to provide the best performance.  
This is optional, if not enabled, the log will only be output to a local file.  
### Redis list(standalone)  
The value of the `Exporter.Provider` option needs to be set to `redis`. In addition, the corresponding redis connection information needs to be set.  
### Redis list(cluster)  
The value of the `Exporter.Provider` option needs to be set to `redis-cluster`. In addition, the corresponding redis connection information needs to be set.  
### Nats subject  
The value of the `Exporter.Provider` option needs to be set to `nats`. In addition, the corresponding nats connection information needs to be set.  
### Kafka topic  
The value of the `Exporter.Provider` option needs to be set to `kafka`. In addition, the corresponding kafka connection information needs to be set.  
### Others  
The above exporters are built-in implementations of Go-Sail. If you want to use other exporters, you can implement `zapcore.WriteSyncer` yourself.  
## Usage  
### Common  
```go title="main.go"  showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    sail.GetLogger().Error("looks like something went wrong", 
        zap.Errors("errors", []error{err}))
}
```  
### Specify module  
```go title="main.go"  showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    sail.GetLogger("schedule").Error("looks like something went wrong", 
        zap.Errors("errors", []error{err}))
}
```  
### Serialized fields  
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
### Others  
For more native calling methods, please view the official documentation of [uber-go/zap](https://github.com/uber-go/zap).  