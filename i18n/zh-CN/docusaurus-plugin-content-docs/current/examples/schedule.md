---
sidebar_position: 11
---  
# 计划任务  
这个章节将介绍计划任务如何使用。  
## 简介  
计划任务组件大致分为两类。一种是按照时间间隔执行。此类任务是使用本机“time.Ticker”实现的。另一种类型是使用Linux Crontab表达式执行的。该类底层封装了`robfig/cron`类库。计划任务为开发者提供了简单易用的语法糖。  
## 使用方法  
### 间隔性的  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/schedule"
)

func main() {
    ch := make(chan, struct{})
    task := func() {
        fmt.Println("taskName...")
    }
    schedule.NewJob("taskName", task).EverySecond()
    ch <- struct{}{} //waiting...
}
```  
### Linux Crontab风格的    
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/schedule"
)

func main() {
    ch := make(chan, struct{})
    task := func() {
        fmt.Println("taskName...")
    }
    schedule.NewJob("taskName", task).RunAt("* * * * *")
    ch <- struct{}{} //waiting...
}
```  
### 延迟性的 (一次性的)  
```go title="main.go" showLineNumbers  
import (
    "time"
    "github.com/keepchen/go-sail/v3/schedule"
)

func main() {
    ch := make(chan, struct{})
    task := func() {
        fmt.Println("taskName...")
    }
    schedule.NewJob("taskName", task).RunAfter(5*time.Second)
    ch <- struct{}{} //waiting...
}
```  
### 防止任务堆叠  
`WithoutOverlapping()`方法的作用是防止任务同时重复运行。当您的服务运行多个副本并且您希望确保任务的唯一性时，它非常有用。  
:::warning  
该功能需要提前初始化redis连接实例，无论是单机模式还是集群模式。
否则它会**panic**。
:::
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/schedule"
)

func main() {
    ch := make(chan, struct{})
    task := func() {
        fmt.Println("taskName...")
    }
    schedule.NewJob("taskName", task).WithoutOverlapping().EverySecond()
    ch <- struct{}{} //waiting...
}
```  
:::note  
需要注意的是，你的任务不应该完全是一个goroutine，否则会出现意想不到的情况，并且`WithoutOverlapping()`也会失败。  
:::  
这里给出了一个错误的示范：  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/schedule"
)

func main() {
    ch := make(chan, struct{})
    task := func() {
        // highlight-start
        go func() {
            ...
            fmt.Println("taskName...")
            ...
        }
        // highlight-end
    }
    schedule.NewJob("taskName", task).WithoutOverlapping().EverySecond()
    ch <- struct{}{} //waiting...
}
```  
### 取消  
尚未启动或未运行的任务将被直接取消。 正在运行的任务将等待其运行完成，然后不再启动。  
```go title="main.go" showLineNumbers  
import (
    "time"
    "github.com/keepchen/go-sail/v3/schedule"
)

func main() {
    ch := make(chan, struct{})
    task := func() {
        fmt.Println("taskName...")
    }
    cancel := schedule.NewJob("taskName", task).WithoutOverlapping().EverySecond()
    time.Sleep(5*time.Second)
    // highlight-start
    cancel()
    // highlight-end
    ch <- struct{}{} //waiting...
}
```  
### Crontab表达式   
```go title="github.com/keepchen/go-sail/schedule/crontabexpr.go"  
EveryMinute                           = "* * * * *"             //每分钟的开始第0秒
EveryFiveMinute                       = "*/5 * * * *"           //每5分钟的开始第0秒
EveryTenMinute                        = "*/10 * * * *"          //每10分钟的开始第0秒
EveryFifteenMinute                    = "*/15 * * * *"          //每15分钟的开始第0秒
EveryTwentyMinute                     = "*/20 * * * *"          //每20分钟的开始第0秒
EveryThirtyMinute                     = "*/30 * * * *"          //每30分钟的开始第0秒
EveryFortyFiveMinute                  = "*/45 * * * *"          //每45分钟的开始第0秒
FirstDayOfMonth                       = "0 0 1 * *"             //每月的第一天的0点0分
LastDayOfMonth                        = "0 0 L * *"             //每月的最后一天的0点0分
FirstDayOfWeek                        = "0 0 * * 1"             //每周的第一天（周一）的0点0分
LastDayOfWeek                         = "0 0 * * 7"             //每周的最后一天（周天）的0点0分
TenClockAtWeekday                     = "0 10 * * MON-FRI"      //每个工作日（周一~周五）的上午10点0分
TenClockAtWeekend                     = "0 10 * * SAT,SUN"      //每个周末（周六和周日）的上午10点0分
HourlyBetween9And17ClockAtWeekday     = "0 9-17 * * MON-FRI"    //每个工作日（周一~周五）的上午9点0分到下午5点0分每小时一次
HalfHourlyBetween9And17ClockAtWeekday = "*/30 9-17 * * MON-FRI" //每个工作日（周一~周五）的上午9点0分到下午5点0分每半时一次
```  
#### 使用方法
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/schedule"
)

func main() {
    ch := make(chan, struct{})
    task := func() {
        fmt.Println("taskName...")
    }
    schedule.NewJob("taskName", task).WithoutOverlapping().RunAt(schedule.EveryFiveMinute)
    ch <- struct{}{} //waiting...
}
```  
