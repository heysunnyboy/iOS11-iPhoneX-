## iOS11与iPhoneX适配
---
**iPhone所有机型的尺寸表：**

![1546239-994f620cfacb5e04.png](http://upload-images.jianshu.io/upload_images/8163699-bc15bd5b838be5ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**iPhone X基本属性**


![屏幕快照 2017-10-21 下午3.14.57副本副本.png](http://upload-images.jianshu.io/upload_images/8163699-0b01578b72e7f0c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 启动图尺寸：1125px × 2436px(即 375pt × 812pt @3x))
* iphoneX 屏幕高：812.0个点
  * 导航栏高度+状态栏高度：88.0个点(导航栏高度仍是44个点，状态栏高度增高为44个点，所以刘海的高度并不是状态栏的高度。状态栏和导航栏平分了头部总的高度）
* tabbar高度：83.0个点（原是固定49个点，增高了34个点).

### 顶部适配
--- 
* iOS11automaticallyAdjustsScrollViewInsets属性废弃了 系统自己计算内边距 会出现ScorllView下沉20的现象

![Snip20171021_8.png](http://upload-images.jianshu.io/upload_images/8163699-53e02b431aa04642.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以调用scrollview新的`apicontentInsetAdjustmentBehavior`
```
self.automaticallyAdjustsScrollViewInsets = NO;
if (@available(iOS 11.0, *)) {
    self.tableView.contentInsetAdjustmentBehavior = UIScrollViewContentInsetAdjustmentNever;
}
```
但是这么写会导致在iPhoneX下出现，由于在X下安全区域的出现，顶部异形区域不建议覆盖，会造成视觉的差异

![Snip20171021_10.png](http://upload-images.jianshu.io/upload_images/8163699-3c8df989c4fc363e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在代码中我们需要来根据设备高度来判断iPhoneX，从而来避免这种情况

```swift
    [tableview mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(self.view);
        if (LL_iPhoneX) {
            if (@available(iOS 11.0, *)) {
                make.top.equalTo(self.view.mas_safeAreaLayoutGuideTop);
            }
        }else{
            make.top.equalTo(self.view.mas_top).offset(0);
        }

        make.left.equalTo(self.view).offset(0);
        make.right.equalTo(self.view).offset(0);
        make.bottom.equalTo(self.view).offset(0);
    }];
```

![Snip20171021_11.png](http://upload-images.jianshu.io/upload_images/8163699-857fb6b841632ba5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面使用的Masonry,并且版本需要在V1.1.0版本

![Snip20171021_12.png](http://upload-images.jianshu.io/upload_images/8163699-c270b1e0a8a52e4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

苹果自己的约束代码如下
```swift
if (@available(iOS 11.0, *)) {
  NSLayoutConstraint *top = [contentView.topAnchor     constraintEqualToAnchor:self.view.safeAreaLayoutGuide.top  Anchor];
  NSLayoutConstraint *bottom = [contentView.bottomAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.bottomAnchor];
  NSLayoutConstraint *left = [contentView.leftAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.leftAnchor];
  NSLayoutConstraint *right = [contentView.rightAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.rightAnchor];
  [NSLayoutConstraint activateConstraints:@[top, bottom, left, right]];
} else {
  NSLayoutConstraint *top = [contentView.topAnchor constraintEqualToAnchor:self.topLayoutGuide.bottomAnchor];
  NSLayoutConstraint *bottom = [contentView.bottomAnchor constraintEqualToAnchor:self.bottomLayoutGuide.topAnchor];
  NSLayoutConstraint *left = [contentView.leftAnchor constraintEqualToAnchor:self.view.leftAnchor];
  NSLayoutConstraint *right = [contentView.rightAnchor constraintEqualToAnchor:self.view.rightAnchor];
  [NSLayoutConstraint activateConstraints:@[top, bottom, left, right]];
}
```
我使用的coocaPods导入的Masonry的Podfile.

![Snip20171021_13.png](http://upload-images.jianshu.io/upload_images/8163699-9a3a160aded2cbe0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 适配宏
--- 

```swift
// UIScreen width.
#define  HT_ScreenWidth   [UIScreen mainScreen].bounds.size.width

// UIScreen height.
#define  HT_ScreenHeight  [UIScreen mainScreen].bounds.size.height
// iPhone X
#define  HT_iPhoneX (HT_ScreenWidth == 375.f && HT_ScreenHeight == 812.f ? YES : NO)

// Status bar height.
#define  HT_StatusBarHeight      (HT_iPhoneX ? 44.f : 20.f)

// Navigation bar height.
#define  HT_NavigationBarHeight  44.f

// Tabbar height.
#define  HT_TabbarHeight         (HT_iPhoneX ? (49.f+34.f) : 49.f)

// Tabbar safe bottom margin.
#define  HT_TabbarSafeBottomMargin         (HT_iPhoneX ? 34.f : 0.f)

// Status bar & navigation bar height.
#define  HT_StatusBarAndNavigationBarHeight  (HT_iPhoneX ? 88.f : 64.f)

#define HT_ViewSafeAreInsets(view) ({UIEdgeInsets insets; if(@available(iOS 11.0, *)) {insets = view.safeAreaInsets;} else {insets = UIEdgeInsetsZero;} insets;})
```
