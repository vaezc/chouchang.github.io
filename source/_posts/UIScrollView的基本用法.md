title: UIScrollView的基本用法
date: 2016-03-13 13:18:38
categories:
- iOS
tags:
- 笔记
- iOS

---

前言：在我们平常的使用中，UIScrollView也是经常要用到的，比如做图片轮播器什么的，还是蛮重要的，在这篇文章中，我就把UIScrollView的基本用法做一个总结。

# UIScrollerView是什么

因为在我们的手机设备中，往往一个手机屏幕能显示的东西太少了。虽然说现在的手机屏幕的尺寸不断的加大，但是不管是多么大的手机屏幕，能够一次性的显示出我们想要的信息，那还是不且实际的。所以我们就需要UIScrollerView,它是一个可以滚动的视图，当你所想要看到的消息超出屏幕的范围的时候，用手机对屏幕经行滑动，就可以呈现出屏幕以外的内容。最典型的例子就是iPhone上自带的设置应用。

## UIScrollerView的使用

用的时候其实也非常简单，只要我们先把UIScrollerView加入到当前视图，然后再设置UIScrollerView的contenSize，就是告诉UIScrollerView能显示的尺寸，然后通过拖动就能看到所有的内容了。

<!--- more -->


# UIscrollerView的基本属性


下面是在xcode中UIScrollerView的定义中复制过来的，对于这些属性都做了中文注释。


```
//内容的偏移值
@property(nonatomic)         CGPoint                      contentOffset;                  // default CGPointZero

//滚动范围的大小
@property(nonatomic)         CGSize                       contentSize;                    // default CGSizeZero

//设置的内容的间距
@property(nonatomic)         UIEdgeInsets                 contentInset;                   // default UIEdgeInsetsZero. add additional scroll area around content

//指定控件是否只能朝一个方向上滚动（默认为NO）
@property(nonatomic,getter=isDirectionalLockEnabled) BOOL directionalLockEnabled;         // default NO. if YES, try to lock vertical or horizontal scrolling while dragging

//控制控件遇到边框是否反弹(默认为YES)
@property(nonatomic)         BOOL                         bounces;                        // default YES. if YES, bounces past edge of content and back again

//遇到垂直边框是反弹（默认为NO，如果为YES，bounces也为YES）
@property(nonatomic)         BOOL                         alwaysBounceVertical;           // default NO. if YES and bounces is YES, even if content is smaller than bounds, allow drag vertically

//遇到水平边框是反弹（默认为NO，如果为YES，bounces也为YES）
@property(nonatomic)         BOOL                         alwaysBounceHorizontal;         // default NO. if YES and bounces is YES, even if content is smaller than bounds, allow drag horizontally

//控制控件是否整页翻动（默认为NO）
@property(nonatomic,getter=isPagingEnabled) BOOL          pagingEnabled __TVOS_PROHIBITED;// default NO. if YES, stop on multiples of view bounds

//控制控件是否能够滚动（默认为yes）
@property(nonatomic,getter=isScrollEnabled) BOOL          scrollEnabled;                  // default YES. turn off any dragging temporarily

//是否显示水平滚动条（默认为YES）
@property(nonatomic)         BOOL                         showsHorizontalScrollIndicator; // default YES. show indicator while we are tracking. fades out after tracking

//是否显示垂直滚动条（默认为YES）
@property(nonatomic)         BOOL                         showsVerticalScrollIndicator;   // default YES. show indicator while we are tracking. fades out after tracking

//指定滚动条在scrollerView中位置
@property(nonatomic)         UIEdgeInsets                 scrollIndicatorInsets;          // default is UIEdgeInsetsZero. 
adjust indicators inside of insets

//设置滚动条的样式
@property(nonatomic)         UIScrollViewIndicatorStyle   indicatorStyle;                 // default is UIScrollViewIndicatorStyleDefault



```


# UIScrollerViewDelegate
在我们学习的过程中，我们已经很明白，要想使用某个控件的方法，必须得去遵循这个控件的协议，然后实现这个协议中的方法。

```
@property(nullable,nonatomic,weak) id<UIScrollViewDelegate>        delegate;                       // default nil. weak reference

```

同样的如果我们想要使用UIScrollerView的方法，必须先要遵循UIScrollerVieDelegate的协议，然后我们来看下有哪些要实现的方法。


```
//scrollView滚动时，就调用该方法。任何offset值改变都调用该方法。即滚动过程中，调用多次 
- (void)scrollViewDidScroll:(UIScrollView *)scrollView{

    NSLog(@"scrollViewDidScroll");
    CGPoint point=scrollView.contentOffset;
    NSLog(@"%f,%f",point.x,point.y);
    // 从中可以读取contentOffset属性以确定其滚动到的位置。

    // 注意：当ContentSize属性小于Frame时，将不会出发滚动


}

```

```
// 当scrollView缩放时，调用该方法。在缩放过程中，会多次调用
- (void)scrollViewDidZoom:(UIScrollView *)scrollView{

    NSLog(@"scrollViewDidScroll");
    float value=scrollView.zoomScale;
    NSLog(@"%f",value);


}

```

```
// 当开始滚动视图时，执行该方法。一次有效滑动（开始滑动，滑动一小段距离，只要手指不松开，只算一次滑动），只执行一次。
- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView{

    NSLog(@"scrollViewWillBeginDragging");

}

```

```

// 滑动scrollView，并且手指离开时执行。一次有效滑动，只执行一次。
// 当pagingEnabled属性为YES时，不调用，该方法
- (void)scrollViewWillEndDragging:(UIScrollView *)scrollView withVelocity:(CGPoint)velocity targetContentOffset:(inout CGPoint *)targetContentOffset{

    NSLog(@"scrollViewWillEndDragging");

}


```


```
// 滑动视图，当手指离开屏幕那一霎那，调用该方法。一次有效滑动，只执行一次。
// decelerate,指代，当我们手指离开那一瞬后，视图是否还将继续向前滚动（一段距离），经过测试，decelerate=YES
- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate{

    NSLog(@"scrollViewDidEndDragging");
    if (decelerate) {
        NSLog(@"decelerate");
    }else{
         NSLog(@"no decelerate");

    }

    CGPoint point=scrollView.contentOffset;
    NSLog(@"%f,%f",point.x,point.y);

}

```

```
// 返回将要缩放的UIView对象。要执行多次
- (UIView *)viewForZoomingInScrollView:(UIScrollView *)scrollView{

    NSLog(@"viewForZoomingInScrollView");
    return  self.imgView;

}
```

```
// 当将要开始缩放时，执行该方法。一次有效缩放，就只执行一次。
- (void)scrollViewWillBeginZooming:(UIScrollView *)scrollView withView:(UIView *)view{

    NSLog(@"scrollViewWillBeginZooming");

}
```

```
// 当滚动视图动画完成后，调用该方法，如果没有动画，那么该方法将不被调用
- (void)scrollViewDidEndScrollingAnimation:(UIScrollView *)scrollView{

    NSLog(@"scrollViewDidEndScrollingAnimation");
    // 有效的动画方法为：
    //    - (void)setContentOffset:(CGPoint)contentOffset animated:(BOOL)animated 方法
    //    - (void)scrollRectToVisible:(CGRect)rect animated:(BOOL)animated 方法


}
```


```
// 指示当用户点击状态栏后，滚动视图是否能够滚动到顶部。需要设置滚动视图的属性：_scrollView.scrollsToTop=YES;
- (BOOL)scrollViewShouldScrollToTop:(UIScrollView *)scrollView{

    return YES;


}

```

**上面列举的方法很多，但是具体的使用还是要结合到我们开发实际要求中，有些方法用不到就不需要用，等到想要实现什么功能的时候我们再来想到底使用什么方法比较合适。**


# 实例图片轮播器
做图片轮播器的步骤很简单，我大概分为以下几步。

## 设置图片的滚动范围

首先在你要显示图片轮播器的view下添加一个UIScrollerView，让当前的controller遵循UIScrollerViewDelegate协议。再添加一个方法，来初始化这个UIScrollerView设置相关的属性，一般就是设置好scrollerView的contensize，添加定时器和pagecontrol，以及设置代理属性和隐藏到水平滚动条。

**具体代码如下：**
首先在storybord中添加UIScrollerView和pageControl，然后和代码连线，声明两个属性,还要设置一个定时器的属性，这个属性后来我们将要用到。

```
@interface ViewController () <UIScrollViewDelegate>

@property (weak, nonatomic) IBOutlet UIScrollView *scrollView;

@property (weak, nonatomic) IBOutlet UIPageControl *pageControl;

@property (nonatomic, strong) NSTimer *timer;

@end

```

在界面的viewdidload的方法中初始化scrollerView。通过一个循环，就将五个图片的view全部添加到scrollerview中。设置了5个图片来作为图片轮播器的播放contenSize的宽度。图片的高度的话就和scrollerview的高度保持一致。

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    int count = 5;
     CGSize size = self.scrollView.frame.size;
    //1 动态生成5个imageView
    for (int i = 0; i < count; i++) {
        //添加图片的view到scrollerview中
        UIImageView *iconView = [[UIImageView alloc] init];
        [self.scrollView addSubview:iconView];
        
        NSString *imgName = [NSString stringWithFormat:@"img_%02d",i+1];
        iconView.image = [UIImage imageNamed:imgName];
        
       
        CGFloat x = i * size.width;
        //frame
        iconView.frame = CGRectMake(x, 0, size.width, size.height);
    }
    
    //2 设置滚动范围  0表示默认高度
    self.scrollView.contentSize = CGSizeMake(count * size.width, 0);
    
    
    self.scrollView.showsHorizontalScrollIndicator = NO;
    //3 设置分页
    self.scrollView.pagingEnabled = YES;
    
    //4 设置pageControl
    self.pageControl.numberOfPages = count;
    
    //5 设置scrollView的代理
    self.scrollView.delegate = self;
    
    //6 定时器
    [self addTimerO];
    
    //立即执行定时器的方法
	//[timer begin];  
}

```

## 添加定时器

上面的代码中用到的定时器，因为图片轮播器是可以自动切换图片的，所以我们要添加一个定时器，使得过了一个固定的时间就会调用换到一下页的方法。而当我们去用手拖拽的时候就会发生一个问题，因为设置图片定时器的缘故，当你手拖拽图片的时候，定时器还在计时，当你放开手的时候。就会造成图片跳转的页数不是连续的，所以我们就要在代理的方法中去设置，当我们用手去拖拽图片的时候，图片的定时器器就应该停止，当我们松开手的时候，定时器就重新开始。还有关于一个用户体验的问题，当我们用手拖拽这个图片往左的位移超过当前图片的1/2的时候，按照我们的直觉习惯就是应该到下一张图片。要实现这个效果，我们也要去在代理方法中去设置，当用户的拖拽的位移超过1/2的时候，我们就设置页数为当前页数的一页。




```
- (void)addTimerO
{
	//设置一个定时器，间隔是2s来调用切换到下一张图片的方法。
    NSTimer *timer = [NSTimer timerWithTimeInterval:2.0 target:self selector:@selector(nextImage) userInfo:nil repeats:YES];
    self.timer = timer;
    //消息循环
    NSRunLoop *runloop = [NSRunLoop currentRunLoop];
    [runloop addTimer:timer forMode:NSRunLoopCommonModes];
}

- (void)nextImage
{
    //当前页码
    NSInteger page = self.pageControl.currentPage;
    
    if (page == self.pageControl.numberOfPages - 1) {
        page = 0;
    }else{
        page++;
    }

    //这里我们设置一个动画效果，让图片的切换显得不那么生硬
    CGFloat offsetX = page * self.scrollView.frame.size.width;
    [UIView animateWithDuration:1.0 animations:^{
        self.scrollView.contentOffset = CGPointMake(offsetX, 0);
    }];
    
}

```

## 代理方法的设置


```

#pragma mark - scrollView的代理方法
//正在滚动的时候
- (void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    //当滚动的距离超过图片的宽度一般的时候，我们就默认页数为当前页数的下一页
    int page = (scrollView.contentOffset.x + scrollView.frame.size.width / 2)/ scrollView.frame.size.width;
    self.pageControl.currentPage = page;
}

//当用户用手去拖拽图片的时候就应该停止定时器
- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView
{
    //停止定时器
    [self.timer invalidate];
}

//当用户拖拽完图片的时候就应该添加上定时器
- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate
{
    [self addTimerO];
}


```


**注意点：**
在上面设置定时器的时候我们设置了消息循环的mode，关于这个mode我们必须得知道，RunLoop只能运行在一种mode下，如果要换mode，当前的loop也需要停下重启成新的。利用这个机制，ScrollView滚动过程NSDefaultRunLoopMode（kCFRunLoopDefaultMode）的mode会切换到UITrackingRunLoopMode来保证ScrollView的流畅滑动：只能在NSDefaultRunLoopMode模式下处理的事件会影响ScrollView的滑动。

如果我们把一个NSTimer对象NSDefaultRunLoopMode（kCFRunLoopDefaultMode）添加到主运行循环中的时候, ScrollView滚动过程中会因为mode的切换，而导致NSTimer将不再被调度。

同时因为mode还是可定制的，所以

Timer计时会被scrollView的滑动影响的问题可以通过将timer添加到NSRunLoopCommonModes（kCFRunLoopCommonModes）来解决。