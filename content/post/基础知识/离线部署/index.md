---
title: "离线部署"
description: 
date: 2024-09-14T16:31:00+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
categories:
  - 基础知识
tags:
  - 基础知识 离线部署
---

如果你需要在离线环境中使用Go依赖包，你可以提前下载所有需要的依赖包，并将它们打包到项目中。以下是详细的步骤：

### 1. 在有网络的环境中下载依赖

首先，在有网络的环境中，初始化并下载项目的所有依赖包。

#### 初始化Go Modules

在项目根目录运行：

```sh
go mod init your_project
```

#### 下载依赖包

假设你需要使用GORM和达梦数据库驱动，可以使用以下命令下载依赖：

```sh
go get -u gorm.io/gorm
go get -u github.com/godoes/gorm-dameng
```

#### 清理和更新依赖

使用`go mod tidy`命令清理和更新依赖：

```sh
go mod tidy
```

### 2. 导出依赖包

使用`go mod vendor`命令将所有依赖包导出到`vendor`目录中：

```sh
go mod vendor
```

此命令会将所有依赖包下载到`vendor`目录，这样可以在离线环境中使用。

### 3. 打包项目

将项目和`vendor`目录打包以便在离线环境中使用。可以使用压缩工具（如`zip`或`tar`）打包：

```sh
zip -r your_project.zip your_project
# 或者
tar -czvf your_project.tar.gz your_project
```

### 4. 在离线环境中使用

在离线环境中，解压项目文件并确保使用`vendor`目录中的依赖包。解压缩之后，进入项目目录：

```sh
unzip your_project.zip
# 或者
tar -xzvf your_project.tar.gz
cd your_project
```

### 5. 确保使用`vendor`目录

在`go build`或`go run`时，Go会自动使用`vendor`目录中的依赖包。你可以使用以下命令构建和运行项目：

```sh
go build -o your_project
./your_project
```

### 示例项目结构

以下是示例项目结构：

```
your_project/
├── go.mod
├── go.sum
├── main.go
├── vendor/
│   └── 所有依赖包
└── pkg/
    └── example/
        └── example.go
```

### 代码示例

#### `go.mod`

```go
module your_project

go 1.20

require (
    gorm.io/gorm v1.23.8
    github.com/godoes/gorm-dameng v1.0.0
)
```

#### `main.go`

```go
package main

import (
    "encoding/json"
    "fmt"
    "gorm.io/gorm"
    "log"
    "os"

    "github.com/godoes/gorm-dameng"
)

type User struct {
    ID   uint   `gorm:"primaryKey"`
    Name string
    Age  int
}

func main() {
    dsn := os.Getenv("DM_DSN")
    if dsn == "" {
        log.Fatal("Environment variable DM_DSN is not set")
    }

    db, err := gorm.Open(dameng.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal("Failed to connect to database:", err)
    }

    db.AutoMigrate(&User{})

    user := User{Name: "Alice", Age: 25}
    db.Create(&user)

    var users []User
    db.Find(&users)
    for _, user := range users {
        fmt.Println(user)
    }

    db.Model(&user).Update("Age", 26)

    db.Delete(&user)

    var versionInfo []map[string]interface{}
    db.Raw("SELECT * FROM SYS.V$VERSION").Scan(&versionInfo)
    if err := db.Error; err == nil {
        versionBytes, _ := json.MarshalIndent(versionInfo, "", "  ")
        fmt.Printf("达梦数据库版本信息：\n%s\n", versionBytes)
    }
}
```

通过这些步骤，你可以在有网络的环境中下载并打包依赖包，然后在离线环境中使用这些包。这种方法确保你在离线环境中依然可以构建和运行Go项目。