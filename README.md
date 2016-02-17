# NetWork
多线程网络Tips

## 多线程

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

### 网络

```
一、一个HTTP请求的基本要素

1.请求URL：客户端通过哪个路径找到服务器
* URL的基本格式 = 协议头://主机地址/路径

2.请求参数：客户端发送给服务器的数据
* 比如登录时需要发送的用户名和密码

3.返回结果：服务器返回给客户端的数据
* 一般是JSON数据或者XML数据

二、基本的HTTP请求的步骤（移动客户端）

1.拼接"请求URL" + "?" + "请求参数"
* 请求参数的格式：参数名=参数值
* 多个请求参数之间用&隔开：参数名1=参数值1&参数名2=参数值2
* 比如：http://localhost:8080/Server/login?username=123&pwd=456

2.发送请求

3.解析服务器返回的数据

**NSURLConnection**
* 同步请求，不常用 
[NSURLConnection sendSynchronousRequest:request returningResponse:nil error:nil];

1.发布异步请求--block回调

+ (void)sendAsynchronousRequest:(NSURLRequest*) request
queue:(NSOperationQueue*) queue
completionHandler:(void (^)(NSURLResponse* response, NSData* data, NSError* connectionError)) handler
* request : 需要发送的请求
* queue : 一般用主队列，存放handler这个任务
* handler : 当请求完毕后，会自动调用这个block

2.利用NSURLConnection发送请求的基本步骤
1> 创建URL
NSURL *url = [NSURL URLWithString:@"http://4234324/5345345"];
2> 创建request
NSURLRequest *request = [NSURLRequest requestWithURL:url];
3> 发送请求
[NSURLConnection sendAsynchronousRequest:request queue:queue completionHandler:
^(NSURLResponse *response, NSData *data, NSError *connectionError) {
4> 处理服务器返回的数据
}];

3. NSURLConnection代理的方法
[[NSURLConnection alloc]initWithRequest:request delegate:self]; //有返回值,但是自动开启用不到，只有当startImmediately:NO 才需要拿到实例变量手动开启
#pragma mark - delegate
/** 发送异步网络请求之后，请求失败之后会调用(请求超时，没有网络)  */
- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error

/** 开始收到服务器的响应时候调用*/
-(void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response

/** 受到服务器返回的数据时候调用(如果数据很大的话，这个方法会调用多次，一般都在这个方法里面拼接data)  */
-(void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data

/** 请求结束的时候调用 (一般再这个方法里面 做一些数据的处理) */
-(void)connectionDidFinishLoading:(NSURLConnection *)connection

```

**JSON解析**

```
1.利用NSJSONSerialization类解析 -> 官方的，性能较好
* JSON数据（NSData） --> Foundation-OC对象（NSDictionary、NSArray、NSString、NSNumber）

> NSDictionary *dataDict = [NSJSONSerialization JSONObjectWithData:self.data options:NSJSONReadingMutableLeaves error:&err];

> NSData *data = [NSJSONSerialization dataWithJSONObject:dataDict options:NSJSONReadingMutableLeaves error:nil]

2.JSON解析规律
* { } --> NSDictionary @{ }
* [ ] --> NSArray @[ ]
* " " --> NSString @" "
* 10 --> NSNumber @10
```

**XML解析**

```
1.语法
1> 文档声明
<?xml version="1.0" encoding="UTF-8" ?>

2> 元素
3> 属性
<videos>
    <video name="小黄人 第01部" length="10"/>
    <video name="小黄人 第01部" length="10"/>
</videos>

* videos和video是元素（节点）
* name和length叫做元素的属性
* video元素是videos元素的子元素

2.解析 : 一般抽取一个工具类来做解析的功能
```

**SAX解析：逐个元素往下解析，适合大文件**

```objc
//1.创建解析器
NSXMLParser *parser = [[NSXMLParser alloc]initWithData:data];
//2.设置代理: 把解析的结果告诉代理，实际上是代理监听解析的风吹草动
parser.delegate = self;
//3.开始解析(事件驱动)
[parser parse]; // （同步执行:再主线程中执行，所以不能搞太大的文件，不然会阻塞主线程）

/*** 代理 ***/
#pragma mark - NSXMLParserDelegate
/** 开始解析到文档的时候调用 ：解析开始 */
- (void)parserDidStartDocument:(NSXMLParser *)parser

/**
*  开始解析到元素的时候调用
*
*  @param parser        解析器对象
*  @param elementName   元素的名称(这一节点元素的名称)
*  @param attributeDict 属性字典 (这一节点内属性的 字典,注意，只是解析到的当前节点内的字典)
*/
- (void)parser:(NSXMLParser *)parser didStartElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName attributes:(NSDictionary *)attributeDict{
    // 因为节点有多个，所以要判断一下，只需要解析模型对应的节点
    if ([elementName isEqualToString:@"videos"]) return; // 直接返回。不需要创建video 模型

    // 能来到这里那么就是video 对应的节点
    JNVideo *video = [JNVideo videoWithDict:attributeDict];
    [self.tempArray addObject:video];
}

/** 结束解析到元素的时候调用 */
- (void)parser:(NSXMLParser *)parser didEndElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName

/** 结束解析到文档的时候调用 ：解析结束 */
-(void)parserDidEndDocument:(NSXMLParser *)parser
```

**DOM解析：一口气将整个XML文档加载进内存，适合小文件，使用最简单**

```objc
if (data) { // Dom 解析
    //  获取整个XML文档 (dom==》document)
    GDataXMLDocument *doc = [[GDataXMLDocument alloc]initWithData:data options:0 error:nil];
    NSMutableArray *tempArray = [NSMutableArray array]; // 装对象
    // 获取根元素-- videos
    GDataXMLElement *rootElem = [doc rootElement];
    // 获取子元素 video
    NSArray *elements = [rootElem elementsForName:@"video"];
    // 获取子元素的属性，赋值给模型
    for (GDataXMLElement *element in elements) {
        JNVideo *video = [[JNVideo alloc]init];
        video.ID = [element attributeForName:@"id"].stringValue.intValue;
        video.name = [element attributeForName:@"name"].stringValue;
        video.image = [element attributeForName:@"image"].stringValue;
        video.url = [element attributeForName:@"url"].stringValue;
        video.length = [element attributeForName:@"length"].stringValue.intValue;
        [tempArray addObject:video];
    }
}
```

**HTTP的通信过程**

```
1.请求
* 请求行 : 请求方法、请求路径、HTTP协议的版本
GET /MJServer/resources/images/1.jpg HTTP/1.1

* 请求头 : 对客户端的环境描述、客户端请求的主机地址
User-Agent // 客户端的类型,客户端的软件环境
Host: 192.168.1.105:8080 // 客户端想访问的服务器主机地址
Accept: text/html,  // 客户端所能接收的数据类型
Accept-Language: zh-cn // 客户端的语⾔言环境
Accept-Encoding: gzip // 客户端⽀支持的数据压缩格式

* 请求体 : POST请求才有请求体
请求参数，发给服务器的数据 客户端发给服务器的具体数据,⽐如文件数据

// -->响应
2.响应
* 状态行（响应行）: HTTP协议的版本、响应状态码、响应状态描述
HTTP/1.1 200 OK

* 响应头：服务器的一些描述信息
Content-Type : 服务器返回给客户端的内容类型
Content-Length : 服务器返回给客户端的内容的长度（比如文件的大小）

* 实体内容（响应体）
服务器返回给客户端具体的数据，比如文件数据
```

**HTTP的请求方法**
```
1.GET
1> 特点
* 所有请求参数都拼接在url后面

2> 缺点
* 在url中暴露了所有的请求数据，不太安全
* url的长度有限制，不能发送太多的参数

3> 使用场合
* 如果仅仅是向服务器索要数据，一般用GET请求

4> 如何发送一个GET请求
* 默认就是GET请求
// 1.URL
NSURL *url = [NSURL URLWithString:@"http://www.baidu.com"];
// 2.请求
NSURLRequest *request = [NSURLRequest requestWithURL:url];
// 3.发送请求,异步
[NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {
}];

2.POST
* 特点
把所有请求参数放在请求体（HTTPBody）中
理论上讲，发给服务器的数据的大小是没有限制

*使用场合
除开向服务器索要数据以外的请求，都可以用POST请求
如果发给服务器的数据是一些隐私、敏感的数据，绝对要用POST请求
```

如何发送一个POST请求
```objc
// 1.创建一个URL ： 请求路径
NSURL *url = [NSURL URLWithString:@"http://localhost:8080/Server/login"];
// 2.创建一个请求
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
// 设置请求方法
request.HTTPMethod = @"POST";
// 设置请求体 : 请求参数
NSString *param = [NSString stringWithFormat:@"username=%@&pwd=%@", usernameText, pwdText];
// NSString --> NSData
request.HTTPBody = [param dataUsingEncoding:NSUTF8StringEncoding];

// 发送请求
[NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {

    // 真实类型是 NSHTTPURLResponse ，内部获得状态码，请求头信息
    NSHTTPURLResponse *resp =(NSHTTPURLResponse *)response;
    // 打印响应头
    NSLog(@"---%@",resp.allHeaderFields);
    //     NSURLResponse *respons 这个对象非常重要
    //     发给服务器的参数全部放在请求体中
    //     设置请求头的信息(客户端的环境) 
    [ setValue: forHTTPHeaderField:@"User-Agent"];
}];
```

```
URL转码
URL中不能包含中文，得对中文进行转码(加上一堆的%)
NSString *urlStr = [NSString stringWithFormat:@"http://localhost/login?username=喝喝&pwd=123"];
urlStr = [urlStr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
```


```
数据安全
1.网络数据加密
* 加密对象：隐私数据，比如密码、银行信息
* 加密方案
提交隐私数据，必须用POST请求
使用加密算法对隐私数据进行加密，比如MD5
* 加密增强：为了加大破解的难度
对明文进行2次MD5 ： MD5(MD5($pass))
先对明文撒盐，再进行MD5 ： MD5($pass.$salt)

2.本地存储加密
* 加密对象：重要的数据，比如游戏数据

3.代码安全问题
* 现在已经有工具和技术能反编译出源代码：逆向工程
反编译出来的都是纯C语言的，可读性不高
最起码能知道源代码里面用的是哪些框架

参考书籍：《iOS逆向工程》

解决方案：发布之前对代码进行混淆
* 混淆之前
@interface HMPerson :NSObject
- (void)run;
- (void)eat;
@end

* 混淆之后
@interface A :NSObject
- (void)a;
- (void)b;
@end

```

**大文件下载**
1.方案：利用NSURLConnection和它的代理方法

```objc
NSURL *url = [NSURL URLWithString:@"http://localhost:8080/Server/resources/videos.zip"];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
// 下载(创建完conn对象后，会自动发起一个异步请求)
[NSURLConnection connectionWithRequest:request delegate:self];

/**
在接收到服务器的响应时：
1.创建一个空的文件
2.用一个句柄对象关联这个空的文件，目的是：方便后面用句柄对象往文件后面写数据
*/
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response{
    // 文件路径
    NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
    NSString *filepath = [caches stringByAppendingPathComponent:@"videos.zip"];

    // 创建一个空的文件 到 沙盒中
    NSFileManager *mgr = [NSFileManager defaultManager];
    [mgr createFileAtPath:filepath contents:nil attributes:nil];

    // 创建一个用来写数据的文件句柄
    self.writeHandle = [NSFileHandle fileHandleForWritingAtPath:filepath];
}

/**
在接收到服务器返回的文件数据时，利用句柄对象往文件的最后面追加数据
*/
- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data{
    // 移动到文件的最后面
    [self.writeHandle seekToEndOfFile];

    // 将数据写入沙盒
    [self.writeHandle writeData:data];
}

/**
在所有数据接收完毕时，关闭句柄对象
*/
- (void)connectionDidFinishLoading:(NSURLConnection *)connection{
    // 关闭文件
    [self.writeHandle closeFile];
    self.writeHandle = nil;
}

2.注意点：千万不能用NSMutableData来拼接服务器返回的数据
```

```
NSURLConnection发送异步请求的方法
1.block形式 - 除开大文件下载以外的操作，都可以用这种形式
[NSURLConnection sendAsynchronousRequest:req queue:queue completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {

}];

2.代理形式 - 一般用在大文件下载
```

三、NSURLSession
*  NSURLSession :[NSURLSession sharedSession] 单例
*  NSURLSessionDataTask :GET\POST
*  NSURLSessionDownloadTask : 文件下载
*  NSURLSessionUploadTask : 文件上传(不常用)
*  三步走：
1. 获取session 对象
2. 获取任务对象 (session 可以通过request，或者url 来发送请求)
3. [task resume]

*  finishTasksAndInvalidate 完成任务，关闭请求
*  invalidateAndCancel  取消请求


```objc
用途：用于非文件下载的GET\POST请求
NSURLSessionDataTask *task = [self.session dataTaskWithRequest:request];
NSURLSessionDataTask *task = [self.session dataTaskWithURL:url];
NSURLSessionDataTask *task = [self.session dataTaskWithURL:url completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {

}];

```

```objc
NSURLSessionDownloadTask
* 用途：用于文件下载（小文件、大文件）
NSURLSessionDownloadTask *task = [self.session downloadTaskWithRequest:request];
NSURLSessionDownloadTask *task = [self.session downloadTaskWithURL:url];
NSURLSessionDownloadTask *task = [self.session downloadTaskWithURL:url completionHandler:^(NSURL *location, NSURLResponse *response, NSError *error) {
// 回调处理  利用url NSURLSessionDownloadTask 来下载数据 (再异步线程中,不会阻塞主线程)
// 文件下载在tmp 路径中: location.path
}];
[task resume]; // 恢复下载（一定要写，表示开始）
```

```objc
/** 临时文件的路径（下载好的文件）*/
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(NSURL *)location{
    // location : 临时文件的路径（下载好的文件）
    // response.suggestedFilename ： 建议使用的文件名，一般跟服务器端的文件名一致
    // 将临时文件剪切或者复制Caches文件夹

    // AtPath : 剪切前的文件路径
    // ToPath : 剪切后的文件路径
    [mgr moveItemAtPath:location.path toPath:file error:nil];
}

/**
*  恢复下载时调用
*/
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didResumeAtOffset:(int64_t)fileOffset expectedTotalBytes:(int64_t)expectedTotalBytes

/**
*  每当下载完（写完）一部分时就会调用（可能会被调用多次）
*  @param bytesWritten              这次调用写了多少
*  @param totalBytesWritten         累计写了多少长度到沙盒中了
*  @param totalBytesExpectedToWrite 文件的总长度
*/
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
{
    double progress = (double)totalBytesWritten / totalBytesExpectedToWrite;
}

```

```objc
AFN 文件上传

[mgr POST:@"http://localhost:8080/MJServer/upload" parameters:params constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {

    /** 这个block 专门用来上传文件的，调用的比下面两个早
    *  appendPartWithFileData: 需要上传的文件的二进制数据
    *  name: 服务器端文件参数名
    *  filename: 上传到服务器的名称
    *  mimeType: 文件类型
    */

    UIImage *img = [UIImage imageNamed:@"0"];
    NSData *data = UIImagePNGRepresentation(img);
        // 方式1
        [formData appendPartWithFileData:data name:@"file" fileName:@"mm.png" mimeType:@"image/png"];
        // 方式2
        [formData appendPartWithFileURL:url name:@"file" fileName:@"xixi.png" mimeType:@"image/png" error:nil];

        } success:^(AFHTTPRequestOperation *operation, id responseObject) {
        NSLog(@"-请求成功-%@",responseObject);
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
    NSLog(@"--请求失败-%@",error);
}];

```

```
// 加入插入数据库插入一万条数据你怎么搞？
1. 插入数据的操作放在子线程，这里注意：最好的情况是开一条子线程，并且最好是串行执行的任务.这样较好的保证数据的一致。 如果GCD玩的六，那么可以开多条线程。搞好读写

2. 利用事务
* 开启事务-> 备份
* 提交事务-> 删除备份
* 回滚-> 执行失败，用备份还原数据,回滚之后，就立马停止插入操作（跳出循环,没必要继续走下去）
注意： 执行一条sql 语句，都会频繁的开启事务关闭事务，这就是耗时的原因所在,在执行添加数据的前后开启提交数据，那么在插入数据的时候，内部就不会开启事务,这样保证了只开启一次事务，避免了开启一万条数据

3.预编译绑定：之前就准备好，直接给你 + 事务
```

