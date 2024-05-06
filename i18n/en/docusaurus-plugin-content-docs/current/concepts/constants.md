---
sidebar_position: 4
---

# Constants  
Explain what Constants is.  
### Introduction  
In Go-Sail, some system constants are defined and stored under the contents package for use in some specific scenarios.  
- **code**  
It is used to define business response codes or error codes.  
    :::tip  
    The `RegisterCode()` function will provide powerful help in injecting custom error codes and error messages.  
    We will introduce its purpose and usage in detail in the subsequent chapters.
    :::  
- **error**  
It is used to define error codes and error messages. It also supports i18n.  
- **i18n**  
It is used to define language encoding and adopts the ISO-3166-1 standard.  
- **keys**  
Currently used to define the prefix and suffix identifiers of public and private keys.  
- **sail**  
Contains the framework’s logo characters and version number.  
- **status**  
Boolean value used for success and failure in response.  
- **time**  
Currently, time zone characters and date printing templates for various time zones around the world are defined.