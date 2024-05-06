---
sidebar_position: 5
---

# Http工具包 
阐述Http工具包是什么.  

### 简介  
Http 工具包提供统一的响应、路由中间件以及通用的请求和响应实体。  

### 路由中间件  
路由中间件提供了一些常用的功能：  
- **解析客户端语言**  
- **日志追踪**  
- **打印请求载荷**  
- **Prometheus导出器**  
- **Cors (Cross-origin resource sharing)跨域资源共享**  
:::tip   
我们将在[后续章节](../examples/http.md)中详细介绍其目的和用法。
:::  

### 请求、响应实体规范  
- **dto**
    - base
    - pagination  
    - error
- **vo**  
    - pagination  
:::tip   
我们将在[后续章节](../examples/http.md)中详细介绍其目的和用法。
:::  
### 统一响应  
首先，统一响应使得响应数据结构高度统一和完整。 其次，响应实体中的一些数据值由Go-Sail维护，不需要开发人员过多关心，这有助于开发人员将精力集中在其他地方。  
一般结构和格式如下：  
```json showLineNumbers 
{
  "code": 200,
  "data": null,
  "message": "SUCCESS",
  "requestId": "5686efa5-c747-4f63-8657-e6052f8181a9",
  "success": true,
  "ts": 1670899688591
}
```  