---
sidebar_position: 4
---

# 计划任务  
## 简介  
计划任务组件提供了极其简单的定时任务和定时任务操作功能。您也可以单独使用它。但是，如果你需要保证你的任务同时只有一种运行状态，那么你需要提前初始化redis连接。当然，您可以选择独立或集群。  
它们的源代码地址位于[github.com/keepchen/v3/schedule](https://github.com/keepchen/go-sail/tree/main/schedule)。  
例如，您可以使用`go get`命令将计划任务组件安装到本地，然后在代码文件的`import`指令处引入。  
```bash showLineNumbers  
go get -u github.com/keepchen/go-sail/v3/schedule
```  
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
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/lib/redis"
    "github.com/keepchen/go-sail/v3/schedule"
)

func main() {
    // highlight-start
    var conf = redis.Conf{}
    redis.InitRedis(conf)  
    // highlight-end
    ...
    otherFunc()
    ...
}

func otherFunc() {
    ch := make(chan, struct{})
    task := func() {
        fmt.Println("taskName...")
    }
    schedule.NewJob("taskName", task).
            // highlight-start
            WithoutOverlapping().
            // highlight-end
            EverySecond()
    ch <- struct{}{} //waiting...
}
```  