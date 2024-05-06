---
sidebar_position: 1
---  
# 配置  
这个章节将介绍配置项如何使用。  

## 简介  
配置是启动整个Go-Sail服务的关键之一。它决定了Go-Sail的工作方式。  

## 各个部分  
```go title="github.com/keepchen/go-sail/sail/config/config.go" showLineNumbers  
type Config struct {
    HttpServer       HttpServerConf    `yaml:"http_conf" toml:"http_conf" json:"http_conf"`                            //http服务配置
    LoggerConf       logger.Conf       `yaml:"logger_conf" toml:"logger_conf" json:"logger_conf"`                      //日志配置
    DBConf           db.Conf           `yaml:"db_conf" toml:"db_conf" json:"db_conf"`                                  //数据库配置
    RedisConf        redis.Conf        `yaml:"redis_conf" toml:"redis_conf" json:"redis_conf"`                         //redis配置(standalone)
    RedisClusterConf redis.ClusterConf `yaml:"redis_cluster_conf" toml:"redis_cluster_conf" json:"redis_cluster_conf"` //redis配置(cluster)
    NatsConf         nats.Conf         `yaml:"nats_conf" toml:"nats_conf" json:"nats_conf"`                            //nats配置
    JwtConf          *jwt.Conf         `yaml:"jwt_conf" toml:"jwt_conf" json:"jwt_conf"`                               //jwt配置
    EmailConf        email.Conf        `yaml:"email_conf" toml:"email_conf" json:"email_conf"`                         //邮件配置
    KafkaConf        KafkaExtraConf    `yaml:"kafka_conf" toml:"kafka_conf" json:"kafka_conf"`                         //kafka配置
    EtcdConf         etcd.Conf         `yaml:"etcd_conf" toml:"etcd_conf" json:"etcd_conf"`                            //etcd配置
}

type KafkaExtraConf struct {
    Conf    kafka.Conf `yaml:"conf" toml:"conf" json:"conf"`          //配置
    Topic   string     `yaml:"topic" toml:"topic" json:"topic"`       //主题
    GroupID string     `yaml:"groupID" toml:"groupID" json:"groupID"` //分组id
}
```

### Http服务器  
```go title="github.com/keepchen/go-sail/sail/config/config.go" showLineNumbers  
type HttpServerConf struct {
    Debug              bool           `yaml:"debug" toml:"debug" json:"debug" default:"false"`                              //是否是debug模式
    Addr               string         `yaml:"addr" toml:"addr" json:"addr" default:":8080"`                                 //监听地址
    Swagger            SwaggerConf    `yaml:"swagger_conf" toml:"swagger_conf" json:"swagger_conf"`                         //swagger文档配置
    Prometheus         PrometheusConf `yaml:"prometheus_conf" toml:"prometheus_conf" json:"prometheus_conf"`                //prometheus配置
    WebSocketRoutePath string         `yaml:"websocket_route_path" toml:"websocket_route_path" json:"websocket_route_path"` //websocket路由
}

type SwaggerConf struct {
    Enable      bool   `yaml:"enable" toml:"enable" json:"enable" default:"false"`      //是否启用
    RedocUIPath string `yaml:"redoc_ui_path" toml:"redoc_ui_path" json:"redoc_ui_path"` //ui页面文件路径，如/path/to/docs.html，注意文件名必须是docs.html
    JsonPath    string `yaml:"json_path" toml:"json_path" json:"json_path"`             //json文件路径
    FaviconPath string `yaml:"favicon_path" toml:"favicon_path" json:"favicon_path"`    //浏览器页签图标文件路径
}

type PrometheusConf struct {
    Enable     bool   `yaml:"enable" toml:"enable" json:"enable" default:"false"`                   //是否启用
    Addr       string `yaml:"addr" toml:"addr" json:"addr" default:":19100"`                        //监听地址
    AccessPath string `yaml:"access_path" toml:"access_path" json:"access_path" default:"/metrics"` //路由地址
}
```  
:::warning  
需要注意的是，这部分配置不支持热更新。  
:::  

### 日志  
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
:::warning  
需要注意的是，这部分配置不支持热更新。  
:::  
### 数据库  
```go title="github.com/keepchen/go-sail/lib/db/conf.go" showLineNumbers  
type Conf struct {
    Enable         bool               `yaml:"enable" toml:"enable" json:"enable" default:"false"`                   //是否启用
    DriverName     string             `yaml:"driver_name" toml:"driver_name" json:"driver_name" default:"mysql"`    //数据库类型
    AutoMigrate    bool               `yaml:"auto_migrate" toml:"auto_migrate" json:"auto_migrate" default:"false"` //是否自动同步表结构
    LogLevel       string             `yaml:"log_level" toml:"log_level" json:"log_level" default:"info"`           //日志级别
    ConnectionPool ConnectionPoolConf `yaml:"connection_pool" toml:"connection_pool" json:"connection_pool"`        //连接池配置
    Mysql          MysqlConf          `yaml:"mysql" toml:"mysql" json:"mysql"`                                      //mysql配置
    Postgres       PostgresConf       `yaml:"postgres" toml:"postgres" json:"postgres"`                             //postgres配置
    Sqlserver      SqlserverConf      `yaml:"sqlserver" toml:"sqlserver" json:"sqlserver"`                          //sqlserver配置
    Sqlite         SqliteConf         `yaml:"sqlite" toml:"sqlite" json:"sqlite"`                                   //sqlite配置
    Clickhouse     ClickhouseConf     `yaml:"clickhouse" toml:"clickhouse" json:"clickhouse"`                       //clickhouse配置
}

// ConnectionPoolConf 连接池配置
type ConnectionPoolConf struct {
    MaxOpenConnCount       int `yaml:"max_open_conn_count" toml:"max_open_conn_count" json:"max_open_conn_count" default:"100"`                     //最大开启连接数
    MaxIdleConnCount       int `yaml:"max_idle_conn_count" toml:"max_idle_conn_count" json:"max_idle_conn_count" default:"10"`                      //最大闲置数量
    ConnMaxLifeTimeMinutes int `yaml:"conn_max_life_time_minutes" toml:"conn_max_life_time_minutes" json:"conn_max_life_time_minutes" default:"30"` //连接最大存活时间(分钟)
    ConnMaxIdleTimeMinutes int `yaml:"conn_max_idle_time_minutes" toml:"conn_max_idle_time_minutes" json:"conn_max_idle_time_minutes" default:"10"` //连接最大空闲时间(分钟)
}

type MysqlConf struct {
    Read  MysqlConfItem `yaml:"read" toml:"read" json:"read" default:"localhost"`    //读实例
    Write MysqlConfItem `yaml:"write" toml:"write" json:"write" default:"localhost"` //写实例
}

// MysqlConfItem mysql配置
type MysqlConfItem struct {
    Host      string `yaml:"host" toml:"host" json:"host" default:"localhost"`           //主机地址
    Port      int    `yaml:"port" toml:"port" json:"port" default:"3306"`                //端口
    Username  string `yaml:"username" toml:"username" json:"username"`                   //用户名
    Password  string `yaml:"password" toml:"password" json:"password"`                   //密码
    Database  string `yaml:"database" toml:"database" json:"database"`                   //数据库名
    Charset   string `yaml:"charset" toml:"charset" json:"charset"`                      //字符集
    ParseTime bool   `yaml:"parseTime" toml:"parseTime" json:"parseTime" default:"true"` //是否解析时间
    Loc       string `yaml:"loc" toml:"loc" json:"loc" default:"Local"`                  //位置
}

type PostgresConf struct {
    Read  PostgresConfItem `yaml:"read" toml:"read" json:"read" default:"localhost"`    //读实例
    Write PostgresConfItem `yaml:"write" toml:"write" json:"write" default:"localhost"` //写实例
}

// PostgresConfItem postgres配置
type PostgresConfItem struct {
    Host     string `yaml:"host" toml:"host" json:"host" default:"localhost"`                 //主机地址
    Port     int    `yaml:"port" toml:"port" json:"port" default:"9920"`                      //端口
    Username string `yaml:"username" toml:"username" json:"username"`                         //用户名
    Password string `yaml:"password" toml:"password" json:"password"`                         //密码
    Database string `yaml:"database" toml:"database" json:"database"`                         //数据库名
    SSLMode  string `yaml:"ssl_mode" toml:"ssl_mode" json:"ssl_mode"`                         //ssl模式 enable|disable
    TimeZone string `yaml:"timezone" toml:"timezone" json:"timezone" default:"Asia/Shanghai"` //时区
}

type SqlserverConf struct {
    Read  SqlserverConfItem `yaml:"read" toml:"read" json:"read" default:"localhost"`    //读实例
    Write SqlserverConfItem `yaml:"write" toml:"write" json:"write" default:"localhost"` //写实例
}

// SqlserverConfItem sqlserver配置
type SqlserverConfItem struct {
    Host     string `yaml:"host" toml:"host" json:"host" default:"localhost"` //主机地址
    Port     int    `yaml:"port" toml:"port" json:"port" default:"9930"`      //端口
    Username string `yaml:"username" toml:"username" json:"username"`         //用户名
    Password string `yaml:"password" toml:"password" json:"password"`         //密码
    Database string `yaml:"database" toml:"database" json:"database"`         //数据库名
}

type SqliteConf struct {
    Read  SqliteConfItem `yaml:"read" toml:"read" json:"read" default:"localhost"`    //读实例
    Write SqliteConfItem `yaml:"write" toml:"write" json:"write" default:"localhost"` //写实例
}

// SqliteConfItem sqlite配置
type SqliteConfItem struct {
    File string `yaml:"file" toml:"file" json:"file" default:"sqlite.db"` //数据库文件
}

type ClickhouseConf struct {
    Read  ClickhouseConfItem `yaml:"read" toml:"read" json:"read" default:"localhost"`    //读实例
    Write ClickhouseConfItem `yaml:"write" toml:"write" json:"write" default:"localhost"` //写实例
}

// ClickhouseConfItem clickhouse配置
type ClickhouseConfItem struct {
    Host         string `yaml:"host" toml:"host" json:"host" default:"localhost"`                     //主机地址
    Port         int    `yaml:"port" toml:"port" json:"port" default:"9000"`                          //端口
    Username     string `yaml:"username" toml:"username" json:"username"`                             //用户名
    Password     string `yaml:"password" toml:"password" json:"password"`                             //密码
    Database     string `yaml:"database" toml:"database" json:"database"`                             //数据库名
    ReadTimeout  int    `yaml:"read_timeout" toml:"read_timeout" json:"read_timeout" default:"20"`    //读取超时时间
    WriteTimeout int    `yaml:"write_timeout" toml:"write_timeout" json:"write_timeout" default:"20"` //写入超时时间
}
```  
:::warning  
需要注意的是，这部分配置不支持热更新。  
:::    
### Redis单实例模式  
```go title="github.com/keepchen/go-sail/lib/redis/conf.go" showLineNumbers  
type Conf struct {
    Endpoint  `yaml:"endpoint" toml:"endpoint" json:"endpoint"`
    Enable    bool `yaml:"enable" toml:"enable" json:"enable" default:"false"` //是否启用
    Database  int  `yaml:"database" toml:"database" json:"database"`           //数据库名
    SSLEnable bool `yaml:"ssl_enable" toml:"ssl_enable" json:"ssl_enable"`     //是否启用ssl
}

type Endpoint struct {
    Host     string `yaml:"host" toml:"host" json:"host" default:"localhost"` //主机地址
    Port     int    `yaml:"port" toml:"port" json:"port" default:"6379"`      //端口
    Username string `yaml:"username" toml:"username" json:"username"`         //用户名
    Password string `yaml:"password" toml:"password" json:"password"`         //密码
}
```  
:::warning  
需要注意的是，这部分配置不支持热更新。  
:::  
### Redis集群模式  
```go title="github.com/keepchen/go-sail/lib/redis/conf.go" showLineNumbers  
type ClusterConf struct {
    Enable    bool       `yaml:"enable" toml:"enable" json:"enable" default:"false"` //是否启用
    SSLEnable bool       `yaml:"ssl_enable" toml:"ssl_enable" json:"ssl_enable"`     //是否启用ssl
    Endpoints []Endpoint `yaml:"endpoints" toml:"endpoints" json:"endpoints"`        //连接地址列表
}

type Endpoint struct {
    Host     string `yaml:"host" toml:"host" json:"host" default:"localhost"` //主机地址
    Port     int    `yaml:"port" toml:"port" json:"port" default:"6379"`      //端口
    Username string `yaml:"username" toml:"username" json:"username"`         //用户名
    Password string `yaml:"password" toml:"password" json:"password"`         //密码
}
```  
:::warning  
需要注意的是，这部分配置不支持热更新。  
:::  
### Nats  
```go title="github.com/keepchen/go-sail/lib/nats/conf.go" showLineNumbers  
type Conf struct {
    Enable    bool     `yaml:"enable" toml:"enable" json:"enable" default:"false"` //是否启用
    Endpoints []string `yaml:"endpoints" toml:"endpoints" json:"endpoints"`        //服务实例列表
    Username  string   `yaml:"username" toml:"username" json:"username"`           //用户名
    Password  string   `yaml:"password" toml:"password" json:"password"`           //密码
}
```  
:::warning  
需要注意的是，这部分配置不支持热更新。  
:::  
### Jwt  
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
需要注意的是，这部分配置不支持热更新。  
:::  
### 邮件  
```go title="github.com/keepchen/go-sail/lib/email/conf.go" showLineNumbers  
type Conf struct {
    Workers               int    `yaml:"workers" toml:"workers" json:"workers"`                                                 //协程数量
    WorkerThrottleSeconds int    `yaml:"worker_throttle_seconds" toml:"worker_throttle_seconds" json:"worker_throttle_seconds"` //每个协程内发送间隔，单位秒
    Host                  string `yaml:"host" toml:"host" json:"host"`                                                          //邮件服务器域名
    Port                  int    `yaml:"port" toml:"port" json:"port"`                                                          //邮件服务器端口
    Username              string `yaml:"username" toml:"username" json:"username"`                                              //邮件服务登录账号
    Password              string `yaml:"password" toml:"password" json:"password"`                                              //邮件服务登录密码
    From                  string `yaml:"from" toml:"from" json:"from"`                                                          //发送人邮箱
    Subject               string `yaml:"subject" toml:"subject" json:"subject"`                                                 //邮件主题
    Params                struct {
        Variables []string `yaml:"variables" toml:"variables" json:"variables"` //替换变量
    } `yaml:"params" toml:"params" json:"params"` //其他参数
}
```  
:::warning  
需要注意的是，这部分配置不支持热更新。  
:::  
### Kafka  
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
:::warning  
需要注意的是，这部分配置不支持热更新。  
:::  
### Etcd  
```go title="github.com/keepchen/go-sail/lib/etcd/conf.go" showLineNumbers  
type Conf struct {
    Enable    bool        `yaml:"enable" toml:"enable" json:"enable" default:"false"` //是否启用
    Endpoints []string    `yaml:"endpoints" toml:"endpoints" json:"endpoints"`        //地址列表,如: localhost:2379
    Username  string      `yaml:"username" toml:"username" json:"username"`           //账号
    Password  string      `yaml:"password" toml:"password" json:"password"`           //密码
    Timeout   int         `yaml:"timeout" toml:"timeout" json:"timeout"`              //连接超时时间（毫秒）默认10000ms
    Tls       *tls.Config `yaml:"-" toml:"-" json:"-"`                                //tls配置
}
```  
:::warning  
需要注意的是，这部分配置不支持热更新。  
:::  

## 解析  
在config包中，Go-Sail为开发者提供了简单的解析方法，开发者可以选择性地使用。  
### 默认模板
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail/config"
)

func main() {
    //format: json
    config.PrintTemplateConfig("json", "path/to/config.json")
    //format: yaml
    config.PrintTemplateConfig("yaml", "path/to/config.yaml")
    //format: toml
    config.PrintTemplateConfig("toml", "path/to/config.toml")
}
```  
### 从源字符串解析  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail/config"
)

func main() {
    //json
    conf, err := config.ParseConfigFromBytes("json", sourceBytes)
    //yaml
    conf, err := config.ParseConfigFromBytes("yaml", sourceBytes)
    //toml
    conf, err := config.ParseConfigFromBytes("toml", sourceBytes)
}
```  
:::tip  
当您没有配置文件内容时，可以使用config包提供的打印配置信息模板方法到指定文件或终端，以便复制内容并进行快速配置。
:::  

### 从Nacos解析
如果您使用[Nacos](https://nacos.io)，您可以使用Go-Sail提供的便捷方法从Nacos服务中读取和监听配置信息。  

```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/lib/nacos"
    "github.com/keepchen/go-sail/v3/sail/config"
)

func main() {
    nacos.InitClient("appName", "nacos endpoints", "nacos namespace id")

    var conf = &config.Config{}

    //get config and parse config to go struct
    err = nacos.GetConfig("group", "dataID", conf, "yaml")

    callback := func(namespace, group, dataId, data string) {
        err := nacos.ParseConfig([]byte(data), conf, "yaml")
        if err != nil {
            fmt.Printf("<Nacos> listen config {%s:%s} change,but can't be unmarshal: %s\n", group, dataId, err.Error())
            return
        }
    }

    //listening config if it changed
    err = nacos.ListenConfigWithCallback(group, dataID, callback)
    if err != nil {
        panic(err)
    }
}
```  
## 嵌套组合  
Go-Sail的配置文件只会包含框架本身必要的配置项。 因此，在实际项目中，往往会伴随其他业务相关的配置项。 因此，我们需要对配置项进行组合或嵌套来满足实际需求。  
例如：  
### 具名组合
```go title="main.go" showLineNumbers  
import (
    sailConfig "github.com/keepchen/go-sail/v3/sail/config"
)

type GlobalConfigNamed struct {
    AppName  string            `yaml:"appName" json:"appName" toml:"appName"`
    Debug    bool              `yaml:"debug" json:"debug" toml:"debug"`
    // highlight-start
    SailConf sailConfig.Config `yaml:"sailConf" json:"sailConf" toml:"sailConf"` //tag is necessary
    // highlight-end
    ...
}
```  
### 匿名组合  
:::warning  
需要注意的是，匿名组合方法不能直接使用解析库将结构体解析配置到结构体中。
:::  
```go title="main.go" showLineNumbers  
import (
    sailConfig "github.com/keepchen/go-sail/v3/sail/config"
)

type GlobalConfigAnonymous struct {
    AppName  string            `yaml:"appName" json:"appName" toml:"appName"`
    Debug    bool              `yaml:"debug" json:"debug" toml:"debug"`
    // highlight-start
    sailConfig.Config
    // highlight-end
    ...
}
```  
例如这样的解析操作将不会生效：  
```go title="main.go" showLineNumbers  
import (
    "fmt"
    "encoding/json"
)

func main() {
    var conf GlobalConfigAnonymous
    err := json.Unmarshal(sourceBytes, &conf)

    //this code will output empty
    fmt.Println(conf.HttpServer.Addr)
}
```