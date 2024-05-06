---
sidebar_position: 3
---

# 组件  
阐述什么是组件。  

### 术语  
在Go-Sail框架中，Component一般是第三方库的统称，比如数据库、redis、日志库等。  
当Go-Sail启动时，步骤之一是根据配置文件依次启动这些组件。  
这些组件将为后续业务功能开发提供极大的便利。 同时，你不需要关心它们底层的实现细节，一切都将由Go-Sail接管。  

### 获取  
组件初始化后，可以通过 `sail` 关键字获取对应的组件实例。  
例如：  
- 日志  
```go title="main.go" showLineNumbers  
sail.GetLogger()
```

- 数据库  
```go title="main.go" showLineNumbers  
dbr, dbw := sail.GetDB()

dbr := sail.GetDBR()

dbw := sail.GetDBW()
```  
:::tip  
`sail.GetXX` 是安全的，可以在整个业务生命周期（afterFunc）中使用。  
如果您对业务生命周期有疑问，请查看 **[Lifecycle](./lifecycle.md)** 章节。
:::
