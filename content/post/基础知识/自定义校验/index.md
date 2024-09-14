---
title: "自定义校验"
description: 
date: 2024-09-14T17:02:16+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
categories:
  - 基础知识
tags:
  - gin 
---

要为自定义验证规则提供自定义错误消息，你可以使用 `gin` 和 `go-playground/validator` 提供的功能来实现。你可以在自定义验证函数中返回详细的错误信息，或者在处理验证错误时生成自定义的错误消息。

以下是一个示例，展示如何为自定义验证规则提供自定义错误消息：

1. **安装依赖**

确保你已经安装了 Gin 和 `go-playground/validator` 库：

```sh
go get -u github.com/gin-gonic/gin
go get -u github.com/go-playground/validator/v10
```

2. **定义结构体和自定义验证器**

```go
package main

import (
    "github.com/gin-gonic/gin"
    "github.com/go-playground/validator/v10"
    "net/http"
    "regexp"
    "strings"
)

// 定义结构体
type Request struct {
    Numbers string `json:"numbers" binding:"required,commaseparated"`
}

var validate *validator.Validate

// 自定义验证器函数
func validateCommaSeparated(fl validator.FieldLevel) bool {
    value := fl.Field().String()
    if value == "" {
        return false
    }

    // 检查是否是逗号分隔的数字
    pattern := `^\d+(,\d+)*$`
    matched, _ := regexp.MatchString(pattern, value)
    return matched
}

func main() {
    r := gin.Default()

    validate = validator.New()

    // 注册自定义验证器
    validate.RegisterValidation("commaseparated", validateCommaSeparated)

    r.POST("/validate", func(c *gin.Context) {
        var req Request
        if err := c.ShouldBindJSON(&req); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }

        // 使用自定义验证器
        if err := validate.Struct(req); err != nil {
            // 生成自定义错误消息
            var errors []string
            for _, err := range err.(validator.ValidationErrors) {
                var errMsg string
                switch err.Tag() {
                case "required":
                    errMsg = "The field " + err.Field() + " is required."
                case "commaseparated":
                    errMsg = "The field " + err.Field() + " must be a comma-separated list of numbers."
                default:
                    errMsg = "The field " + err.Field() + " is invalid."
                }
                errors = append(errors, errMsg)
            }
            c.JSON(http.StatusBadRequest, gin.H{"errors": errors})
            return
        }

        c.JSON(http.StatusOK, gin.H{"message": "Success"})
    })

    r.Run(":8080")
}
```

在这个例子中：

- 自定义验证器函数 `validateCommaSeparated` 检查字符串是否匹配正则表达式 `^\d+(,\d+)*$`，确保字符串是逗号分隔的数字列表。
- 在 `main` 函数中，注册自定义验证器，并在处理请求时使用它。
- 如果验证失败，会生成自定义的错误消息并返回给客户端。

3. **测试**

运行这个程序，启动 Gin HTTP 服务器：

```sh
go run main.go
```

然后使用 `curl` 或 Postman 等工具向 `/validate` 路由发送 POST 请求来测试验证功能。例如：

```sh
curl -X POST -H "Content-Type: application/json" -d '{"numbers": "1,2,3"}' http://localhost:8080/validate
```

将会得到如下响应：

```json
{
    "message": "Success"
}
```

如果你发送一个不符合要求的字符串，例如：

```sh
curl -X POST -H "Content-Type: application/json" -d '{"numbers": "1,2,a"}' http://localhost:8080/validate
```

将会得到如下响应：

```json
{
    "errors": [
        "The field Numbers must be a comma-separated list of numbers."
    ]
}
```

这个响应显示了验证失败的自定义错误消息，表明字符串未能通过自定义验证器。通过这种方式，你可以为不同的验证规则提供具体的错误提示。