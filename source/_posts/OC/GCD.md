---
title: 多线程 — GCD
date: 2016-06-14 
tags: OC
---
# 简介
GCD 全称为 Grand Central Dispatch，是 libdispatch 的市场名称，而 libdispatch 是 Apple 的一个库，其为并发代码在 iOS 和 macOS 的多核硬件上执行提供支持。确切地说 GCD 是一套低层级的C API，通过 GCD，开发者只需要向队列中添加一段代码块(block或C函数指针)，而不需要直接和线程打交道。GCD在后端管理着一个线程池，它不仅决定着你的代码块将在哪个线程被执行，还根据可用的系统资源对这些线程进行管理。这样通过 GCD 来管理线程，从而解决线程被创建的问题。

<!-- more -->
![gcd_oc](http://o7ttfnm00.bkt.clouddn.com/jiaotong.jpg)

* [官方文档](https://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/)
* [WWDC](https://developer.apple.com/videos/play/wwdc2015/718/)
***
# 创建队列
## dispatch_queue_create
主队列：一个特殊的串行队列，任何需要刷新 UI 的工作都要在主队列执行，所以一般耗时的任务都要放到别的线程执行。

```
dispatch_queue_t queue = dispatch_get_main_queue(); //OC
let queue = dispatch_get_main_queue()               //Swift
```

手动创建队列：可以创建 串行队列, 也可以创建 并行队列。第一个参数是标识符，第二个参数用来表示创建的队列是串行的还是并行的。DISPATCH_QUEUE_SERIAL / NULL 串行队列；DISPATCH_QUEUE_CONCURRENT 并行队列。

```
//OC
dispatch_queue_t queue = dispatch_queue_create("tk.bourne.testQueue", NULL);
dispatch_queue_t queue = dispatch_queue_create("tk.bourne.testQueue", DISPATCH_QUEUE_SERIAL);
dispatch_queue_t queue = dispatch_queue_create("tk.bourne.testQueue", DISPATCH_QUEUE_CONCURRENT);

//Swift
let queue = dispatch_queue_create("tk.bourne.testQueue", nil);
let queue = dispatch_queue_create("tk.bourne.testQueue", DISPATCH_QUEUE_SERIAL)
let queue = dispatch_queue_create("tk.bourne.testQueue", DISPATCH_QUEUE_CONCURRENT)
```

***
# 创建任务
## dispatch\_async / dispatch\_sync
同步派发(sync)会尽可能地在当前线程派发任务。但如果在其他队列往主队列同步派发，任务会在主线程执行；
异步派发(async)也不绝对会另开线程，例如在主线程异步派发到主线程，派发依旧是异步的，任务也会在主线程执行。

* dispatch_sync 同步任务：会阻塞当前线程；
* dispatch_async 异步任务：不会阻塞当前线程

```
//OC
dispatch_sync(<#queue#>, ^{
    NSLog(@"%@", [NSThread currentThread]);
});
dispatch_async(<#queue#>, ^{
    NSLog(@"%@", [NSThread currentThread]);
});

//Swift
dispatch_sync(<#queue#>, { () -> Void in
    println(NSThread.currentThread())
})
dispatch_async(<#queue#>, { () -> Void in
    println(NSThread.currentThread())
})
```
***
## dispatch_after
dispatch_after只是延时提交block，不是延时立刻执行。

```
double delayInSeconds = 2.0;
dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t) (delayInSeconds * NSEC_PER_SEC));
dispatch_after(popTime, dispatch_get_main_queue(), ^(void){
    [self bar];
});
```

## dispatch_set_target_queue
dispatch_set_target_queue可以设置queue的优先级，也可以使多个serial queue在目标queue上一次只有一个执行。

（如果将多个串行的queue使用dispatch\_set\_target\_queue指定到了同一目标，那么多个串行queue在目标queue上就是同步执行的，不再是并行执行。
例如，把一个任务放到一个串行的queue中，如果这个任务被拆分了，被放置到多个串行的queue中，但实际还是需要这个任务同步执行，那么就会有问题，因为多个串行queue之间是并行的。这就可以使用dispatch\_set\_target\_queue了。）

```
dispatch_queue_t serialQueue = dispatch_queue_create("com.starming.gcddemo.serialqueue", DISPATCH_QUEUE_SERIAL);
dispatch_queue_t firstQueue = dispatch_queue_create("com.starming.gcddemo.firstqueue", DISPATCH_QUEUE_SERIAL);
dispatch_queue_t secondQueue = dispatch_queue_create("com.starming.gcddemo.secondqueue", DISPATCH_QUEUE_CONCURRENT);

dispatch_set_target_queue(firstQueue, serialQueue);
dispatch_set_target_queue(secondQueue, serialQueue);

dispatch_async(firstQueue, ^{
    NSLog(@"1");
    [NSThread sleepForTimeInterval:3.f];
});
dispatch_async(secondQueue, ^{
    NSLog(@"2");
    [NSThread sleepForTimeInterval:2.f];
});
dispatch_async(secondQueue, ^{
    NSLog(@"3");
    [NSThread sleepForTimeInterval:1.f];
});
```

打印结果1、2、3。
***
## dispatch_barrier_async / dispatch_barrier_sync
dispatch_barrier_async 这个函数可以设置同步执行的block，它会等到在它加入队列之前的block执行完毕后，才开始执行。在它之后加入队列的block，则等到这个block执行完毕后才开始执行。

dispatch_barrier_sync 同上，除了它是同步返回函数。
```
//防止文件读写冲突，可以创建一个串行队列，操作都在这个队列中进行，没有更新数据读用并行，写用串行。
dispatch_queue_t dataQueue = dispatch_queue_create("com.starming.gcddemo.dataqueue", DISPATCH_QUEUE_SERIAL);
dispatch_async(dataQueue, ^{
    [NSThread sleepForTimeInterval:2.f];
    NSLog(@"read 1");
});
dispatch_async(dataQueue, ^{
    NSLog(@"read 2");
});
//等待前面的都完成，在执行barrier后面的
dispatch_barrier_async(dataQueue, ^{
    NSLog(@"write 1");
    [NSThread sleepForTimeInterval:1];
});
dispatch_async(dataQueue, ^{
    [NSThread sleepForTimeInterval:1.f];
    NSLog(@"read 3");
});
dispatch_async(dataQueue, ^{
    NSLog(@"read 4");
});
```

打印结果：read 1、read 2、write 1、read 3、read 4。

***

## dispatch_apply
dispatch_apply类似一个for循环，会在指定的dispatch queue中运行block任务n次，如果队列是并发队列，则会并发执行block任务，dispatch_apply是一个同步调用，block任务执行n次后才返回。

需要注意的是这个方法是同步返回，也就是说等到所有block执行完毕才返回，所以这里会阻塞主线程，如需异步返回，使用dispatch_async包一下就不会阻塞了。多个block的运行是否并发或串行执行也依赖queue的是否并发或串行。

```
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.starming.gcddemo.concurrentqueue", DISPATCH_QUEUE_CONCURRENT);
dispatch_apply(10, concurrentQueue, ^(size_t i) {
    NSLog(@"%zu",i);
});
dispatch_async(dispatch_get_main_queue(), ^{
    dispatch_apply(10, concurrentQueue, ^(size_t i) {
        NSLog(@"%zu",i);
    });
});
NSLog(@"The end");
//打印结果：0、2、4、1、3、6、5、7、9、8、The end、0、4、1、3、5、2、8、6、7、9。
```

对比：

```
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.starming.gcddemo.concurrentqueue",DISPATCH_QUEUE_CONCURRENT);
if (explode) {
    //有问题的情况，可能会死锁
    for (int i = 0; i < 999 ; i++) {
        dispatch_async(concurrentQueue, ^{
            NSLog(@"wrong %d",i);
            //do something hard
        });
    }
} else {
    //会优化很多，能够利用GCD管理
    dispatch_apply(999, concurrentQueue, ^(size_t i){
        NSLog(@"correct %zu",i);
        //do something hard
    });
    NSLog(@"----");
}
```

***

# Dispatch Block
## dispatch\_block_create
自己创建block并添加到queue中去执行。并且，在创建block时可以通过设置QoS，指定block对应的优先级，在dispatch\_block\_create\_with\_qos\_class中指定QoS类别即可。

```
//normal way
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.starming.gcddemo.concurrentqueue",DISPATCH_QUEUE_CONCURRENT);
dispatch_block_t block = dispatch_block_create(0, ^{
    NSLog(@"run block");
});
dispatch_async(concurrentQueue, block);

//QOS way
dispatch_block_t qosBlock = dispatch_block_create_with_qos_class(0, QOS_CLASS_USER_INITIATED, -1, ^{
    NSLog(@"run qos block");
});
dispatch_async(concurrentQueue, qosBlock);
```

***

## dispatch\_block_wait
可以根据dispatch block来设置等待时间，参数DISPATCH\_TIME_FOREVER会一直等待block结束。

```
dispatch_queue_t serialQueue = dispatch_queue_create("com.starming.gcddemo.serialqueue", DISPATCH_QUEUE_SERIAL);
dispatch_block_t block = dispatch_block_create(0, ^{
    NSLog(@"star");   
    [NSThread sleepForTimeInterval:5.f];
    NSLog(@"end");
});
dispatch_async(serialQueue, block);
//设置DISPATCH_TIME_FOREVER后，会一直等到前面任务都完成
dispatch_block_wait(block, DISPATCH_TIME_FOREVER);
NSLog(@"ok, now can go on");//打印结果：star、end、ok, now can go on。
```

***

## dispatch\_block_notify
dispatch\_block_notify当观察的某个block执行结束之后立刻通知提交另一特定的block到指定的queue中执行，该函数有三个参数，第一参数是需要观察的block，第二个参数是被通知block提交执行的queue，第三参数是当需要被通知执行的block，函数的原型。

```
dispatch_queue_t serialQueue = dispatch_queue_create("com.Kevin.serialqueue", DISPATCH_QUEUE_SERIAL);
dispatch_block_t firstBlock = dispatch_block_create(0, ^{
    NSLog(@"first block start");
    [NSThread sleepForTimeInterval:2.f];
    NSLog(@"first block end");
});
dispatch_async(serialQueue, firstBlock);
dispatch_block_t secondBlock = dispatch_block_create(0, ^{
    NSLog(@"second block run");
});
//first block执行完才在serial queue中执行second block
dispatch_block_notify(firstBlock, serialQueue, secondBlock);

//打印结果：first block start、first block end、second block run。
```

***
## dispatch\_block_cancel
iOS8后GCD支持对dispatch block的取消

```
dispatch_queue_t serialQueue = dispatch_queue_create("com.Kevin.serialqueue", DISPATCH_QUEUE_SERIAL);
dispatch_block_t firstBlock = dispatch_block_create(0, ^{
    NSLog(@"first block start");
    [NSThread sleepForTimeInterval:2.f];
    NSLog(@"first block end");
});
dispatch_block_t secondBlock = dispatch_block_create(0, ^{
    NSLog(@"second block run");
});
dispatch_async(serialQueue, firstBlock);
dispatch_async(serialQueue, secondBlock);
//取消secondBlock
dispatch_block_cancel(secondBlock);
//打印结果：first block start、first block end。
```

***

# Dispatch_groups
当我们想在gcd queue中所有的任务执行完毕之后做些特定事情的时候，也就是队列的同步问题，如果队列是串行的话，那将该操作最后添加到队列中即可，但如果队列是并行队列的话，这时候就可以利用 dispatch\_group 来实现了，dispatch\_group 能很方便的解决同步的问题。dispatch\_group_create可以创建一个group对象，然后可以添加block到该组里面。

***
## dispatch\_group_create
创建dispatch\_group_t

```
dispatch_group_t group = dispatch_group_create();
```
***
## dispatch\_group_async
自己创建队列时，当然就用dispatch\_group_async函数，简单有效。

```
dispatch_group_async(group, queue, ^{
    //Do you work...
});
```

***

## dispatch\_group_wait
dispatch\_group_wait会同步地等待group中所有的block执行完毕后才继续执行,类似于dispatch barrier

```
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.Kevin.concurrentqueue",DISPATCH_QUEUE_CONCURRENT);
dispatch_group_t group = dispatch_group_create();
//在group中添加队列的block
dispatch_group_async(group, concurrentQueue, ^{
    [NSThread sleepForTimeInterval:2.f];
    NSLog(@"1");
});
dispatch_group_async(group, concurrentQueue, ^{
    NSLog(@"2");
});
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
NSLog(@"can continue");//打印结果：2、1、can continue。
```

***

## dispatch\_group_notify
功能与dispatch\_group\_wait类似，不过该过程是异步的，不会阻塞该线程，dispatch\_group\_notify有三个参数,第一个参数指定要观察的group，第二个参数指定block待执行的队列，第三个参数指定group中所有任务执行完毕之后要执行的block。

```
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.Kevin.concurrentqueue",DISPATCH_QUEUE_CONCURRENT);
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, concurrentQueue, ^{
   [NSThread sleepForTimeInterval:2.f];
    NSLog(@"1");
});
dispatch_group_async(group, concurrentQueue, ^{
    NSLog(@"2");
});
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    NSLog(@"end");
});
[NSThread sleepForTimeInterval:2.f];
NSLog(@"can continue");//打印结果：2、can continue、1、end。
```

***
## dispatch\_group\_enter / dispatch\_group\_leave

```
AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];

//Enter group
dispatch_group_enter(group);
[manager GET:@"http://www.baidu.com" parameters:nil success:^(AFHTTPRequestOperation *operation, id responseObject) {
    
    //Leave group
    dispatch_group_leave(group);
}    failure:^(AFHTTPRequestOperation *operation, NSError *error) {

    //Leave group
    dispatch_group_leave(group);
}];
```

***

## dispatch\_semaphore_create
dispatch semaphore用来做解决一些同步的问题，dispatch\_semaphore\_create会创建一个信号量，该函数需要传递一个信号值，dispatch\_semaphore\_signal会使信号值加1，如果信号值的大小等于1，dispatch\_semaphore\_wait会使信号值减1，并继续往下走，如果信号值为0，则等待。

```
dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"start");
    [NSThread sleepForTimeInterval:1.f];
    NSLog(@"semaphore +1");
    dispatch_semaphore_signal(semaphore); //+1 semaphore
});
dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
NSLog(@"continue");//打印结果：start、semaphore +1、continue。
```

***
# Dispatch Source
dispatch源（dispatch source）和RunLoop源概念上有些类似的地方，而且使用起来更简单。要很好地理解dispatch源，其实把它看成一种特别的生产消费模式。dispatch源好比生产的数据，当有新数据时，会自动在dispatch指定的队列（即消费队列）上运行相应地block，生产和消费同步是dispatch源会自动管理的。

Dispatch Source用于监听系统的底层对象，比如文件描述符，Mach端口，信号量等。主要处理的事件如下表：

| Methods                              | explain     |
| ------------------------------------ |:-----------:|
| DISPATCH_SOURCE\_TYPE\_DATA\_ADD        | 变量增加      | 
| DISPATCH_SOURCE\_TYPE\_DATA\_OR         | 变量OR       | 
| DISPATCH_SOURCE\_TYPE\_MACH\_SEND       | Mach端口发送  |
| DISPATCH_SOURCE\_TYPE\_MACH\_RECV       | MACH端口接收  |
| DISPATCH_SOURCE\_TYPE\_MEMORYPRESSURE  | 内存压力  |
| DISPATCH_SOURCE\_TYPE\_PROC            | 进程监听,如进程的退出、创建一个或更多的子线程、进程收到UNIX信号  |
| DISPATCH_SOURCE\_TYPE\_READ            | IO操作，如对文件的操作、socket操作的读响应  |
| DISPATCH_SOURCE\_TYPE\_SIGNAL          | 接收到UNIX信号时响应 |
| DISPATCH_SOURCE\_TYPE\_TIMER           | 定时器  |
| DISPATCH_SOURCE\_TYPE\_VNODE           | 文件状态监听，文件被删除、移动、重命名  |
| DISPATCH_SOURCE\_TYPE\_WRITE           | IO操作，如对文件的操作、socket操作的写响应  |

####方法：
*  dispatch\_source_create：创建dispatch source，创建后会处于挂起状态进行事件接收，需要设置事件处理handler进行事件处理。
*  dispatch_source\_set\_event\_handler：设置事件处理handler
*  dispatch\_source_cancel：关闭dispatch source，设置的事件处理handler不会被执行，已经执行的事件handler不会取消。

```
//监视文件夹内文件变化
NSURL *directoryURL; //指定需要监听的文件夹路径
int const fd = open([[directoryURL path] fileSystemRepresentation], O_EVTONLY);
if (fd < 0) {
    char buffer[80];
    strerror_r(errno, buffer, sizeof(buffer));
    NSLog(@"Unable to open \"%@\": %s (%d)", [directoryURL path], buffer, errno);
    return;
}

//创建dispatch源，这里使用加法来合并dispatch源数据，最后一个参数是指定dispatch队列
dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_VNODE, fd,
                                                  DISPATCH_VNODE_WRITE | DISPATCH_VNODE_DELETE, DISPATCH_TARGET_QUEUE_DEFAULT);

//设置响应dispatch源事件的block，在dispatch源指定的队列上运行
dispatch_source_set_event_handler(source, ^(){
    
    //可以通过dispatch_source_get_data(source)来得到dispatch源数据
    unsigned long const data = dispatch_source_get_data(source);
    if (data & DISPATCH_VNODE_WRITE) {
        NSLog(@"The directory changed.");
    }
    if (data & DISPATCH_VNODE_DELETE) {
        NSLog(@"The directory has been deleted.");
    }
});
dispatch_source_set_cancel_handler(source, ^(){
    close(fd);
});
//dispatch源创建后处于suspend状态，所以需要启动dispatch源
dispatch_resume(source);
//还要注意需要用DISPATCH_VNODE_DELETE 去检查监视的文件或文件夹是否被删除，如果删除了就停止监听
```

***

## dispatch\_time_t

```
dispatch_time_t delayTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0/*延迟执行时间*/ * NSEC_PER_SEC));

dispatch_after(delayTime, dispatch_get_main_queue(), ^{
    [weakSelf delayMethod];
});
```

***

## dispatch_source\_set\_timer

```
dispatch_source_set_timer(dispatch_source_t source, dispatch_time_t start, uint64_t interval, uint64_t leeway);
```

第一个参数:定时器对象；第二个参数:DISPATCH\_TIME_NOW 表示从现在开始计时；第三个参数:间隔时间 GCD里面的时间最小单位为 纳秒；第四个参数:精准度(表示允许的误差,0表示绝对精准)。  

NSTimer在主线程的runloop里会在runloop切换其它模式时停止，这时就需要手动在子线程开启一个模式为NSRunLoopCommonModes的runloop，如果不想开启一个新的runloop可以用不跟runloop关联的dispatch source timer。

   
NSEC\_PER_SEC 1000000000ull  
USEC\_PER_SEC 1000000ull  
NSEC\_PER_USEC 1000ull  

NSEC：纳秒；USEC：微妙；SEC：秒；PER：每。

```
//第一个参数代表：dispatch source类型，最后一个是block会进入的queue，用来执行事件处理器和取消处理器，第二三个参数在会根据source类型设置。
dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER,0, 0, DISPATCH_TARGET_QUEUE_DEFAULT);

//设置事件的处理handler
dispatch_source_set_event_handler(source, ^(){
    NSLog(@"Time flies.");
});

//5秒触发一次，误差100毫秒
dispatch_source_set_timer(source, DISPATCH_TIME_NOW, 5ull * NSEC_PER_SEC,100ull * NSEC_PER_MSEC);

//开始处理定时器事件，dispatch_suspend暂停处理事件
dispatch_resume(source);

```

***

## dispatch_suspend和dispatch\_resume
*  dispatch_suspend 挂起队列
*  dispatch_resume  恢复队列

dispatch_suspend这里挂起不会暂停正在执行的block，只是能够暂停还没执行的block。

```
dispatch_queue_t queue = dispatch_queue_create("me.kevin.gcd", DISPATCH_QUEUE_SERIAL);

//提交第一个block，延时5秒打印。
dispatch_async(queue, ^{
    [NSThread sleepForTimeInterval:5];
    NSLog(@"After 5 seconds...");
});

//提交第二个block，也是延时5秒打印
dispatch_async(queue, ^{
    [NSThread sleepForTimeInterval:5];
    NSLog(@"After 5 seconds again...");
});

//延时一秒
NSLog(@"sleep 1 second...");
[NSThread sleepForTimeInterval:1];

//挂起队列
NSLog(@"suspend...");
dispatch_suspend(queue);

//延时10秒
NSLog(@"sleep 10 second...");
[NSThread sleepForTimeInterval:10];

//恢复队列
NSLog(@"resume...");
dispatch_resume(queue);
```

可知，在dispatch_suspend挂起队列后，第一个block还是在运行，并且正常输出。
结合文档，我们可以得知，dispatch_suspend并不会立即暂停正在运行的block，而是在当前block执行完成后，暂停后续的block执行。

***

# 死锁！
## dispatch_sync导致的死锁
在main线程使用“同步”方法提交Block，必定会死锁：

```
dispatch_sync(dispatch_get_main_queue(), ^{
    NSLog(@"I am block...");
});
```

嵌套调用可能就会造成死锁：

```
- (void)updateUI1 {
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"Update ui 1");

        //死锁！
        [self updateUI2];
    });
}
- (void)updateUI2 {
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"Update ui 2");
    });
}
```

其它情况：

```
- (void)deadLockCase1 {
    NSLog(@"1");
    //主队列的同步线程，按照FIFO的原则（先入先出），2排在3后面会等3执行完，但因为同步线程，3又要等2执行完，相互等待成为死锁。
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"2");
    });
    NSLog(@"3");
}
- (void)deadLockCase2 {
    NSLog(@"1");
    //3会等2，因为2在全局并行队列里，不需要等待3，这样2执行完回到主队列，3就开始执行
    dispatch_sync(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
        NSLog(@"2");
    });
    NSLog(@"3");
}
- (void)deadLockCase3 {
    dispatch_queue_t serialQueue = dispatch_queue_create("com.starming.gcddemo.serialqueue", DISPATCH_QUEUE_SERIAL);
    NSLog(@"1");
    dispatch_async(serialQueue, ^{
        NSLog(@"2");
        //串行队列里面同步一个串行队列就会死锁
        dispatch_sync(serialQueue, ^{
            NSLog(@"3");
        });
        NSLog(@"4");
    });
    NSLog(@"5");
}
```

***

## dispatch_apply导致的死锁:

```
//在串行队列里嵌套使用dispatch_apply
dispatch_queue_t queue = dispatch_queue_create("me.tutuge.test.gcd", DISPATCH_QUEUE_SERIAL);

dispatch_apply(3, queue, ^(size_t i) {
	NSLog(@"apply loop: %zu", i);

    //再来一个dispatch_apply！死锁！
	dispatch_apply(3, queue, ^(size_t j) {
		NSLog(@"apply loop inside %zu", j);
	});
});
```
