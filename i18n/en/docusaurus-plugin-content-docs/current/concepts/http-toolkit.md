---
sidebar_position: 5
---

# Http toolkit 
Explain what Http toolkit is.  

### Introduction  
The Http toolkit provides unified response, routing middleware, and generic request and response entities.  

### Routing middleware  
Routing middleware provides some common functions:  
- **Detect user agent language**  
- **Log trace**  
- **Print request payload**  
- **Prometheus exporter**  
- **Cors (Cross-origin resource sharing)**  
:::tip   
We will introduce its purpose and usage in detail in the [subsequent chapters](../examples/http.md).
:::  

### Standard request and response entities  
- **dto**
    - base
    - pagination  
    - error
- **vo**  
    - pagination  
:::tip   
We will introduce its purpose and usage in detail in the [subsequent chapters](../examples/http.md).
:::  
### Unified response  
First, unified response makes the response data structure highly unified and complete. Secondly, some data values in the response entity are maintained by Go-Sail and do not require developers to care too much, which helps developers focus on other places.  
The general structure and format are as follows:  
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