title: UITableView详解
date: 2016-03-10 20:00:00
categories:
- iOS
tags:
- 笔记
- iOS

---
前言：在现在的iOS开发中，有很多地方都用到了UITableView,可以这样说，基本上市场上流行的应用都用到了，所以它的重要性就是不言而喻的，今天我就做一下总结。

什么是UITableView?
-----------------

1.**UITableView继承自UIScrollView,支持垂直滚动，而且性能极好。**

UITbleView分为两种样式：

![](images/tableview1.png)

![](images/tableview2.png)

1. 不分组的  UITableViewStylePlain
2. 分组		UITableViewStyleGroued

```
 typedefNS_ENUM(NSInteger, UITableViewStyle) {
        UITableViewStylePlain,                 // regular table view
        UITableViewStyleGrouped                // preferences style table view
        };
 ```
 
2.可以分组显示内容，其中分组称为section，行称为row。

3.frame决定了tableview显示的位置和边框，每一行的位置都会放一个UITableViewCell来负责显示行的内容。

4.常用的属性：
1. 分割线样式		separatorStyle
2. 分割线颜色		sepatatorColor
3. 行高			rowHeight
4. 控制代理		delegate
5. 数据代理		dataSource
6. 与边框的分隔距离设置	separatorInset


UITableView用来干嘛
---

UITableView用来展示大量的数据信息给用户，比如自己有很多种信息想呈现给用户，就要通过tableview来呈现，在iOS系统自带的原生app里面非常常见，比如说通讯录这个应用，就通过每一行显示一个联系人来展现联系人的信息，当你点击时就会进入到另一个界面，下一个界面也是通过tableview来实现的，所以说掌握好UITableview对开发学习十分有帮助。

下面我就来讲一讲，它的具体用法。
<!-- more -->

UITableView主要是通过两个协议来实现展示数据和进行交互操作的：

## **UITableViewDataSource**


dataSource是UITableViewDataSource类型，其主要作用是**提供显示用的数据**（UITableViewCell）,UITableView会向数据源查询**一共有多少行数据以及每一行显示什么数据等**。和根据用户的操作进行相应的**数据更新操作**，如果数据没有更新具体操作的话，可能会导致数据显示混乱，严重的话会导致程序崩溃。**凡是遵守**UITableViewDataSource协议的OC对象，都可以是**UITableView的数据源**。


UITableViewDataSource协议
一般共有三个方法:

```

/**
 *  返回tableview有多少组数据，一般不写的话，默认是一组
 *
 *  @param tableView   当前的tableview对象
 *
 *  @return 返回多少组数据
 */
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
    return 1;
}

/**
 *  每一组有多少行数据
 *
 *  @param tableView 当前的tableview对象
 *  @param section   当前的组
 *
 *  @return 返回当前组一共有多少行的数据
 */
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return self.heroes.count;
}

/**
 *  每一行显示的内容
 *
 *  @param tableView 当前tableview的对象
 *  @param indexPath 行和列的集合
 *
 *  @return 返回当前tableview的行
 */

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    //1 创建cell
//    UITableViewCell *cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:nil];
    
    //从缓存池中取可用的cell
    static NSString *reuseId = @"hero";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:reuseId];
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:reuseId];
    }
    
  
    //2 给cell内部的子控件赋值
    //获取数据
    CZHero *hero = self.heroes[indexPath.row];
    cell.textLabel.text = hero.name;
    cell.detailTextLabel.text = hero.intro;
    cell.imageView.image = [UIImage imageNamed:hero.icon];
    
    
    
    //设置cell背景透明
    cell.backgroundColor = [UIColor clearColor];
    
    
    
    //设置cell的附属物
    cell.accessoryType = UITableViewCellAccessoryDisclosureIndicator;
    
    //自定义cell的附属物
    cell.accessoryView = [UIButton buttonWithType:UIButtonTypeContactAdd];
    
    //设置详细内容label的属性
    cell.detailTextLabel.numberOfLines = 0;
    
    
    //3 返回cell
    return cell;
}
```
先将对象遵循UITableViewDataSource协议，然后再实现以上的方法，最后才能在cell中显示出来。

上面代码里面的附属物的主要几个属性：

![](/images/附属物.png)

**UITableViewCell重用问题**

iOS设备的内存有限，如果用UITableView显示成千上万条数据，就需要成千上万个UITableViewCell对象的话，那将会耗尽iOS设备的内存。要解决该问题，需要重用UITableViewCell对象.
在代码中如果按照第一种方式创建cell对象的话，就没有用到缓存池技术。第二种创建方式就用到了缓存池技术，将每次划出界面外的cell放入到缓存池中，然后下面新划入到界面的cell，先判断缓存池中是否有未使用的cell，再从缓存池中拿到前面已经放入到缓存池中的cell对象，dataSource会用心得数据配置这个UITableViewCell,重新显示到窗口上，从而避免创建对象。节约了内存空间。

**重用时应该注意的问题**

有时候需要自定义UITableViewCell(用一个子类去继承UITableViewCell),而且每一行用的不一定是同一种的UITableViewCell,所以一个UITableView可能拥有不同类型的UITableViewCell，缓存池中也会有很多不同类型的UITableViewCell，那么在重用的时候就会导致显示了错误类型的cell。

**解决方法：**

UITableViewCell有一个NSString *reuseIdentifier属性，可以在初始化的UITabelViewCell的时候传入一个特定的字符串来设置reuseIdentifier(一般用UITableViewCell的类型)。当uitableview要求datasource返回UITableViewCell时，先通过一个字符串标示到缓存池中查找对应类型的UITableViewCell对象，如果有就重用，如果没有，就传入这个字符串标示来初始化一个UITableViewCell对象。
具体的重用代码如下:

```


- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{    // 1.定义一个cell的标识      static NSString *ID = @”cell";        // 2.从缓存池中取出cell      UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];        // 3.如果缓存池中没有cell      if (cell == nil) {        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:ID];    }        // 4.设置cell的属性...          return cell;}
```


## UITableViewDelegate

其主要几个方法如下：

```
/**
 *  返回当前行的高度
 */
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
    if (indexPath.row % 2 == 0) {
        return 60;
    }else{
        return 100;
    }
    
}

//当选中cell的时候执行
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    //选中行对应的数据
    CZHero *hero = self.heroes[indexPath.row];
    
    UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"提示" message:nil delegate:self cancelButtonTitle:@"取消" otherButtonTitles:@"确定", nil];
    //
    alertView.alertViewStyle = UIAlertViewStylePlainTextInput;
    
    //给alertView中得文本框赋值
    [alertView textFieldAtIndex:0].text = hero.name;
    
    //记录当前行号
    alertView.tag = indexPath.row;
    
    //
    [alertView show];
    
}

//section头部的title
-(NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section
{

    return @"搞个大新闻";
}

//section尾部的title
-(NSString *)tableView:(UITableView *)tableView titleForFooterInSection:(NSInteger)section
{
    return @"见风就是雨";
}

//section头部的高度
//-(CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section

//section尾部的高度
//-(CGFloat)tableView:(UITableView *)tableView heightForFooterInSection:(NSInteger)section


```
 
 