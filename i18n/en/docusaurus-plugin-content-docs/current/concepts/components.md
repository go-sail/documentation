---
sidebar_position: 3
---

# Components  
Explain what Components is.  

### Terminology  
In the Go-Sail framework, Component is generally the collective name for third-party libraries, such as databases, redis, log libraries, etc.  
When Go-Sail starts, one of the steps is to start these components in sequence according to the configuration file.  
These components will provide great convenience for subsequent business function development. At the same time, you do not need to care about their underlying implementation details, everything will be taken over by Go-Sail.  

### Obtain  
After the component is initialized, you can obtain the corresponding component instance through the `sail` keyword.  
For example:  
- logger  
```go title="main.go" showLineNumbers  
sail.GetLogger()
```

- db  
```go title="main.go" showLineNumbers  
dbr, dbw := sail.GetDB()

dbr := sail.GetDBR()

dbw := sail.GetDBW()
```  
:::tip  
`sail.GetXX` is safe and can be used during the entire business life cycle (afterFunc).  
If you have questions about the business lifecycle, please review the **[Lifecycle](./lifecycle.md)** chapter.
:::
