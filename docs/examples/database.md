---
sidebar_position: 3
---  
# Database  
This chapter contains database usage examples.  
## Introduction  
The database component is a secondary encapsulation of `gorm.io/gorm`. This component only encapsulates the connection processing and log processing of the database, and the rest of the contents are native calls.  
Thanks to the features of gorm, database operations support mysql, postresql, sqlserver, sqlite and clickhouse.  
When Go-Sail starts, it will automatically initialize the Database component if enabled. After that, developers can call it directly through the sail keyword.  
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
## Configuration  
In the previous chapter, we learned about the detailed configuration of Database, which is as follows:  
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
Now let us explain in detail the meaning and function of major field.   
- Enable  
    Specify whether to enable the database component.  
- DriverName  
    Specify the database driver type. Currently, the following values are accepted:  
    - mysql  
    - postgres  
    - sqlite  
    - sqlserver  
    - clickhouse  
- AutoMigrate  
    Specify whether to automatically synchronize the table structure. This configuration is for developers to use at their own discretion. Of course, developers can also ignore this configuration.  
- LogLevel  
    Specify the logging level. This configuration is used for gorm internal log printing and is separate from the Logger component. Go-Sail uses `uber-go/zap` to rewrite the logging behavior of `gorm`. This field supports the following values:  
    - info  
    - warn  
    - error  
    - silent  

The database component divides the connection instance into a **read instance** and a **write instance** according to the two scenarios of reading and writing. This solution is very useful in the scenario of separation of reading and writing. If your business does not require separation of reading and writing, you can set the read configuration and write configuration to the same content.  
## Usage  
### Read instance  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/sail"
)

func main() {
    dbr := sail.GetDBR()
}
```
### Write instance  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/sail"
)

func main() {
    dbr := sail.GetDBR()
}
```  
### Transaction  
If you are going to use database transactions, then you should ensure that operations within your transaction should be completed on the **same connection instance** to avoid unexpected situations.  
:::tip  
This is a correct example.  
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
This is a bad example.  
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

