---
title: "并发错误处理"
description: 
date: 2024-09-10T15:50:45+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
---

在 Go 语言中处理并发错误有多种方式，以下是几种常见的方法：

1. **使用 `sync.WaitGroup` 和 `error` 通道**：
   这种方式中，我们使用 `sync.WaitGroup` 来等待所有并发任务完成，同时使用 `error` 通道来收集各个任务产生的错误。

   ```go
   package main

   import (
       "fmt"
       "sync"
   )

   func worker(id int, wg *sync.WaitGroup, errCh chan error) {
       defer wg.Done()
       // 模拟可能会产生错误的任务
       if id%2 == 0 {
           errCh <- fmt.Errorf("error from worker %d", id)
       }
   }

   func main() {
       var wg sync.WaitGroup
       errCh := make(chan error, 10)

       for i := 0; i < 10; i++ {
           wg.Add(1)
           go worker(i, &wg, errCh)
       }

       wg.Wait()
       close(errCh)

       for err := range errCh {
           if err != nil {
               fmt.Println("Received error:", err)
           }
       }
   }
   ```

2. **使用 `context` 包**：
   `context` 包提供了控制多个 goroutine 生命周期的功能，特别是在处理超时和取消操作时非常有用。

   ```go
   package main

   import (
       "context"
       "fmt"
       "sync"
       "time"
   )

   func worker(ctx context.Context, id int, wg *sync.WaitGroup, errCh chan error) {
       defer wg.Done()

       select {
       case <-time.After(2 * time.Second):
           // 模拟可能会产生错误的任务
           if id%2 == 0 {
               errCh <- fmt.Errorf("error from worker %d", id)
           }
       case <-ctx.Done():
           fmt.Printf("worker %d canceled\n", id)
       }
   }

   func main() {
       var wg sync.WaitGroup
       errCh := make(chan error, 10)
       ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
       defer cancel()

       for i := 0; i < 10; i++ {
           wg.Add(1)
           go worker(ctx, i, &wg, errCh)
       }

       wg.Wait()
       close(errCh)

       for err := range errCh {
           if err != nil {
               fmt.Println("Received error:", err)
           }
       }
   }
   ```

3. **使用 `errgroup` 包**：
   `errgroup` 包提供了一种更方便的方法来处理并发任务和错误。

   ```go
   package main

   import (
       "context"
       "fmt"
       "golang.org/x/sync/errgroup"
   )

   func worker(id int) error {
       // 模拟可能会产生错误的任务
       if id%2 == 0 {
           return fmt.Errorf("error from worker %d", id)
       }
       return nil
   }

   func main() {
       g, ctx := errgroup.WithContext(context.Background())

       for i := 0; i < 10; i++ {
           i := i // 避免闭包捕获变量问题
           g.Go(func() error {
               return worker(i)
           })
       }

       if err := g.Wait(); err != nil {
           fmt.Println("Received error:", err)
       } else {
           fmt.Println("All workers finished without error")
       }
   }
   ```

这些方法各有优缺点，选择哪种方式取决于具体的应用场景和需求。