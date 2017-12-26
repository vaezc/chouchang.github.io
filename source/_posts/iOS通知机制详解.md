title: iOS通知机制详解
date: 2016-03-14 14:49:16
categories:
- iOS
tags:
- 笔记 
- iOS

---
前言：在iOS中关于消息在不同的对象之间的通信，有很多种模式。在我刚开始学习iOS开发的时候就知道iOS开发主要是采用了MVC的设计模式，然后在斯坦福的公开课视屏中第一节课就讲到了MVC的设计模式，当时还不是很了解。但是其中记得数据模型要和模型之间经行消息传递时，要用到通知的机制。

![](/images/MVC.png)

# 什么是通知机制呢？
每一个应用程都有一个通知中心（NSNotificationCenter）实例，专门来负责协助不同对象之间的消息通信。任何一个对象都可以向通知中心发布通知（NSNotofication），描述自己在做什么，其他对感兴趣的对象可以申请在某个特定通知发布时收到通知。

消息的发送者和消息的接受者两者之间可以一无所知，完全解耦。这种消息通知机制可以应用于任意时间和对象，观察者（消息的接受者）可以有多个，所以消息具有广播性质。只是需要注意的是：**观察者向通知中心注册以后，在不需要接受消息的时需要向消息中心注销，这种广播机制是典型的观察者模式**

举个栗子：
比如当我们在学校到了吃饭的点，这个时候一室友对寝室群说：”肚子好饿啊，有谁一起去买饭”。那么这个时候你对吃饭这个消息感兴趣，收到了消息之后，你立马做出了相应的回应，告诉你的室友叫他帮你带。那么你们这个寝室群叫做通知中心，你室友在通知中心发布了“吃饭”这个通知，而你就是通知的接受者，你现在只对吃饭这件事感兴趣，所以你就成为了观察者，也就是消息的接受者，而且你们寝室群不只是有你们两个人，同时其他室友可能也对吃饭这件事感兴趣，那么他们也是观察者。

<!-- more -->

# 通知的属性
一个完整的通知一般包含三个属性：

```
@interface NSNotification : NSObject <NSCopying, NSCoding>

//通知的名称
@property (readonly, copy) NSString *name;

//通知的发布者(是谁发送的通知)
@property (nullable, readonly, retain) id object;

//一些额外的信息（通知者发送给消息接受者的信息内容）
@property (nullable, readonly, copy) NSDictionary *userInfo;
```

初始化一个通知(NSNotification)对象一共有三个方法:

```

+(instancetype)notificationWithName:(NSString *)aName object:(id)anObject;
+(instancetype)notificationWithName:(NSString *)aName object:(id)anObject userInfo:(NSDictionary *)aUserInfo;
-(instancetype)initWithName:(NSString *)name object:(id)object userInfo:(NSDictionary *)userInfo;


```

# 通知机制的使用
## 观察者注册消息通知
通知中心有相应的方法来给观察者注册消息的通知的

第一种：

```
    
   [NSNotificationCenter defaultCenter]addObserver:<#(nonnull id)#> selector:<#(nonnull SEL)#> name:<#(nullable NSString *)#> object:<#(nullable id)#>];
   
  //默认的通知中心的方法原型
  - (void)addObserver:(id)observer selector:(SEL)aSelector name:(NSString *)aName object:(id)anObject;
```
上述原型参数说明：

* observer:监听器，就是谁要接受这个通知
* aSelector:收到通知后，回调监听器这个方法，并且把通知对象当做参数传入
* aName: 通知的名称。如果为nil，那么无论通知的名称是什么，监听器都能收到这个通知
* anObjiect: 通知的发布者。如果anObjuect和aName都为nil，监听器都收到所有的通知


第二种：

```
- (id)addObserverForName:(NSString *)name object:(id)obj queue:(NSOperationQueue *)queue usingBlock:(void (^)(NSNotification *note))block;

```
上述原型参数说明：

* name:通知的名称
* obj：通知的发布者
* block：收到相应通知时，会回调这个block
* queue：决定了block在哪个操作队列中执行，如果传nil，默认在当前操作队列中同步执行。


# 发送消息通知


```

- (void)postNotificationName:(NSString *)aName object:(id)anObject userInfo:(NSDictionary *)aUserInfo;

```
发布一个名称为aName的通知，anObject为这个通知的发布者，aUserInfo为额外信息.


# 处理消息通知
当监听器接受到通知的时候，就会调用监听器中的所对应的处理消息通知的方法，然后做出相应的反应。

# 观察者注销，移除消息观察者
通知中心不会保留监听器的对象，在通知中心注册过的对象，必须在该对象释放前取消注册。否则，当相应的通知再次出现时，通知中心任然会向该监听器发送消息。因为相应的监听器已经被释放了，所以可能会导致应用程序崩溃。虽然在 iOS 用上 ARC 后，不显示移除 NSNotification Observer 也不会出错，但是这是一个很不好的习惯，不利于性能和内存。

通知中心提供了相应的方法来注销观察者（监听器）：

```

-(void)removeObserver:(id)observer;

-(void)removeObserver:(id)observer name:(NSString *)aName object:
(id)anObject;

```

一般在监听器销毁之前取消注册，所以我们一般放在监听器的dealloc方法中：

```
- (void)dealloc
{
	//在非ARC中需要调用此句
	[super dealloc];

	[NSNotificationCenter defaultCenter] removerObserver:self];
}
```
