---
sidebar_position: 5
---

# ORM  
这个章节介绍基于`gorm`类库实现ORM相关的一些范式。  
## 数据库模型  
### 简介  
数据库模型定义了数据库表字段与Go代码的映射关系，在数据库操作的过程中尤为重要。合理的数据模型定义将为开发提供强有力的帮助，也会让业务逻辑变得更加合理清晰。  
### 定义  
首先，我们可以将公共的数据表字段抽象成一个公共的结构，代码如下：  
```go title="examples/pkg/common/db/models/base.go" showLineNumbers  
package models

// BaseModel 基础模型字段
type BaseModel struct {
    ID        uint64     `gorm:"column:id;type:bigint;primary_key;AUTO_INCREMENT;comment:主键ID"` //主键id
    CreatedAt time.Time  `gorm:"column:created_at;type:datetime;comment:创建时间"`                  //创建时间
    UpdatedAt time.Time  `gorm:"column:updated_at;type:datetime;comment:更新时间"`                  //更新时间
    DeletedAt *time.Time `gorm:"column:deleted_at;type:datetime;comment:(软)删除时间"`               //(软)删除时间
}
```  
随后，我们可以在其他的数据表模型中引用它：  
```go title="examples/pkg/common/db/models/user.go" showLineNumbers  
package models

import (
    "github.com/keepchen/go-sail/v3/utils"
    "gorm.io/gorm"
)

// User 用户表
type User struct {
    // highlight-start
    BaseModel
    // highlight-end
    UserID         string `gorm:"column:user_id;type:varchar(10);not null;unique:user_unique;comment:用户id"`
    Nickname       string `gorm:"column:nickname;type:varchar(50);comment:用户昵称"`
    Avatar         string `gorm:"column:avatar;type:varchar(1024);comment:用户头像"`
}

func (*User) TableName() string {
    return "users"
}

func (o *User) BeforeCreate(_ *gorm.DB) (err error) {
    o.BaseModel.CreatedAt = utils.NewTimeWithTimeZone(...).Now()
    o.BaseModel.UpdatedAt = o.BaseModel.CreatedAt

    return nil
}

func (o *User) BeforeSave(_ *gorm.DB) (err error) {
    o.BaseModel.CreatedAt = utils.NewTimeWithTimeZone(...).Now()
    o.BaseModel.UpdatedAt = o.BaseModel.CreatedAt

    return nil
}

func (o *User) BeforeUpdate(_ *gorm.DB) (err error) {
    o.BaseModel.UpdatedAt = utils.NewTimeWithTimeZone(...).Now()

    return nil
}
```  
:::tip  
其中，我们基于`gorm`，为`User`数据模型设置了三个勾子函数，用于创建时间和更新时间两个字段的赋值。
:::  
## 数据库操作  
### 简介  
一般来讲，数据库模型仅作为数据表字段定义使用，在实际的开发过程中，它不应该参与具体的业务逻辑。在常见的开发规范中，为了结构清晰和后续的可维护性，会把数据库操作单独剥离出来。一般习惯于把剥离出来的数据库操作称为**service**。  
因此，我们将数据库操作的这部分内容规划到名为'service'的包下面。  
### 定义  
下面给出一个关于`User`数据模型的操作范例：  
```go title="examples/pkg/common/db/service/user.go" showLineNumbers  
package service

import (
    "github.com/keepchen/go-sail/v3/lib/logger"
    "go.uber.org/zap"
    "gorm.io/gorm"
    "examples/pkg/common/db/models"
)

type UserSvc interface {
    //GetUser 获取用户信息
    GetUser(userID string) (models.User, error)
    //CreateUser 创建用户
    CreateUser(user models.User) error
    //UpsertUser 当用户不存在时创建用户存在则更新
    UpsertUser(user models.User) error
    //UpdateUser 更新用户信息
    UpdateUser(user models.User) error
}

type UserSvcImpl struct {
    dbr    *gorm.DB //读实例
    dbw    *gorm.DB //写实例
    logger *zap.Logger
}

var _ UserSvc = &UserSvcImpl{}

// NewUserSvcImpl 实例化服务
var NewUserSvcImpl = func(dbr *gorm.DB, dbw *gorm.DB, logger *zap.Logger) UserSvc {
    return &UserSvcImpl{
        dbr:    dbr,
        dbw:    dbw,
        logger: logger,
    }
}

func (u UserSvcImpl) CreateUser(user marketModels.User) error {
    err := u.dbw.Create(&user).Error
    if err != nil {
        u.logger.Error("数据库操作:CreateUser:错误",
            zap.Any("value", logger.MarshalInterfaceValue(user)), zap.Errors("errors", []error{err}))
    }

    return err
}

func (u UserSvcImpl) UpsertUser(user marketModels.User) error {
    exist, _ := u.GetUser(user.UserID)
    if exist.ID > 0 {
        err := u.dbw.Where(&marketModels.User{UserID: exist.UserID}).Updates(&user).Error
        if err != nil {
            u.logger.Error("数据库操作:UpsertUser:错误",
                zap.Any("value", logger.MarshalInterfaceValue(user)), zap.Errors("errors", []error{err}))
        }
        return err
    }

    return u.CreateUser(user)
}

func (u UserSvcImpl) UpdateUser(user marketModels.User) error {
    err := u.dbw.Where(&marketModels.User{UserID: user.UserID}).Updates(&user).Error
    if err != nil {
        u.logger.Error("数据库操作:UpdateUser:错误",
            zap.Any("value", logger.MarshalInterfaceValue(user)), zap.Errors("errors", []error{err}))
    }

    return err
}

func (u UserSvcImpl) GetUser(userID string) (marketModels.User, error) {
    var user marketModels.User
    err := u.dbw.Where(&marketModels.User{UserID: userID}).First(&user).Error
    if err != nil {
        u.logger.Error("数据库操作:GetUser:错误",
            zap.Any("value", logger.MarshalInterfaceValue(user)), zap.Errors("errors", []error{err}))
    }

    return user, err
}
```  
### 使用  
在定义好了数据库操作后，你可以在需要对`User`模型操作的地方直接发起调用，比如在路由处理函数、计划任务等这些地方。  
比如我们在处理函数中对`User`模型进行调用：  
```go title="examples/pkg/app/user/handler/userinfo.go" showLineNumbers  
package handler

import (
    "errors"

    "github.com/gin-gonic/gin"
    "github.com/keepchen/go-sail/v3/constants"
    "github.com/keepchen/go-sail/v3/sail"
    "github.com/keepchen/go-sail/v3/examples/pkg/app/user/http/vo/request"
    "github.com/keepchen/go-sail/v3/examples/pkg/app/user/http/vo/response"
    userSvc "examples/pkg/common/db/service/user"
    "github.com/keepchen/go-sail/v3/lib/logger"
    "go.uber.org/zap"
    "gorm.io/gorm"
)

func GetUserInfoSvc(c *gin.Context) {
    var (
        ...
        loggerSvc = c.MustGet("logger").(*zap.Logger)
        ...
    )
    if err := c.ShouldBind(&form); err != nil {
        sail.Response(c).Assemble(constants.ErrRequestParamsInvalid, nil).Send()
        return
    }

    if errorCode, err := form.Validator(); err != nil {
        sail.Response(c).Assemble(errorCode, nil, err.Error()).Send()
        return
    }

    // highlight-start
    user, sqlErr := userSvc.NewUserSvcImpl(sail.GetDBR(), sail.GetDBW(), loggerSvc).GetUser(form.UserID)
    // highlight-end
    if sqlErr != nil && errors.Is(sqlErr, gorm.ErrRecordNotFound) {
        sail.Response(c).Assemble(constants.ErrRequestParamsInvalid, nil, "user not found").Send()
        return
    }

    ...

    sail.Response(c).Assemble(constants.ErrNone, resp).Send()
}
```  
:::tip  
你可能注意到了在上面的这段代码中有这样一行代码：  
```go  
loggerSvc = c.MustGet("logger").(*zap.Logger)
```  
它的作用我们在接下来的[章节](logtrace.md)为你详解。
:::  
上面这些示例代码你可以在[源码](https://github.com/keepchen/go-sail/tree/main/examples/pkg)中查看它们。
