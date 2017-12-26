title: NSOperation详解
date: 2016-07-19 23:10:23
categories:
- iOS
tags:
- 笔记
- iOS


---

# NSOperation详解

## NSOperation是什么?
NSOperation 是oc中多线程技术的一种，是基于 GCD 做的面向对象的封装。相比较与GCD的使用更为简单，并且提供了一些用GCD不是很好实现的功能：

 * 取消在任务处理队列中的任务
 * 添加任务间的依赖关系
 * 设置任务的最大并发数
 * 提供了任务的状态：isExecuteing, isFinished.
 
 
### NSOperation的主要特点
NSOperation是一个抽象类，不能直接使用，要实现它的子类才可以使用。实现NSOperation子类的方式有3种：

1. NSInvocationOperation
2. NSBlockOperation
3. 自定义子类继承NSOperation，实现内部相应的方法


NSOperation的一些主要属性

```
@property (readonly, getter=isCancelled) BOOL cancelled;

@property (readonly, getter=isExecuting) BOOL executing;

@property (readonly, getter=isFinished) BOOL finished;

```
属性的基本使用会在下面说到。



## 如何使用NSOperation
按照之前讲述的NSOperation是一个抽象类，所以我们使用时，必须用其子类才可以使用：
先看一个小例子：
### NSInvocationOperation
使用的时候我们可以直接创建操作，然后启动就可以了，操作会在当前线程中**同步执行**任务。


```
NSOperation *operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(doSomeThings:) object:@"Invocation"];
    
[operation start];

```

我们在GCD中并发（全局）队列使用的最多，所以NSOperation直接把GCD里面的并发队列封装起来。NSOperationQueue队列，本质上就是GCD的并发队列。
我们还可以将操作放到队列中，只要将操作添加到队列，操作会自动**异步执行**调度方法。

```
NSOperation *operation = [[NSInvocationOperation alloc]initWithTarget:self selector:@selector(doSomeThings:) object:@"Invocation"];

NSOperationQueue *queue = [[NSOperationQueue alloc] init];
[queue addOperation:operation];

```

### NSBlockOperation
NSBlockOperation使用时是直接将block中的操作放到异步执行。


```
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
NSBlockOperation * operation = [NSBlockOperation blockOperationWithBlock:^{
			
            NSLog(@"%@", [NSThread currentThread]);
}];
[queue addOperation:operation];
       

```

线程间通信

```
 NSOperationQueue *queue = [[NSOperationQueue alloc]init];
    
    
    
 [queue addOperationWithBlock:^{
     NSLog(@"耗时操作....%@", [NSThread currentThread]);
        
     // 在主线程更新UI
       [[NSOperationQueue mainQueue] addOperationWithBlock:^{
            NSLog(@"更新UI....%@", [NSThread currentThread]);
       }];
  }];
```

### 自定义子类继承NSOperation
你也可以实现自己的子类，通过重写**main**方法或者**start**方法来定义自己的operations。

#### 创建非并发的NSOperation子类
创建非并发的NSOperation子类很简单，重写main方法就行了。
使用main方法非常简单，开发者不需要管理一些Operation的一些状态属性。当main方法执行完毕的时候，这个Operation也就结束了。这种只重写main方法的使用起来简单，灵活性不如重写start方法。因为main方法执行完就可以认为operation结束了，所以一般可用来做执行同步任务。

```
@implementation YourOperation
- (void)main
{
    // 任务代码 ...
}
@end
```
#### 创建并发的NSOperation子类
重写start方法可以使你拥有更多的可操作性，如果你想再一个操作中执行异步任务，就重写start方法，但是这样你就必须得手动管理操作的状态了。这就涉及到前面所说的NSOperation的属性了。NSOperation提供了**ready** **cancelled** **executing**  **finished** 这几个状态的变化，并且这些状态都是通过设置了keypath的kvo通知所决定的。我们开发的时候也就是关注这些其中的状态。所以当我们重写NSOperation的start方法的时候，我们必须得手动的修改这些状态。上面的这些状态都是BOOL值。finished这个状态在操作完成后请及时设置为YES，因为NSOperationQueue所管理的队列中，只有isFinished为YES时才将其移除队列，这点在内存管理和避免死锁很关键。



```

@interface SavePhotoOperation() {
@private
    BOOL executing;
    BOOL finished;
    BOOL cancelled;
}
@end

@implementation PASavePhotoOperation



- (void)start {
    // Always check for cancellation before launching the task.
    if ([self isCancelled]) {
        // Must move the operation to the finished state if it is canceled.
        [self willChangeValueForKey:@"isFinished"];
        finished = YES;
        [self didChangeValueForKey:@"isFinished"];
        return;
    }
    
    // If the operation is not canceled, begin executing the task.
    [self willChangeValueForKey:@"isExecuting"];
    [NSThread detachNewThreadSelector:@selector(main) toTarget:self withObject:nil];
    executing = YES;
    [self didChangeValueForKey:@"isExecuting"];
    
}


- (void)main {
  
   @try {
        // Do the main work of the operation here.

        [self completeOperation];
    }
    @catch(...) {
        // Do not rethrow exceptions.
    }
    
}

- (void)completeOperation {
    [self willChangeValueForKey:@"isFinished"];
    [self willChangeValueForKey:@"isExecuting"];
    
    executing = NO;
    finished = YES;
    
    [self didChangeValueForKey:@"isExecuting"];
    [self didChangeValueForKey:@"isFinished"];
}

- (BOOL)isConcurrent {
    return YES;
}

- (void)cancel {
    [self willChangeValueForKey:@"isCancelled"];
    [self willChangeValueForKey:@"isExecuting"];
    [self willChangeValueForKey:@"isExecuting"];
    
    cancelled = YES;
    executing = NO;
    finished = YES;
    
    [self didChangeValueForKey:@"isCancelled"];
    [self didChangeValueForKey:@"isExecuting"];
    [self didChangeValueForKey:@"isFinished"];
}

- (BOOL)isExecuting {
    return executing;
}

- (BOOL)isFinished {
    return finished;
}

- (BOOL)isCancelled {
    return cancelled;
}

@end

```
上面就是一个基本的自定义子类继承的NSOperation的写法，其中手动发送KVO消息。通知状态更改如下：

```
    [self willChangeValueForKey:@"isCancelled"];
    [self willChangeValueForKey:@"isExecuting"];
    [self willChangeValueForKey:@"isExecuting"];
    
    cancelled = YES;
    executing = NO;
    finished = YES;
    
    [self didChangeValueForKey:@"isCancelled"];
    [self didChangeValueForKey:@"isExecuting"];
    [self didChangeValueForKey:@"isFinished"];
```

#### NSOperation依赖关系

我们可以设定各个任务之间的依赖关系，等到一个任务执行完毕之后才能执行之后的任务。


```
  NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"1. 下载一个小说的压缩包, %@",[NSThread currentThread]);
   }];
  NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"2. 解压缩，删除压缩包, %@",[NSThread currentThread]);
    }];
  NSBlockOperation *op3 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"3. 更新UI, %@",[NSThread currentThread]);
    }];
    
    // 指定任务之间的依赖关系 -- 依赖关系可以跨队列(可以在子线程下载完，到主线程更新UI)
    
    [op2 addDependency:op1];
    [op3 addDependency:op2];
    // waitUntilFinished 类似GCD的调度组的通知
    // NO 不等待，会直接执行  NSLog(@"come here");
    // YES 等待上面的操作执行结束，再 执行  NSLog(@"come here")
    [self.opQueue addOperations:@[op1, op2] waitUntilFinished:YES];
    
    // 在主线程更新UI
    [[NSOperationQueue mainQueue] addOperation:op3];
    NSLog(@"come here");


```

**注意点**：千万不要出现循环依赖，也就是说不要出现A依赖B，B又依赖A的关系。

####  NSOperation最大并发数
设置了最大并发数之后，同时执行的操作的数量便是最大并发数的数量。

#### NSOperation任务的处理
我们可以取消操作队列中得任务，也可以挂起操作队列中得任务，都是由操作队列中的属性来进行控制的。

```
// 取消队列的所有操作
    [self.opQueue cancelAllOperations]; // 取消队列的所有操作，会把任务从队列里面全部删除
    
    NSLog(@"取消所有的操作");
    
 // 取消队列的挂起状态
 // （只要是取消了队列的操作，我们就把队列处于启动状态。以便于队列的继续）
 self.opQueue.suspended = NO;

```
