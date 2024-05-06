---
sidebar_position: 7
---  
# 邮件  
这个章节将介绍邮件如何使用。  
## 介绍  
邮件组件提供常用的邮件发送操作。它支持以多goroutine的方式的发送池，并且还支持结果回调。  
## 配置  
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
}
```  
## 使用方法  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/lib/email"
)

func main() {
    var conf email.Conf
    pool := email.NewPool(conf)
    pool.Emit()
    defer func() {
        pool.Done()
    }()

    cb := func(e *email.Envelope, err error) {
        if err != nil {
            log.Println("Send email error:", err, " | receiver:", e.To)
        } else {
            log.Println("Send mail success!", " | receiver:", e.To)
        }
    }

    envelopes := []*email.Envelope{
        {
            From:     conf.From,
            Subject:  conf.Subject,
            MimeType: mimeType,
            Body:     bodyStr,
            To:       to,
            Callback: cb,
        },
        {
            From:     conf.From,
            Subject:  conf.Subject,
            MimeType: mimeType,
            Body:     bodyStr,
            To:       to,
            Callback: cb,
        },
        {
            From:     conf.From,
            Subject:  conf.Subject,
            MimeType: mimeType,
            Body:     bodyStr,
            To:       to,
            Callback: cb,
        },
        ...
    }

    for index := range envelopes {
        pool.Mount(index, envelopes[index])
    }
}
```  

