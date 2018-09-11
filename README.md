# TGGProgressHUD
**HUD这类控件在我们的实际开发中很是常用，几乎每个项目中都会大量的用到，用于弹出提示的文字或者图片告知用户当前正在做的事情**

**系统提供了UIAlertController(iOS8+)这个类，使用它我们可以弹出一部分的提示，但是往往和我们实际需求的是不一样的。所以我们需要自定义弹出视图**

**HUD这一类工具，MBProgressHUD和SVProgressHUD这两大开源控件技术每个iOS开发者都用过，而且这两个也几乎能全部满足于我们的需求了，但是随着APP的数量多不胜数，加上一些特定的项目，大家都希望自己的APP会有独特的地方或者创新的东西，而这些小小的“创新”，就会让我们开发者要多写很多东西了，而当这个创新落在“HUD”上的时候，熟悉了使用三方HUD的开发者就可能需要修改源码了，虽然“MBProgressHUD”在拓展性已经很好了，但是自己写一个HUD何乐而不为**

**最终的API是这样的：**
```objectivec
@interface TCProgressHUD (Public)
/** 提示文字，不会自动隐藏*/
+ (void)showStatus:(NSString *)status;
/** 提示文字，会自动隐藏*/
+ (void)showStatus:(NSString *)status andAutoHideAfterTime:(CGFloat)showTim e;
/** 提示成功 显示默认的图片，不会自动隐藏*/
+ (void)showSuccess;
/** 提示成功 显示默认的图片，会自动隐藏*/
+ (void)showSuccessAndAutoHideAfterTime:(CGFloat)showTime;
/** 提示成功 显示默认的图片，同时显示设定的文字提示，不会自动隐藏*/
+ (void)showSuccessWithStatus:(NSString *)status;
/** 提示成功 显示默认的图片，同时显示设定的文字提示，会自动隐藏*/
+ (void)showSuccessWithStatus:(NSString *)status andAutoHideAfterTime:(CGFlo at)showTime;
/** 提示失败 显示默认的图片，不会自动隐藏*/
+ (void)showError;
/** 提示失败 显示默认的图片，会自动隐藏*/
+ (void)showErrorAndAutoHideAfterTime:(CGFloat)showTime;
/** 提示失败 显示默认的图片，同时显示设定的文字提示，不会自动隐藏*/
+ (void)showErrorWithStatus:(NSString *)status;
/** 提示失败 显示默认的图片，同时显示设定的文字提示，会自动隐藏*/
+ (void)showErrorWithStatus:(NSString *)status andAutoHideAfterTime:(CGFloa t)showTime;
/** 提示正在加载 显示默认的图片 不会自动隐藏*/
+ (void)showProgress;
/** 提示正在加载 显示默认的图片，同时显示设定的文字提示 不会自动隐藏*/ + (void)showProgressWithStatus:(NSString *)status;
/** 弹出自定义的提示框 不会自动隐藏*/
+ (void)showCustomHUD:(UIView *)hudView;
/** 弹出自定义的提示框 会自动隐藏*/
+ (void)showCustomHUD:(UIView *)hudView andAutoHideAfterTime:(CGFloat)showTi me;
/** 移除提示框*/
+ (void)hideHUD;
/** 移除所有提示框*/
+ (void)hideAllHUDs; @end
```

###构思和实现部分
**因为我们希望是在APP运行的时候，在任何页面都能够方便的弹出HUD来，所以我们使用一个view来作为HUD的容器，在需要的地方直接弹出这个view即可，弹出的方式有两种**
* 添加为当前的view的subview展示到页面上
*  直接使用一个window来弹出HUD，以便于实现弹出的HUD在页面所有的控件的最上面

**上面提到的两种方式，我们采用第二种，因为window本身就是一个单例对象，并且很方便的在各个地方获取到，而且可以保证它在所有控件的最上层**

**接着看我们的层次关系，弹出的分为两层，底层是TCProgressHUD这个view层，用来作为hudView的容器，第二层就是hudView，用来显示弹出的内容**
![Alt text](https://github.com/TongCong/TGGProgressHUD/blob/master/img/Snip20180911_10.png)
**首先我们创建TCProgressHUD，这个view作为容器，是被window弹出来的，所以必然对外提供接口，初始化方法，被window弹出来的和移除的方法**
```objectivec
// 初始化方法
- (instancetype)init;
// 设置/添加hudView的方法
- (void)setHudView:(UIView *)hudView;
// 被window弹出的方法，同时设置显示的时间，当设置的时间小于等于0的时候将不会自动移除 
- (void)showWithTime:(CGFloat)time;
// 移除hud的方法
- (void)hide;
// 移除所有hud的方法
- (void)hideAllHUDs;
```


**接下来就是这些方法的内部实现了，这里就不一一赘述了，无非就是添加到window上，移除子控件等操作。但是有个地方需要注意，一般的HUD都会在几秒后消失这种需求，所以我们在这里规定一下API，传入时间为0的时候，默认不会消失，传入时间大于0，那么就按传入时间来倒计时：**
```objectivec
- (void)showWithTime:(CGFloat)time {
    UIWindow *window = [[UIApplication sharedApplication] keyWindow];
    if (self.superview == nil) { 
	    [window addSubview:self];
    }
    if (time > 0) {
        __weak typeof(self) weakself = self;
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(time* NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            __strong typeof(weakself) strongSelf = weakself;
            if (strongSelf) {
                [strongSelf hide];
            } });
    }
}
```

**那么我们在使用的时候就可以像下面的方式使用了：**
```objectivec
    /// 初始化contentView
    TCProgressHUD *contentView = [[TCProgressHUD alloc] init];
    contentView.frame = CGRectMake(100.0f, 100.0f, 200.0f, 200.0f); // 居中显示
    contentView.center = self.view.center;
    /// 初始化HUDView 和contentView大小一样
    UILabel *hudView = [[UILabel alloc] initWithFrame:contentView.bounds];
    hudView.textColor = [UIColor redColor];
    hudView.text = @"提示文字...";
    /// 设置/添加HUDView
    [contentView setHudView:hudView];
    // 被window弹出, 同时设置显示的时间, 当设置的时间小于等于0的时候将不会自动移除
    // 这里设置为0 则会一直显示在页面上, 直到手动调用hide方法才会被移除
    [contentView showWithTime:0];
    /// 这里模拟延时操作
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        [contentView hide];
    });
```

**现在就完成了自定义弹窗了，而且自定义程度非常高。。。但是这样的东西只能自己用用，因为`使用太复杂`了，我们需要初始化几个对象，设置frame，调用好几个方法，每次这样操作自己都觉得麻烦，所以，接下来就是怎样把自己写的效果，封装一下，达到好用方便的第三方效果**

**封装的时候，首先第一要想的就是别人想要怎么使用，怎么使用起来方便，并且合乎开发狗的逻辑思维。除了自定义一些视图，是不是有一些经常固定的几种视图可以封装起来。`像我们的这个HUD，首先考虑到是常用的功能，提示框一般居中显示，提示文字，提供成功失败的图片和文字，提供进度的动画等等`**

**比如：提示文字的需求，我们就可以封装一下，我希望使用的接口是这样的：**
```objectivec
// 弹出文字提示框, 提示框居中显示, 宽度会随着文字的长度调整 
[TCProgressHUD showStatus:@"正在努力加载中..."];
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3.0 * NSEC_PER_SE C)), dispatch_get_main_queue(), ^{
// 加载完后 移除提示框
   [TCProgressHUD hideHUD];
});
```

**接着实现这两个类方法，实现还是很简单的，和我们之前直接使用的方法类似，只需要做一些调用即可：**
```objectivec
/// 获得单例对象
TCProgressHUD *hudView = [TCProgressHUD sharedInstance];
/// 这里讲前面的UILabel换成了TCTextOnlyHUDView来显示文字
/// 所以需要封装一个TCTextOnlyHUDView来显示文字提示，并且在其中处理根据文字长的来设置frame的过程 所以在这里我们并没有设置frame
TCTextOnlyHUDView *textView = [[TCTextOnlyHUDView alloc] init];
textView.text = status;
/// 设置/添加 textView
[hudView setHudView:textView];
///弹出textView 设置时间为0.0f, 表示不需要自动被移除 
[hudView showWithTime:0.0f];
```

**当然每次都需要模拟一下延迟调用移除HUD太麻烦了，所以我们也可以这样调用**
```objectivec
// 传入延迟1S时间
[TCProgressHUD showStatus:@"正在努力加载中..." andAutoHideAfterTime:1.0];
```

**这里有个地方需要注意：在`showStatus`的方法里面，我们都没有设置它的frame，其实在这里是可以直接设置的，就是计算文字的尺寸，然后设置两者的frame，但是我们并不应该在这里设置。因为我们可能APP会有横屏的时候，如果在这里设置还要去监听旋转的通知，再重新设置frame，所以在这里将两者的frame设置放到了他们的`layoutSubviews`方法里面，这样就可以自动适配了**

**所以在TCProgressHUD文件里面，重写`layoutSubviews`方法**
```objectivec
- (void)layoutSubviews {
    [super layoutSubviews];
    if (self.superview) {// superView == window
        self.frame = self.superview.bounds;
        for (UIView *subview in self.subviews) {
	        // 这里的判断其实是封装完全之后，为了区分自定义HUD还是常规的几种
            if ([subview conformsToProtocol:@protocol(TCPrivateHUDProtocol)]) {
                /// 居中显示
                subview.center = self.center;
            }
            else {
                /// 自定义的hudView
                CGRect frame = subview.frame;
                subview.frame = frame;
                
            }
        }
    }
}
```

**然后还需要新建继承自UIView的TCTextOnlyHUDView文件，这里只需要有一个Label来显示文字就好，同时还需要有一个对外的属性text来接受外部的文字，然后根据文字计算frame：**
```objectivec
@interface TCTextOnlyHUDView : UIView
/** 设置提示文字*/
@property (strong, nonatomic) NSString *text; 
@property (strong, nonatomic) UILabel *label;
@end
```
**具体的实现就不写出来了，这里就说一下计算frame的部分，重写text的set方法**
```objectivec
- (void)setText:(NSString *)text {
    self.label.text = text;
    const CGFloat padding = 10.0f;
    CGRect textRect = [text boundingRectWithSize:CGSizeMake(MAXFLOAT, 100.0f) options:NSStringDrawingUsesLineFragmentOrigin attributes:@{NSFontAttributeName: self.label.font} context:nil];
    CGFloat screenWidth = [UIScreen mainScreen].bounds.size.width;
    CGFloat screenHeight = [UIScreen mainScreen].bounds.size.height;

    CGRect frame = self.frame;
    frame.size.width = textRect.size.width + 2*padding;
    if (frame.size.width > screenWidth) {
        frame.size.width = screenWidth - 2*padding;
    }
    frame.origin.x = (screenWidth - frame.size.width)/2;
    frame.size.height = textRect.size.height + 2*padding;
    frame.origin.y = (screenHeight - frame.size.height)/2;

    self.frame = frame;
    [self layoutIfNeeded];
}
```

**到目前为止，我们就实现了文字提示的接口，接下来一些其他的常用的接口就都可以按照这个设计规范来实现了，比如：成功失败的文字提示，成功失败的图片提示，加载中的转圈提示等等，在最终的demo里面已经实现了这些功能，运行的时候就可以看到效果和查看源码**

-------
**在写这个小工具之前，我也看了MBProgressHUD的源码，打开一看，就一个.m文件，瞬间很惊讶，难道是我实现这些功能太复杂了？我用了好多.m文件来着，进入里面一看，1000多行的代码，而且里面定义了一大堆宏，到处都有宏判断，瞬间觉得智商不够用，不知道从什么地方开始读......**

**自习看了下MBPregressHUD的源码，其实里面由很多class文件组成，这样一看就和我们之前的设计差不多了** 
![Alt text](./1520846346801.png)

**按照刚刚我们自己写的HUD来看，MBProgressHUD这个class应该是作为容器的，那么里面应该类似的处理了弹出，移除，设置frame等操作，其他的class应该就是所谓的自定义view的部分了，那么就重点看看MBProgressHUD这个class里面的东西**

**看一下他的使用demo:**
```objectivec
- (void)colorExample {
	MBProgressHUD *hud = [MBProgressHUD showHUDAddedTo:self.navigationController.view animated:YES];
	hud.contentColor = [UIColor colorWithRed:0.f green:0.6f blue:0.7f alpha:1.f];
	// Set the label text.
	hud.label.text = NSLocalizedString(@"Loading...",@"HUD loading title"); 
	dispatch_async(dispatch_get_global_queue(QOS_CLASS_USER_INITIATED, 0),
^{
	[self doSomeWork];
	dispatch_async(dispatch_get_main_queue(), ^{ 
		[hud hideAnimated:YES];
	}); });
}
```

**第一眼看上去觉得和我们的API有很大的区别，不过仔细看看，极其的类似。只是MBPregressHUD这个自定义的程度更高。虽然都是提供了类方法，里面处理和很多初始化的操作，不过MB并没有采用单例模式，所以在每个弹出hud的地方都会重新初始化(这里的内存优劣就不作研究了)，他这样做就便于了每个地方的hud的一些属性的自定义。`这里人家的MBPregressHUD的确牛逼，但是我们自己写的HUD也可以根据实际也去需求来拓展，而且可以修改自己的HUD，不需要去修改别人的三方，还担心代码不兼容。`**
