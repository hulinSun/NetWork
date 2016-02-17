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
*  cpu再时间片内的切换---.调度

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

2.创建完自动启动 --. 分离出子线程
```objc
[NSThread detachNewThreadSelector:@selector(download:) toTarget:self withObject:nil];
```

3.隐式创建（自动启动）,在后台线程(子线程中)执行
```objc
[self performSelectorInBackground:@selector(download:) withObject:nil];
```
**常见方法**
```
1 获得当前线程
+ (NSThread *)currentThread;

2 获得主线程
+ (NSThread *)mainThread;

3. 睡眠（暂停），退出线程
+ (void)sleepUntilDate:(NSDate *)date;
+ (void)sleepForTimeInterval:(NSTimeInterval)ti;
[NSThread exit]; // 销毁线程：线程执行完毕之后就会自动消亡

4. 设置线程的名字
- (void)setName:(NSString *)n;
- (NSString *)name;

5. 再当前线程中执行某一个方法，相当于直接执行某一个方法
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
.  可以让多个任务并发（同时）执行（自动开启多个线程同时执行任务）
.  并发功能只有在异步（dispatch_async）函数下才有效

*  串行队列(Serial)：让任务一个接着一个地执行（一个任务执行完毕后，再执行下一个任务）
*  容易混淆概念
.  同步和异步主要影响：能不能开启新的线程
.  同步：在当前线程中执行任务，不具备开启新线程的能力
.  异步：在新的线程中执行任务，具备开启新线程的能力

.  并发和串行主要影响：任务的执行方式
.  并发：多个任务并发（同时）执行
.  串行：一个任务执行完毕后，再执行下一个任务

**GCD要点**
* GCD是苹果公司为多核的并行运算提出的解决⽅方案
* GCD会⾃动利用更多的CPU内核(⽐比如双核、四核)
* GCD会⾃动管理线程的⽣命周期(创建线程、调度任务、销毁线程)
* 程序员只需要告诉GCD想要执⾏什么任务,不需要编写任何线程管理代码

**概念**
1.队列和任务
1. 任务 ：需要执行什么操作
* 用block来封装任务

2. 队列 ：存放任务
* 全局的并发队列 ： 可以让任务并发执行
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

* 自己创建的串行队列 ： 让任务一个接着一个执行
dispatch_queue_t queue = dispatch_queue_create("queue", NULL);

* 主队列 ： 让任务在主线程执行
dispatch_queue_t queue = dispatch_get_main_queue();

2.执行任务的函数
1. 同步执行 : 不具备开启新线程的能力
dispatch_sync...

2. 异步执行 : 具备开启新线程的能力
dispatch_async...

3.常见的组合
1. dispatch_async + 全局并发队列 -. 会开启多条线程，并且开启的线程个数有系统决定
2. dispatch_async + 自己创建的串行队列 -. 会开启一条子线程

4.线程间的通信
```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
// 执行耗时的异步操作...
dispatch_async(dispatch_get_main_queue(), ^{
// 回到主线程，执行UI刷新操作
});
});
```
5.GCD的所有API都在libdispatch.dylib，Xcode会自动导入这个库
* 主头文件 ： #import <dispatch/dispatch.h.
6.延迟执行
```3秒后自动回到当前线程调用self的download:方法，并且传递参数：@"http://555.jpg"
[self performSelector:@selector(download:) withObject:@"http://555.jpg" afterDelay:3];
```
```objc
// 任务放到哪个队列中执行
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
double delay = 3; // 延迟多少秒
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delay * NSEC_PER_SEC)), queue, ^{
// 3秒后需要执行的任务 : 会一直往下走，并不会卡住当前线程，只是相当于一个回调
});
```
7.一次性代码
```objc
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
// 这里面的代码，在程序运行过程中，永远只会执行1次
});
```

####单例模式
```objc
+ (id)allocWithZone:(struct _NSZone *)zone{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instace = [super allocWithZone:zone];
    });
    return _instace;
}

+ (instancetype)sharedDataTool{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instace = [[self alloc] init];
    });
    return _instace;
}

- (id)copyWithZone:(NSZone *)zone{
    return _instace;
}

```

**非ARC**
```objc
// 用来保存唯一的单例对象
static id _instace;

+ (id)allocWithZone:(struct _NSZone *)zone{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instace = [super allocWithZone:zone];
    });
    return _instace;
}

+ (instancetype)sharedDataTool{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instace = [[self alloc] init];
    });
    return _instace;
}

- (id)copyWithZone:(NSZone *)zone{
    return _instace;
}

- (oneway void)release {
// 保证不让释放
}

- (id)retain {
return self;
}

- (NSUInteger)retainCount {
return 1;
}

- (id)autorelease {
return self;
}

```
**队列组**
*  队列组 创建队列组 ：dispatch_group_create();
*  dispatch_group_async 将任务放在队列中，将队列放在队列组中，
*   dispatch_group_notify，唤醒队列。 队列组中的任务都做好了会自动来到这个方法。做一些操作

需求：开两条子线程下载图片，并且等两张图片下完之后打水印。(因为时异步，所以不确定两张图片什么时候下载完毕，但是需要保证两张图片都下载完毕才做操作)

```objc
// 1.获取全局的并发队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
// 2.创建队列组
dispatch_group_t group = dispatch_group_create();
__block UIImage *img1 = nil;
// 3.1将任务放在队列里面，再将队列放在队列组里面
dispatch_group_async(group, queue, ^{
NSString *urlStr = @"https://www.baidu.com/img/bd_logo1.png";
NSURL *url = [NSURL URLWithString:urlStr];
NSData *data = [NSData dataWithContentsOfURL:url];
// 再外面修改内部的值，加上__
img1 = [UIImage imageWithData:data];
});
__block UIImage *img2 = nil;
// 3.2将任务放在队列里面，再将队列放在队列组里面
dispatch_group_async(group, queue, ^{
    NSString *urlStr = @"http://a.hiphotos.baidu.jpg";
    NSURL *url = [NSURL URLWithString:urlStr];
    NSData *data = [NSData dataWithContentsOfURL:url];
    img2 = [UIImage imageWithData:data];
});

// 4.唤醒队列.当队列中的任务都执行完毕之后，会自动来到这个方法，做下一步的操作
dispatch_group_notify(group, queue, ^{

// 两张图片打水印操作。。。

// 回到主线程
dispatch_async(dispatch_get_main_queue(), ^{
    self.imgView.image = newImg;
    });
});

```
#### NSOperation和NSOperationQueue

*  NSOperation 是抽象类，需要用子类来实现一些功能 (NSInvocationOperation 几乎不用initWithTarget)
*  NSBlockOperation 用block 封装操作，如果封装操作的个数大于一的话(addExecutionBlock) ,那么这个操作会自动开启异步线程来执行操作
*  需要调用start 方法开启 NSOperation可以调用start方法来执行任务，但默认是同步执行的
*  不管是什么操作，只要将操作放在队列中，默认是异步线程

```
1.队列的类型
// blockOperation
blockOperation.completionBlock = ^{ // 监听block 执行完毕的回调要放在开启之前，否则不会调用
    NSLog(@"completionBlock 操作执行完毕----");
};
[blockOperation start]; // 开始
```

```
1. 主队列
* [NSOperationQueue mainQueue]
* 添加到"主队列"中的操作，都会放到主线程中执行

2. 非主队列
* [[NSOperationQueue alloc] init]
* 添加到"非主队列"中的操作，都会放到子线程中执行

2.队列添加任务(队列的方法)
* - (void)addOperation:(NSOperation *)op;
* - (void)addOperationWithBlock:(void (^)(void))block;

3.常见用法
设置最大并发数
- (NSInteger)maxConcurrentOperationCount;
- (void)setMaxConcurrentOperationCount:(NSInteger)cnt;

队列的其他操作
* 取消所有的操作
- (void)cancelAllOperations;

* 暂停所有的操作
[queue setSuspended:YES];

* 恢复所有的操作
[queue setSuspended:NO];

4.操作之间的依赖
* NSOperation之间可以设置依赖来保证执行顺序
[block2 addDependency:block1]; // 2 依赖于1 ，意味着，1 执行完在执行2
* [operationB addDependency:operationA];
// 操作B依赖于操作A，等操作A执行完毕后，才会执行操作B
* 注意：不能相互依赖，比如A依赖B，B依赖A
* 可以在不同queue的NSOperation之间创建依赖关系

```

**5.线程之间的通信**
```objc
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
[queue addOperationWithBlock:^{
// 1.执行一些比较耗时的操作
    // 2.回到主线程
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
    // 主线程 处理点击事件 刷新UI等
    }];
}];

```
#### 图片下载原理

*  一个图片字典存取图片,key:图片下载地址 value:图片
*  一个图片字典存取操作， 图片下载的操作 key：url value ：下载操作 --.保证一对一
*  先从缓存字典里面取图片
*  有--.显示，没有--. 显示占位图片
*  没有缓存图片的情况
*  取出是否有下载操作，有：什么也不做，让他下载 ，没有，那么开启一条操作下载
*  下载好，保存图片 刷新表格
*  存操作

下载好图片就刷新这一行，为什么不通过cell.imageView.image 方法设置图片呢？ 因为当下载的过程中，用户偶有滚动，那么正在下载的cell会在缓存池中，给别的cell 用的话，会发生下载两张图片的效果，而下载好立马刷新，（存图片的操作的刷新前面）那么就会被第一宠img循环拦截，所以保证了只下载一次图片

```objc
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    static NSString *ID = @"cell";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];
    if (!cell) {
    cell = [[UITableViewCell alloc]initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:ID];
    }
    // 取出模型
    JNApp *app = self.apps[indexPath.row];
    cell.textLabel.text = app.name;
        // 先从缓存池中取
        UIImage *img = self.imgs[app.icon];
        if (img) { // 有图片，已经下载好了图片
        cell.imageView.image = img;
        }else{ // 没有图片，没有下载
        // 显示占位图片
        cell.imageView.image = [UIImage imageNamed:@"placeholder"];
        // 取出操作
        NSBlockOperation *opera = self.operations[app.icon];
        if (opera) { // 有操作，正在下载
            // 正在下载什么也不做
                }else{ // 没有下载，那么开启下载
                opera = [NSBlockOperation blockOperationWithBlock:^{
                NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:app.icon]];
                UIImage *downloadImg = [UIImage imageWithData:data];
            // 回到主线程
                [[NSOperationQueue mainQueue]addOperationWithBlock:^{
            // 下载好了，存图片
                if (downloadImg) { // 有图片才存
                    self.imgs[app.icon] = downloadImg;
                }
        // 刷新表格
        [self.tableView reloadRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationNone];
        // 删除操作-.防止操作太多,注意，这是主线程，意味着已经刷新过图片了.方便下载不成功的时候下次再下载
        [self.operations removeObjectForKey:app.icon];
    }];
}];
        // 异步执行
        [self.queue addOperation:opera];
        // 存操作 // 添加到字典中 (这句代码为了解决重复下载)
        self.operations[app.icon] = opera;
        }
    }
    return cell;
}

```

**自定义Operation**

```objc
- (void)main
{
@autoreleasepool { // 释放掉子线程中的一些变量(固定写法)
    // 因为是自定义operation ，那么线程都是自己掌控,时刻监听操作是否被取消
    if (self.isCancelled) return; // 开始前判断一下
    // 异步下载操作
    NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:self.imgUrl]];
    UIImage *img = [UIImage imageWithData:data];

    if (self.isCancelled) return; // 下载完之后判断一下(防止收到内存警告之后还是通知代理)

    // 下载完毕，通知代理，通过代理来交互
    if ([self.delegate respondsToSelector:@selector(downloadOperation:didFinishDownloadImage:)]) {
        [self.delegate downloadOperation:self didFinishDownloadImage:img];
        }
    }
}

```

```
判断编译器的环境：ARC还是MRC？
#if __has_feature(objc_arc)
// 当前的编译器环境是ARC
#else
// 当前的编译器环境是MRC
#endif
```

```
类的初始化方法
1.+(void)load
* 当某个类第一次装载到OC运行时系统（内存）时，就会调用
* 程序一启动就会调用
* 程序运行过程中，只会调用1次

2.+(void)initialize
* 当某个类第一次被使用时（比如调用了类的某个方法），就会调用
* 并非程序一启动就会调用

3.在程序运行过程中：1个类中的某个操作，只想执行1次，那么这个操作放到+(void)load方法中最合适
```

**SDWebImage**

```objc
1.常用方法
- (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder;
- (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options;
- (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder completed:(SDWebImageCompletionBlock)completedBlock;
- (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options progress:(SDWebImageDownloaderProgressBlock)progressBlock completed:(SDWebImageCompletionBlock)completedBlock;
```
2. 当app接收到内存警告
```objc
- (void)applicationDidReceiveMemoryWarning:(UIApplication *)application{
    SDWebImageManager *mgr = [SDWebImageManager sharedManager];
    // 1.取消正在下载的操作
    [mgr cancelAll];
    // 2.清除内存缓存
    [mgr.imageCache clearMemory];
}

```

```
SDWebImageOptions
* SDWebImageRetryFailed : 下载失败后，会自动重新下载
* SDWebImageLowPriority : 当正在进行UI交互时，自动暂停内部的一些下载操作
* SDWebImageRetryFailed | SDWebImageLowPriority : 拥有上面2个功能
```

