---
sidebar_position: 11
---  
# Schedule  
This chapter contains schedule usage examples.  
## Introduction  
Schedule components are roughly divided into two categories. One type is executed according to time intervals. This type of tasks is implemented using the native `time.Ticker`. The other type is executed using Linux Crontab expressions. This type of bottom layer encapsulates `robfig/cron` Class library. Scheduled tasks provide developers with easy-to-use syntax sugar.  
## Usage  
### Interval  
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
### Linux Crontab Style    
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
### Delay (One-time)  
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
### Without Overlapping  
The function of the `WithoutOverlapping()` method is to prevent tasks from running repeatedly at the same time. It is useful when your service runs multiple copies and you want to ensure the uniqueness of the task.  
:::warning  
This feature requires initializing the redis connection instance in advance, both in standalone mode and cluster mode.  
Otherwise it will **panic**.
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
It should be noted that your task should not be entirely a goroutine, otherwise unexpected situations will occur and `WithoutOverlapping()` will also fail.  
:::  
Here is an example of an error:  
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
### Cancel  
Tasks that have not been started or are not running will be canceled directly. A running task will wait for its run to complete and then not start again.  
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
### Crontab Expressions   
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
#### usage
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
