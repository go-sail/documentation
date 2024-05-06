---
sidebar_position: 3
---  
# 数据库  
这个章节介绍数据库组件如何使用。  
## 简介  
数据库组件是`gorm.io/gorm`的二次封装。 该组件只封装了数据库的连接处理和日志处理，其余内容均为原生调用。
得益于gorm的特性，数据库操作支持mysql、postresql、sqlserver、sqlite和clickhouse。  
当Go-Sail启动时，如果启用，它将自动初始化数据库组件。之后开发者可以直接通过sail关键字来调用。    
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    // highlight-start
    dbr, dbw := sail.GetDB()
    dbr := sail.GetDBR()
    dbw := sail.GetDBW()
    // highlight-end
}
```  
## 配置  
上一章我们学习了数据库的详细配置，如下：  
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
下面我们来详细解释一下major字段的含义和作用。   
- Enable  
    指定是否启用数据库组件。  
- DriverName  
    指定数据库的驱动类型，目前支持以下类型：  
    - mysql  
    - postgres  
    - sqlite  
    - sqlserver  
    - clickhouse  
- AutoMigrate  
    指定是否自动同步表结构。 此配置供开发者自行决定使用。 当然开发者也可以忽略这个配置。  
- LogLevel  
    指定日志记录级别。该配置用于gorm内部日志打印，与Logger组件分开。Go-Sail使用`uber-go/zap`重写了`gorm`的日志记录行为。 
    该字段支持以下值：  
    - info  
    - warn  
    - error  
    - silent  

数据库组件根据读和写两种场景，将连接实例分为**读实例**和**写实例**。 这个方案在读写分离的场景下非常有用。如果您的业务不需要读写分离，您可以将读配置和写配置设置为相同的内容。  
## 使用方法  
### 读实例  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/sail"
)

func main() {
    dbr := sail.GetDBR()
}
```
### 写实例  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/sail"
)

func main() {
    dbr := sail.GetDBR()
}
```  
### 事务  
如果您要使用数据库事务，那么您应该确保事务中的操作应在**同一连接实例**上完成，以避免出现意外情况。  
:::tip  
这是一个正确的示例。  
```go title="main.go" showLineNumbers  
import (
    "gorm.io/gorm"
    "github.com/keepchen/go-sail/sail"
)

func main() {
    sail.GetDBW().Transaction(func(tx *gorm.DB) err error{
        err = tx.Model(...).Where(...).First(...).Error
        if err != nil {
            return err
        }
        err = tx.Model(...).Where(...).Updates(...).Error
        return err
    })
}
```  
:::  
:::danger  
这是一个错误的示例。  
```go title="main.go" showLineNumbers  
import (
    "gorm.io/gorm"
    "github.com/keepchen/go-sail/sail"
)

func main() {
    tx := sail.GetDBR().Begin()
    sail.GetDBW().Model(...).Where(...).Updates(...).Error
    tx := sail.GetDBR().Commit()
}
```  
:::  

