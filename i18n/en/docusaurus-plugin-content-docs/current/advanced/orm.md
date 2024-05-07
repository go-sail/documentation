---
sidebar_position: 5
---

# ORM  
This chapter introduces some paradigms related to implementing ORM based on the `gorm` class library.  
## Database model  
### Intorduction  
The database model defines the mapping relationship between database table fields and Go code, which is particularly important during database operations. Reasonable data model definition will provide powerful help for development and make business logic more reasonable and clear.  
### Definition  
First, we can abstract the public data table fields into a public structure. The code is as follows:  
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
Later, we can reference it in other data table models:  
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
Among them, we set up three hook functions for the `User` data model based on `gorm`, which are used to assign values to the creation time and update time fields.
:::  
## Database operations  
### Introduction  
Generally speaking, the database model is only used as a data table field definition. In the actual development process, it should not participate in specific business logic. In common development specifications, database operations are separated out separately for the sake of clear structure and subsequent maintainability. It is generally customary to call the stripped-out database operations **service**.  
Therefore, we plan this part of the database operation under the package named 'service'.    
### Definition  
The following is an example of operation of the `User` data model:  
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
### Usage  
After defining the database operation, you can directly initiate calls where you need to operate the `User` model, such as routing processing functions, scheduled tasks, etc.  
For example, we call the `User` model in the processing function:    
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
You may have noticed this line of code in the above code:  
```go  
loggerSvc = c.MustGet("logger").(*zap.Logger)
```  
Its function will be explained to you in detail in the next [Chapter](logtrace.md).  
:::  
You can view the above sample codes in [source code](https://github.com/keepchen/go-sail/tree/main/examples/pkg).  
