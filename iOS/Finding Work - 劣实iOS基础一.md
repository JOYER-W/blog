# Finding Work - 劣实iOS基础

## **iOS内存管理**

在Objective-C中有两套内存管理策略，MRC(Mannul Reference Counting)和ARC(Automatic Referenct Counting)，但是两套都是基于引用计数来进行内存管理的。
	
MRC遵循着谁创建谁释放，一般会有如下情况，创建一个新对象的方法：`new`,`alloc`,`copy`,`mutableCopy`,此时的引用计数都是retain count都是为1。此时当进行了如下操作，新对象会获取原来对象的所有权，引用计数就会增加，`id newObj = [oldObj retain]`, 所以，针对如上的操作都需要手动调用`release`。如下，`[newObj release]`.  
	
ARC是自iOS5之后推出的新的内存管理策略，一种自动引用计数技术。其管理内存的策略和MRC一致，仅仅只是编译器在特定的位置帮我们加上`release`代码。
> Automatic Reference Counting (ARC) is a compiler feature that provides automatic memory management of Objective-C objects. Rather than having to think about retain and release operations, ARC allows you to concentrate on the interesting code.

本人也是从iOS7开始接触iOS开发的。所以一直以来都是使用ARC内存管理规则。
> ARC works by adding code at compile time to ensure that objects live as long as necessary, but no longer.

ARC的解释是在编译时，编译器会在对象的生命周期中添加必要的代码(`release`代码)。即，现在不需要我们手动的调用`release`

* **iOS 中可能的内存泄漏**
	* 循环引用  
		只要是使用了的是引用计数策略进行内存管理的，就会有出现循环引用的可能性，导致内存无法释放掉而内存泄漏。
		1. 对象间的循环引用	，使用弱引用来破解。如下，分别有objA，objB，objC，三个对象，属性间相互引用，造成了循环引用。此时可以设置对象objC指向对象objA的属性为弱引用类型。
		
				objA.property = objB;
				objB.property = objC;
				objC.property = objA;
		
			
		2. 对象和Block之间的循环引用
				
				self.someBlock = ^{ self.property = xxx; };
			如上出现了循环引用，此时编译器可以检测出，直接会报错。但是当有其他对象介入的时候时候，编译器就不能在编译期发现是否出现了循环引用。  
				
				someObj.someBlock = ^{ self.property = xxx;};
				self.someObj = someObj;
			
			* **解决方案**
			
			使用`__weak`修饰，弱化block中对象引用。如下
			
				__weak SomeObjectClass *weakSelf = self;
				someObj.someBlock = ^{
					SomeObjectClass *strongSelf = weakSelf;
					if (strongSelf == nil) {
						// handle 
					}
					[strongSelf sendMsg];
				};
		
		3. 	NSTimer 
			
			在OC中有很多的`Action/Target`的模式，大多数的情况下都是弱引用了`target`，但是在NSTimer中并不是这样的，`timer`对象强引用了`target`
			
				+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)seconds
                        					target:(id)target
					                      selector:(SEL)aSelector
                      					  userInfo:(id)userInfo
                       					   repeats:(BOOL)repeats
			> 解释target : The object to which to send the message specified by aSelector when the timer fires. The timer maintains a strong reference to this object until it (the timer) is invalidated.

## **UITableView的优化**

`UITableView`对于iOS开发而言，不能再熟悉了，这也是iOS和Android流畅性最为直观的地方。所以做好`UITableView`的优化也是至关重要的。（😄，其实只要在开发中保持一个好习惯就行）

**概念**  
`UITableView`最为关键的就是几个代理方法，一个是返回cell的`- tableView:cellForRowAtIndexPath:`，一个是返回高度的`- tableView:heightForRowAtIndexPath:`。由于`UITableView`是继承于`UIScrollView`，所以必须优先计算出`contentSize`，所有会先调用`- tableView:heightForRowAtIndexPath:`进行整体布局，之后调用返回cell的方法，进行cell内部的布局。

所以： 

1. 返回高度的方法`- tableView:heightForRowAtIndexPath:`调用次数最多。
2. 返回cell的方法`- tableView:cellForRowAtIndexPath:`最耗时。

优化：

1. 在获取到数据后，把cell的高度提前计算好，存储在model(或者其他地方)中，在`- tableView:heightForRowAtIndexPath:`回调中直接使用。
2. 对cell中不要添加太多的subView。更多的subView需要更多的布局等其他方面的计算。
3. 对于要求很高的tableView，cell不要使用xib，毕竟需要对xib进行转化。
4. 可以用去绘画一个cell，不用系统自带的view.(对tableView滑动要求更高)
5. 使用动态按需加载，即只有当tableView快速滑动结束后再加载可视范围内的cell

tableView其他一些常用的正确的用法：

1. 正确使用重用机制
2. cell中尽量不要使用alpha值不为1的view，不然会造成图层混合
3. cell中不要大量设置圆角，避免出现离屏渲染。需要圆角可以自己绘制。
4. cell中图层尽量不设置阴影，必要时可以使用栅格化技术，和自行绘制阴影路径，防止由于阴影照成的离屏渲染。
5. 在cell中最好不要动态的创建subview，最好在初始化的时候创建好subview，之后使用属性`hide`来控制显示和隐藏。
6. cell中的需要加载的东西来自于web，使用异步加载的方式。

具体的参照	 
**圆角绘制：** [iOS - 圆角的设计策略 - 及绘画出没有离屏渲染的圆角](http://www.jianshu.com/p/af70feefbc89)  
**图层调试：** [记-UI性能调试及调优](http://www.jianshu.com/p/3656ef4143da)  
**UITableView优化：**[UITableView优化技巧](http://longxdragon.github.io/2015/05/26/UITableView%E4%BC%98%E5%8C%96%E6%8A%80%E5%B7%A7/)
	
## **UDP和TCP**
	
TCP和UDP是计算机网络中学习到的概念，它们两者都属于运输层。  
	
TCP(Transmission Control Protocol)是一种**面相连接，可靠的流协议**，其中连接的意思指的是两个应用为了通信，在两者之间建立的一种虚拟的线路。可靠的流协议指的是不间断数据结构，就比如管道中的水流，TCP提供了如重发机制，流量控制等一系列的可靠性保障。虽然有TCP有很多优点，也有不足的地方，就是传输效率比较低。	
	
UDP(User Datagram Protocol)是一种**不具有可靠性的数据报协议**，只负责发送信息，但是不确定对方是否收到。但是优点就是效率非常高。
	
使用场景：场景都是基于其优缺点。由于TCP传输可靠性很强，所以在传输文件，收发邮件等。UDP效率高，可靠性比较低，一般用于网络电话，视频，语音等，必须是实时的，其中可能有一点点断断续续，可能影响不会很大。

## **SDWebImage的原理** 

**SDWebImage作为开源第三方框架中的姣姣者，经常被我们的项目所用到，看似很复杂，其实实现的原理中并不算很复杂——总的来说  
1. 入口setImageWithURL:placeholderImage:options:开始  
2. 先从内存缓存中查找，是否存在图片，存在，显示图片。  
3. 不存在，开启一个新线程从磁盘缓存中查找，查看是否存在图片，存在，回调到主线程，显示图片。  
4. 不存在，开始一个新的线程下载图片。下载成功，对图片进行解码，解码成功后，回调到主线程，显示图片。并且缓存图片到内存缓存和磁盘缓存中。  
具体的详细步骤：如下**
> 1. UIImageView+WebCache:  `setImageWithURL:placeholderImage:options:` 先显示 placeholderImage ，同时由SDWebImageManager 根据 URL 来在本地查找图片。

> 2. SDWebImageManager: `downloadWithURL:delegate:options:userInfo:` SDWebImageManager是将UIImageView+WebCache同SDImageCache链接起来的类， SDImageCache： `queryDiskCacheForKey:delegate:userInfo:`用来从缓存根据CacheKey查找图片是否已经在缓存中

> 3. 如果内存中已经有图片缓存， SDWebImageManager会回调SDImageCacheDelegate : `imageCache:didFindImage:forKey:userInfo:`

> 4. 而 UIImageView+WebCache 则回调SDWebImageManagerDelegate:  `webImageManager:didFinishWithImage:`来显示图片。

> 5. 如果内存中没有图片缓存，那么生成 NSInvocationOperation 添加到队列，从硬盘查找图片是否已被下载缓存。

>6. 根据 URLKey 在硬盘缓存目录下尝试读取图片文件。这一步是在 NSOperation 进行的操作，所以回主线程进行结果回调 notifyDelegate:。

>7. 如果上一操作从硬盘读取到了图片，将图片添加到内存缓存中（如果空闲内存过小，会先清空内存缓存）。SDImageCacheDelegate 回调 `imageCache:didFindImage:forKey:userInfo:`。进而回调展示图片。

>8. 如果从硬盘缓存目录读取不到图片，说明所有缓存都不存在该图片，需要下载图片，回调 `imageCache:didNotFindImageForKey:userInfo:`。

>9. 共享或重新生成一个下载器 SDWebImageDownloader 开始下载图片。

>10. 图片下载由 NSURLConnection 来做，实现相关 delegate 来判断图片下载中、下载完成和下载失败。

>11. connection:didReceiveData: 中利用 ImageIO 做了按图片下载进度加载效果。

>12. connectionDidFinishLoading: 数据下载完成后交给 SDWebImageDecoder 做图片解码处理。
>13. 图片解码处理在一个 NSOperationQueue 完成，不会拖慢主线程 UI。如果有需要对下载的图片进行二次处理，最好也在这里完成，效率会好很多。

>14. 在主线程 notifyDelegateOnMainThreadWithInfo: 宣告解码完成，`imageDecoder:didFinishDecodingImage:userInfo:` 回调给 SDWebImageDownloader。

>15. `imageDownloader:didFinishWithImage:` 回调给 SDWebImageManager 告知图片下载完成。

>16. 通知所有的 downloadDelegates 下载完成，回调给需要的地方展示图片。

>17. 将图片保存到 SDImageCache 中，内存缓存和硬盘缓存同时保存。

>18. 写文件到硬盘在单独 NSInvocationOperation 中完成，避免拖慢主线程。

>19. 如果是在iOS上运行，SDImageCache 在初始化的时候会注册notification 到 UIApplicationDidReceiveMemoryWarningNotification 以及  UIApplicationWillTerminateNotification,在内存警告的时候清理内存图片缓存，应用结束的时候清理过期图片。

>20. SDWebImagePrefetcher 可以预先下载图片，方便后续使用。

其他一些有关**SDWebImage**的资料可以参照
[SDWebImage源码解析之SDWebImageManager的注解](http://www.jianshu.com/p/6ae6f99b6c4c)

## **什么时候会造成内存泄漏**

1. 对于在使用MRC内存管理策略的年代，你创建了对象，或者`retain`了对象后，并没有对对象做`release`操作，会造成内存泄漏		
(以下针对包括ARC和MRC两种内存管理策略)
2. 对象之间的循环引用
3. 对象和Block的循环引用
4. 使用KVC和KVO，对象添加了观察者，但是对象没有在`- dealloc`方法中移除观察者
5. 使用通知的时候，没有移除观察者 
6. `performSelector`可能导致内存泄漏，当所执行的`selector`有返回值，必须对返回值重新释放。
7. `NSTimer`，定时器会不断的给`target`发送消息，即`timer`持有了`target`，当没有调用`[timer invalidate]`，`target`就不会被释放掉。造成内存泄漏。
8. @try...@catch中，当抛出了移除，移除对象则不能被释放。

**某些点可以参照**[ARC下内存泄露的那些点](https://www.zybuluo.com/MicroCai/note/67734)

## **自动释放池的原理**

在iOS应用开始的main方法中，就创建了全局`autoreleasepool`，之后所有的对象都存在于该自动释放池中。当然，我们可以在代码的局部创建局部的`autoreleasepool`，使用`@autoreleasepool{// your code}`

	int main(int argc, char * argv[]) {
	    @autoreleasepool {
	        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
	    }
	}

在最开始的理解中，认为对象的释放是在`autoreleasepool`的大括号({})结束的时候释放了对象。其实并不是这样的。在自动释放池真正释放对象是在当前的`runloop`迭代结束的时候，它能够真正释放的原因是由于---**系统在每个runloop迭代中都加入了自动释放池Push和Pop**

### Autorelease 原理 － AutoreleasePoolPage

在ARC中，自己写一个`@autoreleasepool{}`会被翻译成

	void *context = objc_autoreleasePoolPush();
	// {}中的代码
	objc_autoreleasePoolPop(context);

![AutoreleasePoolPage](http://upload-images.jianshu.io/upload_images/1626952-db2b2f5ce8d22dd4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> * AutoreleasePool没有特定的数据结构，它是由一个或多个AutoreleasePoolPage结构构成的，它的组织方式是双向链表，体现在结构体中的指针parent和指针child。
> * 每个`AutoreleasePoolPage`都有对应特定的线程，其中的thread指向的是当前线程。
> * 指针`id *next`指向的是当前`AutoreleasePool`新增进来的autorelease对象的下一个位置。
> * AutoreleasePoolPage每个对象会开辟4096字节内存（也就是虚拟内存一页的大小），除了上面的实例变量所占空间，剩下的空间全部用来储存`autorelease`对象的地址。
> * 当当前的`AutoreleasePoolPage`空间大小不足时，就会新创建一个`AutoreleasePoolPage`。

假设当前的线程中只有一个`AutoreleasePoolPage`，并且已经快存满了`autorelease`对象，如下图:
![将要存满的AutoreleasePoolPage](http://upload-images.jianshu.io/upload_images/1626952-319e46fcde7ba29b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图的`next`指针下一个位置就是栈顶，当加入一个新的`autorelease`对象时，会创建一个新的`AutoreleasePoolPage`并用指针`child`连接新的`AutoreleasePoolPage`，完成连接后，新的`AutoreleasePoolPage`next指针会初始化栈底(begin)位置，新的`autorelease`对象又可以添加了，😄.

### Autorelease 原理 － 释放

当调用`objc_autoreleasePoolPush`时，会向`AutoreleasePoolPage`中插入一个哨兵对象，值为`nil`(0)，如下：

![插入哨兵对象](http://upload-images.jianshu.io/upload_images/1626952-7288fdfe17f32bf6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

函数`objc_autoreleasePoolPush`的返回值是哨兵对象的地址，作为函数`objc_autoreleasePoolPop`的入参。

所以会有如下操作：

* 根据哨兵的地址查找到page的地址
* 对晚于哨兵加入所有的`autorelease`对象发送一个`release`通知，并且移动next指针到哨兵对象位置。
* 移除所有的next指针之后的`autorelease`对象，可以跨多页page

![调用push后指针对象的移动图](http://upload-images.jianshu.io/upload_images/1626952-bee2d158b4c4f503.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**总结：以上即autoreleasepool的基本原理，所有图片均来自下方的链接**

[黑幕背后的Autorelease](http://blog.sunnyxx.com/2014/10/15/behind-autorelease/)  
[Objective-C Autorelease Pool 的实现原理](http://blog.leichunfeng.com/blog/2015/05/31/objective-c-autorelease-pool-implementation-principle/)

## **nonatomic和atomic**

这是保证属性在自动生成`getter/setter`方法时，对`setter`方法是否加锁，以确保线程安全。

* atomic : 线程安全，会在生成`setter`方法时候加上锁，确保在`setter`方法结束前只能有一个线程操作变量。也就是说，threadA正在修改变量，threadB就无法修改变量。可以确保`getter`到的数据一定是完整的。
* nonatmic : 没有对`getter/setter`进行原子操作，可能产生`getter`方法没有获取到完整数据。比如，当时threadA正在调用`getter`，此时threadB正在使用`setter`，假设对象的有几个属性，可能获取的对象就只能修改成功了其中两个属性。

优缺点 : `atomic`虽然保证了线程安全，但是添加了线程锁，所以效率比较低，在单线程中，没有必要使用原子性。`nonatomic`则相反，效率比较高。



 