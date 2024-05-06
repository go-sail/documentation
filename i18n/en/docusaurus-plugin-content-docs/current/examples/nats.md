---
sidebar_position: 6
---  
# Nats  
This chapter contains nats usage examples.  
## Introduction  
The nats component is a secondary encapsulation of `nats-io/nats.go`.  
When Go-Sail starts, it will automatically initialize the Nats component if enabled. After that, developers can call it directly through the `sail` keyword.  
```go title="main.go" showLineNumbers  
import (
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    // highlight-start
    instance := sail.GetNats()
    // highlight-end
}
```  
## Configuration  
In the previous chapter, we learned about the detailed configuration of Redis (standalone and cluster), which is as follows:  
```go title="github.com/keepchen/go-sail/lib/nats/conf.go" showLineNumbers  
type Conf struct {
    Enable    bool     `yaml:"enable" toml:"enable" json:"enable" default:"false"` //是否启用
    Endpoints []string `yaml:"endpoints" toml:"endpoints" json:"endpoints"`        //服务实例列表
    Username  string   `yaml:"username" toml:"username" json:"username"`           //用户名
    Password  string   `yaml:"password" toml:"password" json:"password"`           //密码
}
```  
## Usage  
### Publish  
```go title="main.go" showLineNumbers  
import (
    natsLib "github.com/nats-io/nats.go"
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    stream, err := sail.GetNats().JetStream(natsLib.PublishAsyncMaxPending(256))

    if err != nil {
        panic(fmt.Errorf("initial nats failed: %v", err))
    }

    natsConfig := &natsLib.StreamConfig{
        Name:      "streamName",
        Subjects:  []string{"subject.>"},
        Retention: natsLib.WorkQueuePolicy,
        Discard:   natsLib.DiscardOld,
        Storage:   natsLib.FileStorage,
        Replicas:  3,
    }

    info, err := stream.AddStream(natsConfig)

    subject := fmt.Sprintf("subject.%d", 0)
    if err != nil {
        log.Println("[STREAM] add stream error:", err.Error(), info)
    } else {
        // highlight-start
        pubAck, pushErr := stream.Publish(subject, []byte(`{}`))
        // highlight-end
        log.Printf("[STREAM] test publish ACK: %d, error: %v\n", pubAck.Sequence, pushErr)
    }
}
```  
### Subscribe  
```go title="main.go" showLineNumbers  
import (
    natsLib "github.com/nats-io/nats.go"
    "github.com/keepchen/go-sail/v3/sail"
)

func main() {
    stream, err := sail.GetNats().JetStream(natsLib.PublishAsyncMaxPending(256))

    if err != nil {
        panic(fmt.Errorf("initial nats failed: %v", err))
    }

    natsConfig := &natsLib.StreamConfig{
        Name:      "streamName",
        Subjects:  []string{"subject.>"},
        Retention: natsLib.WorkQueuePolicy,
        Discard:   natsLib.DiscardOld,
        Storage:   natsLib.FileStorage,
        Replicas:  3,
    }

    info, err := stream.AddStream(natsConfig)

    // highlight-start
    cc := &natsLib.ConsumerConfig{
        Durable: "consumerName", 
        AckPolicy: natsLib.AckExplicitPolicy, 
        FilterSubject: "",
    }
    _, err := stream.AddConsumer("streamName", cc)
    // highlight-end

    // highlight-start
    sub, err := stream.PullSubscribe("filterSubject", "consumerName", 
                                natsLib.Bind("streamName", "consumerName"))
    // highlight-end
    if err != nil {
        panic(err)
    }

    for {
        msgs, _ := sub.Fetch(1, natsLib.MaxWait(5*time.Second))
        for _, msg := range msgs {
            ...
            _ = msg.AckSync()
            ...
        }
    }
}
```  

### Others  
For more native calling methods, please view the official documentation of [nats-io/nats.go](https://github.com/nats-io/nats.go).  


