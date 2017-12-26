title: iOS开发中四种本地存储
date: 2016-03-01 18:03:58
categories:
- iOS
tags:
- 笔记 
- iOS

---
# 前言
在学习的过程中学习到iOS中有四种本地存储，也就是数据持久化，把数据存储在沙盘中，使得应用程序在退出下次访问的时候可以继续读取数据，现在把他们做一个总结。

+ **plist**(文件属性)
+ **preference**(偏好设置)
+ **NSkeyedArchiver**(归档)
+ **SQLite3**
+ **CoreDate**

# 沙盒

在说上述存储方式之前，我们先要清楚社和目录的机制，在iOS中，每个程序都只能访问自己的目录，那么那个目录就成为沙盒目录。

## 1.结构

我们先来看下沙盒下有那些目录

![](/images/目录结构.png)

### Documents

保存应用运行时生成的需要持久化的数据，iTunes同步设备时会备份该目录。例如，游戏应用可将游戏存档保存在该目录

```
NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,NSUserDomainMask, YES).firstObject;
  NSLog(@"%@", path);
```

### Library/Caches

iTunes不会同步此文件夹，适合存储一些体积较大，不需要备份的数据。

```
NSString *caches=[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
NSLog(@"%@",caches);
 ```

### Library/Preferences

保存应用的所有偏好设置，iOS的Settings(设置)应用会在该目录中查找应用的设置信息。iTunes同步设备时会备份该目录。


### tmp

保存应用运行时所需的临时数据，使用完毕后再将相应的文件从该目录删除。应用没有运行时，系统也可能会清除该目录下的文件。iTunes同步设备时不会备份该目录。


```
    NSString *tmpPath = NSTemporaryDirectory();
    NSLog(@"%@",tmpPath);
 ```

## 2.Plist文件

Plist就是属性列表（Property List）的简称，是Apple公司自己发明的来存储数据的。plist文件就是将某些特定的类，通过xml文件的方式保存在目录中。

可以被序列化的类型只有以下几种：

+ NSArray

+ NSMutableArray

+ NSDictionary

+ NSMutableDictionary

+ NSString

+ NSMutableString

+ NSNumber

+ NSData

<!-- more -->

### 2.1 获得文件路径

我们要存储plist文件，首先要获得文件的路径，一般申明一个属性来获得文件的路径，通过懒加载来初始化文件的路径。

```
@interface ViewController ()

@property (nonatomic, copy) NSString * plistPath;

@end

@implementation ViewController

-(NSString *)plistPath{
    if (!_plistPath) {
        NSString *doc = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask,YES)lastObject];
        NSString *plistPath = [doc stringByAppendingString:@"test.plist"];
        _plistPath = plistPath;
    }
    return  _plistPath;
}
```
### 2.2 存储数据
现在可以存储数据到文件中，注意只有上述的类型才可以序列化到文件中。下面的atomically表示师父需要先写入一个辅助文件，再把辅助文件拷贝到目标文件地址。这是更安全的写入方式，一般都写上YES。

```
  NSArray * array = @[@"123",@"hello",@"navie"];
    [array writeToFile:self.plistPath atomically:YES];
```

### 2.3 读取

```
  NSArray *data = [NSArray arrayWithContentsOfFile:self.plistPath];
  NSLog(@"%@",data);
```

##3.Preference偏好设置
通常用来保存应用的设置信息，也就是用户的偏好设置

### 3.1 先获得NSUserDefaults文件

```
 NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
```
### 3.2 向文件中存储数据
无论是增删改的话，都要执行文件同步，理由在下面代码中。

```
//设置用户偏好的里面的属性
    [defaults setObject:@"zhangsan" forKey:@"name"];
    [defaults setObject:@"18" forKey:@"age"];
    
//因为当刚开始存储的时候，数据是存储在内存中，还没来的及存储在硬盘中，所以必须进行手动同步，将数据同步在硬盘中
    [defaults synchronize];
    
//更改数据
    [defaults setObject:@"lisi" forKey:@"name"];
    [defaults synchronize];
    
//删除数据
    [defaults removeObjectForKey:@"age"];
    [defaults synchronize];


```

### 3.3 读取文件中的数据
偏好设置会将所有数据保存到同一个文件中。即preference目录下的一个以此应用包名来命名的plist文件。

```
 NSString *name = [defaults objectForKey:@"name"];
 
 NSNumber *age  = [defaults objectForKey:@"age"];
    
 NSLog(@"姓名：%@\n,年龄：%@",name,age);
```

## 4.NSkeyedArchiver(归档)
注意只有遵循了NSCoding协议的对象都可以通过NSkeyedArchiver实现序列化，存储在沙盒中。但是由于绝大多数支持数据存储的Foundation和cocoa Touch 类都遵循了NSCoding协议，因此，对于绝大多数类来说，归档还是很容易实现的，而对于自定义的类，必须要实现NSCoding协议，并实现协议声明的方法，才能使用归档来存储对象的数据。
### 4.1遵循NSCoding协议
NSCoding 协议声明了两个方法，这两个方法是必须要实现的，一个用来归档，一个用来解档。通俗点讲就是一个用来将对象存储在文件中，另一个用来将对象从文件中读取出来。

#### 4.1.1对于实现了nscoding协议的类来说

```
//首先还是要定义存储文件的路径的属性
@interface ViewController ()

@property (nonatomic, copy) NSString *path;

@end

@implementation ViewController

-(NSString *)path{
    if (!_path) {
 
 NSString *doc= 	 [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, 	 NSUserDomainMask, YES)lastObject];
	 
   	 NSString *path = [doc stringByAppendingString:@"test.data"];
        _path = path;
    }
    return  _path;
}

//然后就可以经行归档和解档的操作了
- (void)viewDidLoad {
    [super viewDidLoad];
    
    //数组的归档和存储
    NSArray *data1 = @[@"123",@"zhangsan",@"nimen",@"navie"];
    [NSKeyedArchiver archiveRootObject:data1 toFile:self.path];
    
  id data2 = [NSKeyedUnarchiver unarchiveObjectWithFile:self.path];
    NSLog(@"%@",data2);
}

```

#### 4.2对于没有实现nscoding协议的自定义类来说
1. 首先是让自定义的类遵循nscoding协议。

```
@interface Person : NSObject<NSCoding>

@property (nonatomic,copy)NSString *name;
@property (nonatomic,copy)NSString *age;
@property (nonatomic,copy)NSString *addres;
@end
```

2. 实现协议中的两个方法

```
@implementation Person

//encodeWithCoder:
//每次归档对象时，都会调用这个方法。一般在这个方法里面指定如何归档对象中的每个实例变量，可以使用encodeObject:forKey:方法归档实例变量

//initWithCoder:
//每次从文件中恢复(解码)对象时，都会调用这个方法。一般在这个方法里面指定如何解码文件中的数据为对象的实例变量，可以使用decodeObject:forKey方法解码实例变量

-(void)encodeWithCoder:(NSCoder *)aCoder{
    [aCoder encodeObject:self.name forKey:@"name"];
    [aCoder encodeObject:self.age forKey:@"age"];
    [aCoder encodeObject:self.addres forKey:@"address"];

}

-(id)initWithCoder:(NSCoder *)aDecoder{
    //解析文件的每一个人的键值
    if (self = [super init]) {
        self.name = [aDecoder decodeObjectForKey:@"name"];
        self.age = [aDecoder decodeObjectForKey:@"age"];
        self.addres = [aDecoder decodeObjectForKey:@"address"];
    }
    return self;
}
```
**特别注意**
如果需要归档的类是某个自定义类的子类时，就需要在归档和解档之前先实现父类的归档和解档方法。即 [super encodeWithCoder:aCoder] 和 [super initWithCoder:aDecoder] 方法;

3. 开始使用

*归档*

```
	Person *person = [[Person alloc]init];
    person.name = @"zhansan";
    person.age =@"18";
    person.addres =@"earth";
    
    [NSKeyedArchiver archiveRootObject:person toFile:self.path];
```
*解档*

```
   Person *person = [NSKeyedUnarchiver 	
   unarchiveObjectWithFile:self.path];
   
   NSLog(@"%@,%@,%@",person.name,person.age,person.addres);

```
#### 对于多个对象的解档和归档
注意点：如果文件中要写入多个对象的话，就需要建立一个数据块，然后将数据块链接到NSKeyedArchiver对象上，然后在将数据块存入如到文件中，具体实现如下：

**归档**

```
 	Person *person1 = [[Person alloc]init];
    person1.name = @"zhangsan";
    person1.age = @"14";
    person1.addres =  @"city";
    
    Person *person2 = [[Person alloc] init];
    person2.name = @"lisi";
    person2.age = @"18";
    person2.addres = @"city";
    
    //新建一个可变数据块
    NSMutableData * data = [NSMutableData data];
    
    //将数据块链接到一个NSKeyedArchiver对象上
    NSKeyedArchiver *archiver = 
    [[NSKeyedArchiver 	alloc]initForWritingWithMutableData:data];
    
    //开始归档，数据会存储到NSMutableData中
    [archiver encodeObject:person1 forKey:@"zhangsan"];
    [archiver encodeObject:person2 forKey:@"lisi"];
    
    //存档完毕。注意：一定要调用这个方法。
    [archiver finishEncoding];
    
    //将数据写入到文件中
    [data writeToFile:self.path atomically:YES];
```

**解档**

```

	NSMutableData *data = 	
	[NSMutableDatadataWithContentsOfFile:self.path];
    
    NSKeyedUnarchiver *unarchiver = 
    [[NSKeyedUnarchiver alloc]initForReadingWithData:data];
    
    Person *person1 = [unarchiver decodeObjectForKey:@"zhangsan"];
    Person *person2 = 
    [unarchiver decodeObjectForKey:@"lisi"];
    
    [unarchiver finishDecoding];
    NSLog(@"%@,%@",person1.name,person2.name);
    
```


## 5.SQLite3

上述所讲的所有存储方法，都是覆盖存储。如果想要增加一条数据就必须把整个文件读出来，然后修改数据后再把整个内容覆盖写入文件。所以它们都不适合存储大量的内容。

    
