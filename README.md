# NetWork
多线程网络Tips

## 多线程网络

#### 线程和进程的区别？一个程序至少有一个进程,一个进程至少有一个线程
进程：一个程序的一次运行，在执行过程中拥有独立的内存单元，而多个线程共享一块内存
线程：线程是指进程内的一个执行单元。
联系：线程是进程的基本组成单位

区别：
1. 调度：线程作为调度和分配的基本单位，进程作为拥有资源的基本单位
2. 并发性：不仅进程之间可以并发执行，同一个进程的多个线程之间也可并发执行
3. 拥有资源：进程是拥有资源的一个独立单位，线程不拥有系统资源，但可以访问隶属于进程的资源.
4. 系统开销：在创建或撤消进程时，由于系统都要为之分配和回收资源，导致系统的开销明显大于创建或撤消线程时的开销。
举例说明：操作系统有多个软件在运行（QQ、office、音乐等），这些都是一个个进程，而每个进程里又有好多线程（比如QQ，你可以同时聊天，发送文件等）

* 如果有大量的线程,会影响性能,因为操作系统需要在它们之间切换。CPU调度时间缓慢，会出现卡的情况
* 更多的线程需要更多的内存空间。主线程1M,子线程512K
* 通常块模型数据是在多个线程间共享的，需要防止线程死锁情况的发生。
* @synchronized()必须要是同一把锁，NSLcok或者self

*  同一时间，cpu只能执行一条线程，那为什么开启多条线程能达到同时执行的效果呢？？
cpu 再同一时间只执行一条线程，但是，cpu再很短的时间片内切换与不同的线程，所以表面上开起来多线程同时执行
*  cpu再时间片内的切换--->调度

#### 线程概念

1. 主线程 ： UI线程，显示、刷新UI界面，处理UI控件的事件
2. 子线程 ： 后台线程，异步线程
3. 不要把耗时的操作放在主线程，要放在子线程中执行

#### NSThread

1.先创建，后启动
```objc
NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(download:) object:nil];
[thread start];
```

2.创建完自动启动 --> 分离出子线程
```objc
[NSThread detachNewThreadSelector:@selector(download:) toTarget:self withObject:nil];
```

3.隐式创建（自动启动）,在后台线程(子线程中)执行
```objc
[self performSelectorInBackground:@selector(download:) withObject:nil];
```
**常见方法**
```
1> 获得当前线程
+ (NSThread *)currentThread;

2> 获得主线程
+ (NSThread *)mainThread;

3> 睡眠（暂停），退出线程
+ (void)sleepUntilDate:(NSDate *)date;
+ (void)sleepForTimeInterval:(NSTimeInterval)ti;
[NSThread exit]; // 销毁线程：线程执行完毕之后就会自动消亡

4> 设置线程的名字
- (void)setName:(NSString *)n;
- (NSString *)name;

5> 再当前线程中执行某一个方法，相当于直接执行某一个方法
[self performSelector:@selector(test:) withObject:@"哈哈"];
```

### 线程同步

1.实质：为了防止多个线程抢夺同一个资源造成的数据安全问题
2.实现：给代码加一个互斥锁（同步锁）
```objc
@synchronized(self) { // 一定要使用同一把锁，这样才能全局的把控,其他线程也能被锁住
// 被锁住的代码
}
```

```objc
// 回到主线程，线程之间的通信
[self performSelectorOnMainThread:@selector(downloadFinished:) withObject:img waitUntilDone:YES];
```

### GCD

**GCD要点**
*  两个核心概念：任务(执行什么操作,用block 封装任务) 和 队列:(用来存放任务)
*  将任务添加到队列中
*  GCD会自动将队列中的任务取出，放到对应的线程中执行
*  任务的取出遵循队列的FIFO原则：先进先出，后进后出
*  dispatch_async: 异步的方式执行任务, 具备开启线程的能力(一般用来开启子线程)
*  dispatch_sync :同步的方式执行任务 (不具备开启线程的能力)
*  并发队列： Concurrent
>  可以让多个任务并发（同时）执行（自动开启多个线程同时执行任务）
>  并发功能只有在异步（dispatch_async）函数下才有效

*  串行队列(Serial)：让任务一个接着一个地执行（一个任务执行完毕后，再执行下一个任务）
*  容易混淆概念
>  同步和异步主要影响：能不能开启新的线程
>  同步：在当前线程中执行任务，不具备开启新线程的能力
>  异步：在新的线程中执行任务，具备开启新线程的能力

>  并发和串行主要影响：任务的执行方式
>  并发：多个任务并发（同时）执行
>  串行：一个任务执行完毕后，再执行下一个任务







