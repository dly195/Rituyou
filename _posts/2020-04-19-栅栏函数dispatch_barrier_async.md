---
layout: post
title:  "栅栏函数dispatch_barrier_async"
date:   2020-04-19 18:30:00
categories: coding
tags: [Objective-C,iOS]
excerpt: 用栅栏函数解决一个标准的多线程数据竞争问题
---

# 利用GCD的栅栏函数dispatch_barrier_async解决数据竞争

### 问题：

我们在http请求的header中放了一个字符串来标识用户的一些信息，这个字符串可能会随着用户进行的某些操作发生变化。而用户在进行某些页面操作的时候，可能会出现多个线程同时访问这个字符串的情况。我们不仅希望可以保证读取的安全性，还想保证读取的正确性。
这是一个标准的多线程数据竞争的问题。

### 分析：

因为读取操作的需求是远大于写入操作的，我们没有必要限制读取的操作，希望读取操作可以并发执行，以节省读取时的等待时间；
我们希望每次读取到的数据都是准确的数据，所以读和写之间必须按顺序执行。

### 解决方案：

最终的方案是使用GCD的栅栏函数：**dispatch_barrier_async**

**dispatch_barrier_async在进程管理中起到一个栅栏的作用，也就是说它会把当前的任务保护起来，等到队列中其它之前的任务都完成之后再执行栅栏内的任务。**

1. 创建一个并发队列，将读取操作全部放到这个队列当中。**我们可以让所有的读取都并发执行**。
2. 使用dispatch_barrier_async函数，将写入的操作放入这个队列中。**依靠栅栏的特征，保证写入操作是在前面的读取任务全部完成之后才会进行。**

####代码实例

```
- (void)writeInBarrier {

    dispatch_queue_t queue = dispatch_queue_create("ReadOrWrite", DISPATCH_QUEUE_CONCURRENT);
    dispatch_barrier_async(queue, ^{
        self.testStr = @"barrier1";
        NSLog(@"write ----%@", self.testStr);
    });
    for (int i = 0 ; i < 5; i++) {
        dispatch_async(queue, ^{
            NSLog(@"read  ---%d----%@", i,self.testStr);
        });
    }
    
    for (int i = 1 ; i < 5; i++) {
        
        dispatch_barrier_async(queue, ^{
            self.testStr = [NSString stringWithFormat:@"barrier%d",i];
            NSLog(@"write ---%d----%@", i,self.testStr);
        });
        
        for (int j = 0 ; j < 5; j++) {
            dispatch_async(queue, ^{
                NSLog(@"read  ---%d----%@", j,self.testStr);
            });
        }
    }
}

```

输出结果:

```

write ----barrier1
read  ---2----barrier1
read  ---1----barrier1
read  ---0----barrier1
read  ---4----barrier1
read  ---3----barrier1
write ---1----barrier1
read  ---0----barrier1
read  ---2----barrier1
read  ---3----barrier1
read  ---1----barrier1
read  ---4----barrier1
write ---2----barrier2
read  ---0----barrier2
read  ---2----barrier2
read  ---1----barrier2
read  ---4----barrier2
read  ---3----barrier2
write ---3----barrier3
read  ---0----barrier3
read  ---2----barrier3
read  ---3----barrier3
read  ---1----barrier3
read  ---4----barrier3
write ---4----barrier4
read  ---0----barrier4
read  ---1----barrier4
read  ---2----barrier4
read  ---3----barrier4
read  ---4----barrier4
```

###注意点

dispatch_barrier_async 函数必须配合自己create的并行队列使用

```
 * @function dispatch_barrier_sync_f
 *
 * @abstract
 * Submits a barrier function for synchronous execution on a dispatch queue.
 *
 * @discussion
 * Submits a function to a dispatch queue like dispatch_sync_f(), but marks that
 * fuction as a barrier (relevant only on DISPATCH_QUEUE_CONCURRENT queues).
```

  
  
  






