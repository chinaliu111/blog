---
title: "反向代理"
description: 
date: 2024-10-08T16:22:29+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
categories:
  - 第三方库
tags:
  - Example Tag
---

在 Go 中，`httputil.ReverseProxy` 是一个强大的工具，用于实现反向代理服务器，它能够将客户端请求转发到另一个服务器，并将响应返回给客户端。反向代理通常用于负载均衡、API 网关、缓存等场景。

以下是一个完整的 `httputil.ReverseProxy` 示例，它展示了如何创建一个简单的反向代理服务器。

### 示例说明

假设我们想要将所有进入 `localhost:8080` 的请求转发到 `http://example.com`，并且可以在转发之前和返回响应之后对请求和响应进行处理。

### 示例代码

```go
package main

import (
	"log"
	"net/http"
	"net/http/httputil"
	"net/url"
)

// 创建一个反向代理
func newReverseProxy(target string) *httputil.ReverseProxy {
	targetURL, err := url.Parse(target)
	if err != nil {
		log.Fatalf("Failed to parse target URL: %v", err)
	}

	// 自定义 Director 方法来修改请求
	proxy := httputil.NewSingleHostReverseProxy(targetURL)
	proxy.Director = func(req *http.Request) {
		// 修改请求的目标主机、路径和头部
		req.URL.Scheme = targetURL.Scheme
		req.URL.Host = targetURL.Host
		req.Host = targetURL.Host

		// 打印原始请求的 URL 和头部信息（用于调试）
		log.Printf("Proxying request: %s", req.URL)
	}

	// 修改响应数据
	proxy.ModifyResponse = func(res *http.Response) error {
		// 你可以在这里修改返回的响应，例如添加新的头部信息
		res.Header.Add("X-Reverse-Proxy", "Powered by Go")
		log.Printf("Response status: %s", res.Status)
		return nil
	}

	// 错误处理
	proxy.ErrorHandler = func(w http.ResponseWriter, r *http.Request, err error) {
		log.Printf("Proxy error: %v", err)
		http.Error(w, "Proxy error: "+err.Error(), http.StatusBadGateway)
	}

	return proxy
}

func main() {
	target := "http://example.com" // 替换为你需要转发的目标地址

	proxy := newReverseProxy(target)

	// 设置路由，将所有请求代理到目标地址
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		proxy.ServeHTTP(w, r)
	})

	log.Println("Starting reverse proxy on :8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### 代码说明

1. **`newReverseProxy` 函数**：
    - 通过 `httputil.NewSingleHostReverseProxy(targetURL)` 创建一个指向 `targetURL` 的反向代理对象。
    - 自定义 `Director` 方法用于修改请求。该方法会在每个请求被代理之前调用，用于修改请求的 URL、Host 和其他相关信息。
    - `ModifyResponse` 方法允许在代理服务器接收到目标服务器的响应后，对响应进行修改。此处我们为每个响应添加了自定义头部 `X-Reverse-Proxy`。
    - `ErrorHandler` 方法用于处理代理过程中出现的错误，并返回一个适当的错误响应给客户端。

2. **`http.HandleFunc` 路由**：
    - 将所有路径的请求 (`/`) 都代理到 `target` 目标服务器。

3. **启动服务器**：
    - `http.ListenAndServe(":8080", nil)` 在本地启动一个监听 `8080` 端口的 HTTP 服务器。

### 运行和测试

假设你将代理服务器设置为转发请求到 `http://example.com`，以下是如何测试这个代理服务器。

1. **运行程序**：

   运行程序后，代理服务器会监听 `localhost:8080`。

   ```bash
   go run main.go
   ```

2. **测试代理**：

   使用 `curl` 测试：

   ```bash
   curl -v http://localhost:8080
   ```

   你应该会看到请求被转发到 `http://example.com`，并且在返回的响应头中包含自定义的 `X-Reverse-Proxy`。

   ```text
   HTTP/1.1 200 OK
   X-Reverse-Proxy: Powered by Go
   Content-Type: text/html; charset=UTF-8
   Content-Length: 1256
   ...
   ```

3. **调试日志**：

   在代理过程中，日志将会打印出原始请求的 URL 和响应的状态码。例如：

   ```text
   Proxying request: http://example.com/
   Response status: 200 OK
   ```

### 功能扩展

你可以根据需要扩展此代码来实现更多功能：

1. **负载均衡**：可以将多个目标服务器添加到反向代理，并根据算法（如轮询、最少连接等）来选择目标服务器。

2. **SSL/TLS 支持**：可以使用 `http.ListenAndServeTLS` 来支持 HTTPS。

3. **缓存**：可以结合缓存机制，在代理服务器上缓存常见请求，以提高响应速度。

4. **请求限流**：可以使用中间件或者自定义逻辑来限制请求频率，防止代理服务器被过载。

### 总结

通过 `httputil.ReverseProxy`，你可以轻松地实现一个简单但功能强大的反向代理服务器。该示例展示了如何自定义请求和响应、处理错误，并将所有请求转发到指定的目标服务器。反向代理在许多场景中都非常有用，包括负载均衡、缓存、API 网关等场景。