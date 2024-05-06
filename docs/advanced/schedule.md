---
sidebar_position: 4
---

# Schedule  
## Introduction  
The Schedule component provides extremely simple timed tasks and scheduled task operation functions. You can also use it alone. However, if you need to ensure that your task has only one running state at the same time, then you need to initialize the redis connection in advance. Of course, you can choose either standalone or cluster.  
Their source code addresses are located at [github.com/keepchen/v3/schedule](https://github.com/keepchen/go-sail/tree/main/schedule).  
For example, you can use the `go get` command to install the schedule locally, and then introduce it at the `import` instruction of the code file.  
```bash showLineNumbers  
go get -u github.com/keepchen/go-sail/v3/schedule
```  
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