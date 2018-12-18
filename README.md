# KNPhotoBrower
高仿 微博 图片浏览器


⭐️⭐️⭐️⭐️⭐️⭐️⭐️ 有任何需要增加的功能,请直接邮箱联系我.欢迎点赞,谢谢 ⭐️⭐️⭐️⭐️⭐️⭐️⭐️




![image](https://github.com/LuKane/KNImageResource/blob/master/PhotoBrower/PhotoBrower.gif?raw=true)
![image](https://github.com/LuKane/KNImageResource/blob/master/PhotoBrower/collectionView.gif?raw=true)
![image](https://github.com/LuKane/KNImageResource/blob/master/PhotoBrower/scrollView.gif?raw=true)
![image](https://github.com/LuKane/KNImageResource/blob/master/PhotoBrower/tableView.gif?raw=true)

## 更新内容
* 1.增加 横竖屏处理 photoBrower.isNeedDeviceOrientation = true; // 默认为 false, 不旋转
* 2.适配iPhoneX
* 3.下一个版本 即将更新 : 改成高仿 微信朋友圈的样式,更好的提供屏幕旋转和加载样式, 下一版时间可能较长

## 一.功能描述及要点
* 1.加载网络九宫格图片,tableView,scrollView,collectionView
* 2.适配 SDWebImage 4.0,并下载图片,KNProgressHUD显示加载进度
* 3.高仿微博,显示动画,KNToast提示
* 4.支持删除功能,并提供 删除的 相对下标 和 绝对下标
* 5.已提供详细Demo,方便开发者阅读
* 6.新增 本地图片的加载

## 二.方法调用
### 1.创建KNPhotoBrower,并传入相应的参数
```
// 每一个图片控件对象, 对一一对应 KNPhotoItems ,再将多个KNPhotoItems 对象放入数组
KNPhotoItems *items = [[KNPhotoItems alloc] init];
items.url = [urlArr[i] stringByReplacingOccurrencesOfString:@"thumbnail" withString:@"bmiddle"];
items.sourceView = imageView;

KNPhotoBrower *photoBrower = [[KNPhotoBrower alloc] init];
photoBrower.itemsArr = [_itemsArray copy];// KNPhotoItems对象的数组
photoBrower.currentIndex = tap.view.tag;// 当前点击的哪个图片
photoBrower.actionSheetArr = [self.actionSheetArray mutableCopy];//设置 ActionSheet的选项

/***************************** 注意 *******************************/ 
[photoBrower setIsNeedRightTopBtn:NO]; // 如果不想要 图片的长按手势 和 右上角 操作按钮, 则设置 为NO
[photoBrower setIsNeedPictureLongPress:NO]; // 是否 需要 长按图片 弹出框功能 , 默认:需要

// 2016.10.17日 bug修改 --> 当collectionViewCell被循环利用时,会产生 图片错位. 解决方案 :增加两个属性, 保证collectionViewCell循环利用后,一样可以用
/****************  为了 循环利用 而做出的 新的属性 (collectionView中才会用到)  *****************/
[photoBrower setDataSourceUrlArr:[self.collectionPrepareArr copy]];
[photoBrower setSourceViewForCellReusable:_collectionView];
/****************  为了 循环利用 而做出的 新的属性 (collectionView中才会用到)  *****************/
[photoBrower present];// 显示
```

### 2.本地图片的加载 需注意 (由于是本地图片,所以 sourceView上的图片一开始就能加载出来,所以不需要传入 )
#### 2.1 如果不需要循环利用
```
KNPhotoItems *items = [[KNPhotoItems alloc] init];
items.sourceView = imageView;
// 而 items.url 则不设置任何值
``` 
#### 2.2 如果需要循环利用 -->例如collectionView
```
1.已经加载的Cell :
KNPhotoItems *items = [[KNPhotoItems alloc] init];
items.sourceView = cell.iconView;

2.没有加载出来的Cell:
KNPhotoItems *items = [[KNPhotoItems alloc] init];
items.sourceImage = [UIImage imageNamed:@"19.jpg"];
items.sourceView = nil;
```
### 3.提供代理方法 --> KNPhotoBrowerDelegate
```
/* PhotoBrower 即将消失 */
- (void)photoBrowerWillDismiss;
/* PhotoBrower 右上角按钮的点击 */
- (void)photoBrowerRightOperationActionWithIndex:(NSInteger)index;
/* PhotoBrower 保存图片是否成功 */
- (void)photoBrowerWriteToSavedPhotosAlbumStatus:(BOOL)success;
/* PhotoBrower 删除图片成功后返回-- > 相对 Index */
- (void)photoBrowerRightOperationDeleteImageSuccessWithRelativeIndex:(NSInteger)index;
/* PhotoBrower 删除图片成功后返回-- > 绝对 Index */
- (void)photoBrowerRightOperationDeleteImageSuccessWithAbsoluteIndex:(NSInteger)index;

```
### 3.提供 消失方法
```
[_photoBrower dismiss];
```

### 4.设置 参数
```
/**
 *  是否需要右上角的按钮. Default is YES;
 */
@property (nonatomic, assign) BOOL isNeedRightTopBtn;
/**
 *  是否需要 顶部 1 / 9 控件 ,Default is YES
 */
@property (nonatomic, assign) BOOL isNeedPageNumView;
/**
 *  是否需要 底部 UIPageControl, Default is NO
 */
@property (nonatomic, assign) BOOL isNeedPageControl;
/**
 *  存放 ActionSheet 弹出框的内容 :NSString类型
 */
@property (nonatomic, strong) NSMutableArray *actionSheetArr;
```
### 5.[强烈要求将以下代码写到代理方法中去]关于弹出框的内容,可在KNPhotoBrower.m 的operationBtnIBAction 方法中增减 (注意:代码中会存在强引用,所以切记将 weakSelf写入,不然当浏览器消失的时候,会存在强引用,不走 dealloc 方法)
```
#pragma mark - 右上角 按钮的点击
- (void)operationBtnIBAction{
    __weak typeof(self) weakSelf = self;
    
    if(_actionSheetArr.count != 0){ // 如果是自定义的 选项
        
        KNActionSheet *actionSheet = [[KNActionSheet alloc] initWithCancelBtnTitle:nil destructiveButtonTitle:nil otherBtnTitlesArr:[weakSelf.actionSheetArr copy] actionBlock:^(NSInteger buttonIndex) {
            
            // 让代理知道 是哪个按钮被点击了
            if([weakSelf.delegate respondsToSelector:@selector(photoBrowerRightOperationActionWithIndex:)]){
                [weakSelf.delegate photoBrowerRightOperationActionWithIndex:buttonIndex];
            }
            
#warning 如果传入的 ActionSheetArr 有下载图片,删除图片 这一选项. 则在这里调用和下面一样的方法 switch.....,如果没有下载图片,则通过代理方法去实现...
            
        }];
        [actionSheet show];
    }else{
        KNActionSheet *actionSheet = [[KNActionSheet alloc] initWithCancelBtnTitle:nil destructiveButtonTitle:@"删除" otherBtnTitlesArr:@[@"保存图片",@"转发微博",@"赞"] actionBlock:^(NSInteger buttonIndex) {
            
            // 让代理知道 是哪个按钮被点击了
            if([weakSelf.delegate respondsToSelector:@selector(photoBrowerRightOperationActionWithIndex:)]){
                [weakSelf.delegate photoBrowerRightOperationActionWithIndex:buttonIndex];
            }
            
            switch (buttonIndex) {
                case 0:{ // 删除图片
#pragma mark - 删除图片 
                    // 0: 删除后 回调返回 相对 下标
                    if([weakSelf.delegate respondsToSelector:@selector(photoBrowerRightOperationDeleteImageSuccessWithRelativeIndex:)]){
                        [weakSelf.delegate photoBrowerRightOperationDeleteImageSuccessWithRelativeIndex:weakSelf.currentIndex];
                    }
                    
                    KNPhotoItems *items = _itemsArr[_currentIndex];
                    NSInteger index = [_tempArr indexOfObject:items];
                    // 1: 删除后 回调返回 绝对 下标
                    if([weakSelf.delegate respondsToSelector:@selector(photoBrowerRightOperationDeleteImageSuccessWithAbsoluteIndex:)]){
                        [weakSelf.delegate photoBrowerRightOperationDeleteImageSuccessWithAbsoluteIndex:index];
                    }
                    
                    [weakSelf deleteImageIBAction];
                }
                    break;
                case 1:{ // 下载图片
#pragma mark - 下载图片
                    SDWebImageManager *mgr = [SDWebImageManager sharedManager];
                    KNPhotoItems *items = weakSelf.itemsArr[weakSelf.currentIndex];
                    if(![mgr diskImageExistsForURL:[NSURL URLWithString:items.url]]){
                        [[KNToast shareToast] initWithText:@"图片需要下载完成"];
                        return ;
                    }else{
                        UIImage *image = [[mgr imageCache] imageFromDiskCacheForKey:items.url];
                        dispatch_async(dispatch_get_main_queue(), ^{
                            UIImageWriteToSavedPhotosAlbum(image, self, @selector(image:didFinishSavingWithError:contextInfo:), NULL);
                        });
                    }
                }
                /**
                 *  剩下的需要自己去实现,获取通过代理方法去实现
                 */
                default:
                    break;
            }
        }];
        [actionSheet show];
    }
}
```

## 补充
* 1.适合 九宫格样式,tableView,scrollView,collectionView
* 2.如果有bug, 请在Github上通过 '邮箱' 或者 直接issue ,我会尽快修改
* 3.图片删除功能,提供相对和绝对下标方便开发者调用
* 4.支持iPhone、iPad端
* 5.适应于 本地图片和网络图片的加载
* 6.屏幕旋转已提供API
