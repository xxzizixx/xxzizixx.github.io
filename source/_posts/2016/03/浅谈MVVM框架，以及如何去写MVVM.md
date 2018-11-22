---
title: 浅谈MVVM框架，以及如何去写MVVM
date: 2016-03-11 11:24:17
tags: 
    - 求学之路
---

对于MVVM框架，大家应该并不陌生，如果对这方面还不清楚的，可以去看一下一下三篇文章，应该会有一个比较清楚的认识。

[MVVM奇葩说](http://www.cocoachina.com/ios/20160520/16004.html)
[被误解的 MVC 和被神化的 MVVM](http://blog.devtang.com/2015/11/02/mvc-and-mvvm/)
[iOS 架构模式–解密 MVC，MVP，MVVM以及VIPER架构](https://www.cnblogs.com/oc-bowen/p/6255475.html)

读了这三篇文章，你应该就不会对MVVM陌生了， 我这里算是对以上几篇文章以及个人的理解，上代码展示一下自己认为的MVVM写法，当然：我这里的写法是从唐巧的猿题库里面借鉴过来的，算是对MVVM的一个变种吧。


## Talk is cheap, show you the code.

### 1、M层
```
#import <Foundation/Foundation.h>
@interface JBSystemMessageModel : NSObject
/** 消息ID */
@property (nonatomic, assign) int messageID;
/** 作者 */
@property (nonatomic, copy) NSString *author;
/** 标题 */
@property (nonatomic, copy) NSString *title;
/** 内容 */
@property (nonatomic, copy) NSString *content;
/** 时间 */
@property (nonatomic, copy) NSString *publishedTime;
/** 是否阅读 */
@property (nonatomic, assign, readonly) BOOL isRead;
@end
```


### 2、V层

当然，严格上说Controller也是V层，但我比较喜欢把Controller看成是“胶水”，也就是把M、V、VM链接在一起然后展示到界面的强力胶，所以这里的V层主要展示SystemMessageCell。

```
#import <UIKit/UIKit.h>
#import "JBSystemMessageModel.h"

@protocol JBSystemMessageDelegate <NSObject>

@optional
- (void)moreInformation:(JBSystemMessageFrameModel *)frameModel;

@end

@interface JBSystemMessageCell : UITableViewCell

@property (nonatomic, strong) JBSystemMessageFrameModel *frameModel;

@property (nonatomic, weak) id<JBSystemMessageDelegate> delegate;    

+ (instancetype)cellWithTableView:(UITableView *)tableView;

@end
```

SystemMessageCell.m文件，其实也就是大家常写的控件的创建(单纯的创建，不写任何业务逻辑，最后赋值还是用setFrameModel进行赋值)。
```
@implementation JBSystemMessageCell   
+ (instancetype)cellWithTableView:(UITableView *)tableView {
staticNSString *reuseID = @"JBSystemMessageCell";
JBSystemMessageCell *cell = [tableView dequeueReusableCellWithIdentifier:reuseID];      
if (!cell) {
cell = [[JBSystemMessageCell alloc] initWithStyle:UITableViewCellStyleDefaultreuseIdentifier:reuseID];
}
return cell;
}

- (instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier {
if (self = [superinitWithStyle:style reuseIdentifier:reuseIdentifier]) {    
self.backgroundColor = BackgroundColor;
// 点击cell的时候不要变色
self.selectionStyle = UITableViewCellSelectionStyleNone;    
// 设置标题cell
[self setUpCell];
}
return self;
}
```

赋值：setFramModel，当然你也可以像猿题库里面那样自己写一个方法进行赋值都是可以的
```
- (void)setFrameModel:(JBSystemMessageFrameModel *)frameModel {  
_frameModel = frameModel; 
self.container.frame = frameModel.containerFrame;
// 标题
self.titleLabel.text = frameModel.messageModel.title;
self.titleLabel.frame = frameModel.titleLabelFrame;
// 时间-作者
self.timeLabel.frame = frameModel.timeLabelFrame;
// 内容
self.contentLabel.text = frameModel.messageModel.content;
self.contentLabel.frame = frameModel.contentLabelFrame;
// 查看详情
self.moreView.frame = frameModel.moreViewFrame;
}

- (void)moreInformation {   
if (self.delegate && [self.delegate respondsToSelector:@selector(moreInformation:)]) {
[self.delegate moreInformation:self.frameModel];
}
}
```



### 3.VM层

VM层即ViewModel，就是处理API获取的数据转化成界面展示的模型数据。

```
// 控件与cell之前的顶部间距
#define kJBMessageCellTopMargin           20
// 控件之间的顶部间距；
#define kJBMessageTopMargin               10
// 控件与cell之间的左右间距；
#define kJBMessageCellLeftMargin          15
// 标题高度
#define kJBMessageTitleHeight             14
// 时间和作者高度
#define kJBMessageTimeHeight              10
// 查看详情高度
#define kJBMessageInfoHeight              35
// 标题字体
#define kJBMessageCellTitleFont     [UIFont systemFontOfSize:15]
// 时间和作者字体
#define kJBMessageCellTimeFont      [UIFont systemFontOfSize:10]
// 内容字体
#define kJBMessageCellSourceFont    [UIFont systemFontOfSize:12]

@interface JBSystemMessageFrameModel : NSObject

@property (nonatomic, strong) JBSystemMessageModel *messageModel;

/** cell展示容器的Frame */
@property (nonatomic, assign) CGRect containerFrame;

/** 标题的Frame */
@property (nonatomic, assign) CGRect titleLabelFrame;

/** 时间／作者的Frame */
@property (nonatomic, assign) CGRect timeLabelFrame;

/** 内容的Frame */
@property (nonatomic, assign) CGRect contentLabelFrame;

/** 查看详情的Frame */
@property (nonatomic, assign) CGRect moreViewFrame;

/** cell的高度 */
@property (nonatomic, assign) CGFloat cellHeight;

/** cell是否展开 */
@property (nonatomic, assign) BOOL isShowMore;
```

在ViewModel的.m文件中，依然是利用重写setMessageModel进行控件的尺寸以及展示数据逻辑等计算和转换。
```
@implementation JBSystemMessageFrameModel

- (void)setMessageModel:(JBSystemMessageModel *)messageModel {    
_messageModel = messageModel;
CGFloat cellWidth = ScreenWidth;
/** 标题 */
CGFloat titleX = kJBMessageCellLeftMargin;
CGFloat titleY = kJBMessageCellLeftMargin;
CGFloat titleH = kJBMessageTitleHeight;
CGSize titleSize = [messageModel.titlesizeWithfont:kJBMessageCellTitleFontmaxSize:CGSizeMake(MAXFLOAT, titleH)];
self.titleLabelFrame = (CGRect){{titleX, titleY}, titleSize};

/** 时间／作者 */
CGFloat timeX = kJBMessageCellLeftMargin;
CGFloat timeY = CGRectGetMaxY(self.titleLabelFrame) +kJBMessageTopMargin;
CGFloat timeH = kJBMessageTimeHeight;
CGSize timeSize = [messageModel.publishedTimesizeWithfont:kJBMessageCellTimeFontmaxSize:CGSizeMake(MAXFLOAT, timeH)];
CGSize authorSize = [messageModel.authorsizeWithfont:kJBMessageCellTimeFontmaxSize:CGSizeMake(MAXFLOAT, timeH)];
CGFloat timeLabelWidth = timeSize.width + kJBMessageTopMargin + authorSize.width;
self.timeLabelFrame = CGRectMake(timeX, timeY, timeLabelWidth, timeH);

/** 内容 */
CGFloat contentX = kJBMessageCellLeftMargin;
CGFloat contentY = CGRectGetMaxY(self.timeLabelFrame) + kJBMessageTopMargin;
CGSize contentSize = [messageModel.contentsizeWithfont:kJBMessageCellSourceFontmaxSize:CGSizeMake(cellWidth -kJBMessageCellLeftMargin * 4, MAXFLOAT)];
CGSize contentOneLineSize = [@"聚保"sizeWithfont:kJBMessageCellSourceFontmaxSize:CGSizeMake(cellWidth -kJBMessageCellLeftMargin * 4,MAXFLOAT)];
CGFloat contentHeight = self.isShowMore ? contentSize.height : contentOneLineSize.height;
self.contentLabelFrame = CGRectMake(contentX, contentY, contentSize.width, contentHeight);

/** 查看详情 */
CGFloat moreViewX = 0;
CGFloat moreViewY = CGRectGetMaxY(self.contentLabelFrame) + kJBMessageTopMargin;
CGFloat moreViewH = kJBMessageInfoHeight;
CGFloat moreViewW = cellWidth - kJBMessageCellLeftMargin * 2;
self.moreViewFrame = CGRectMake(moreViewX, moreViewY, moreViewW, moreViewH);

/** 容器 */
CGFloat contrainerX = kJBMessageCellLeftMargin;
CGFloat contrainerY = kJBMessageCellTopMargin;
CGFloat contrainerW = moreViewW;
CGFloat contrainerH = CGRectGetMaxY(self.moreViewFrame);
self.containerFrame = CGRectMake(contrainerX, contrainerY, contrainerW, contrainerH);

/** cell高度 */
self.cellHeight = contrainerY + contrainerH;
}
```


现在View部分的cell视图有了，Model模型和ViewModel展示数据模型都有了，Controller该怎么写呢？毕竟前面我说过，我认为Controller只是一个胶水而已，怎么才能不导致回到MVC（Massive Controller）呢？在这里我借鉴[猿题库 iOS 客户端架构设计](http://mp.weixin.qq.com/s?__biz=MjM5NTIyNTUyMQ==&amp;mid=444322139&amp;idx=1&amp;sn=c7bef4d439f46ee539aa76d612023d43&amp;scene=0#wechat_redirect)在中间引入一个DataSerVice来对Controller进行瘦身，并达到对每个模块解耦并可单独测试。

##### DataService部分的代码
```
#import <Foundation/Foundation.h>
#import "JBSystemMessageModel.h"

typedef void(^JBCompletionCallback) (id_Nonnull callback);

@interface JBSystemMessageDataService : NSObject
@property (nonatomic, strong, nonnull, readonly) NSMutableArray<JBSystemMessageFrameModel *> *systemMessageArray;

// 获取系统消息
- (void)requestSystemMessageDataWithCallback:(nonnull JBCompletionCallback)callback;

// 更新数据模型
- (void)updateModel:(JBSystemMessageFrameModel *_Nonnull)frameModel callback:(nonnull JBCompletionCallback)callback;

@end
```

这里要注意一下，对于给外部暴露的systemMessageArray这个数组最好在生命属性的时候加上readonly，因为dataService是专门处理数据的，数据不应该在其他任何外部地方被修改，做到各司其职。
```
@interface JBSystemMessageDataService ()
@property (nonatomic, strong, nonnull) NSMutableArray<JBSystemMessageFrameModel *> *systemMessageArray;
@end

@implementation JBSystemMessageDataService

- (NSMutableArray<JBSystemMessageFrameModel *> *)systemMessageArray {
if (!_systemMessageArray) {
_systemMessageArray = [NSMutableArrayarray];
}
return_systemMessageArray;
}

- (void)requestSystemMessageDataWithCallback:(JBCompletionCallback)callback {

JBWeakSelf;
[JBHttpTool get:JBSystemMessageInfoparameters:nilsuccess:^(id json) {

callback(...);

} failure:^(NSError *error) {
callback(...);
}];
}


- (void)updateModel:(JBSystemMessageFrameModel *_Nonnull)frameModel callback:(nonnullJBCompletionCallback)callback {

JBSystemMessageFrameModel *newFrameModel = [JBSystemMessageFrameModel new];
newFrameModel.isShowMore = frameModel.isShowMore;
newFrameModel.messageModel = frameModel.messageModel;

NSInteger index = [self.systemMessageArrayindexOfObject:frameModel];
[self.systemMessageArrayreplaceObjectAtIndex:index withObject:newFrameModel];

callback(@(index));
}
```
```
requestSystemMessageDataWithCallback是请求API并对请求的结果进行回调。当然这里只是进行简单展示一下，你也可以进行自己的理解和设计把数据请求这一块单独做一个处理并供整个项目使用，这里就不再累述。    
- (void)updateModel:(JBSystemMessageFrameModel *_Nonnull)frameModel callback这个方法就是对数据进行更新处理，这里的场景是点击某一个cell的时候cell内部会展开，并对当前cell的数据模型进行更新处理。
```


现在DataService部分已经进行简单展示了，Controller就很好处理：进行简单的胶水黏合作用。
```
#import "JBSystemMessageController.h"
#import "JBSystemMessageCell.h"
#import "JBSystemMessageDataService.h"

@interface JBSystemMessageController () <JBSystemMessageDelegate>
@property (nonatomic,strong, nullable) JBSystemMessageDataService *dataService;
@end

@implementation JBSystemMessageController

- (JBSystemMessageDataService *)dataService {
if (!_dataService) {
_dataService = [[JBSystemMessageDataServicealloc] init];
}
return_dataService;
}

- (void)viewDidLoad {
[superviewDidLoad];

[selfsetUpData];

[selfsetUpTableView];
}

- (void)setUpTableView {
self.tableView.backgroundColor =BackgroundColor;
self.tableView.separatorStyle = UITableViewCellSeparatorStyleNone;
self.tableView.estimatedRowHeight = 0;
self.tableView.estimatedSectionHeaderHeight = 0;
self.tableView.estimatedSectionFooterHeight = 0;
self.tableView.showsVerticalScrollIndicator = NO;
}

- (void)setUpData {

JBWeakSelf;
[self.dataServicerequestSystemMessageDataWithCallback:^(id _Nonnull callback) {
JBIsSuccess(callback) ? [weakSelf.tableViewreloadData] : [MBProgressHUDshowError:callback];
}];
}

- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {

return 1;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {

return self.dataService.systemMessageArray.count;
}

- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {

return self.dataService.systemMessageArray[indexPath.row].cellHeight;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {

JBSystemMessageCell *systemMessageCell = [JBSystemMessageCellcellWithTableView:tableView];

systemMessageCell.frameModel = self.dataService.systemMessageArray[indexPath.row];
systemMessageCell.delegate = self;

return systemMessageCell;
}

#pragma mark- cell点击代理
- (void)moreInformation:(JBSystemMessageFrameModel *)frameModel {

JBWeakSelf;
[self.dataServiceupdateModel:frameModel callback:^(id  _Nonnull callback) {
// 模型在数组中的索引
NSInteger index = [callback integerValue];   
[weakSelf.tableView beginUpdates];
[weakSelf.tableViewreloadRow:index inSection:0withRowAnimation:UITableViewRowAnimationAutomatic];
[weakSelf.tableView endUpdates];
[weakSelf.tableViewscrollToRow:index inSection:0atScrollPosition:UITableViewScrollPositionBottomanimated:YES];
}];
}
```

初始化数据，初始化tableView， 然后复制tableView数据源 并 走一下cell的代理方法就OK了～ 其他的controller根本不需要管，Model和ViewModel压根就跟Controller没有半毛钱的关系，头文件都不需要倒入，controller真正关心的只是View和DataService两个；dataService关心的只是ViewModel界面展示数据的处理。而ViewModel关心的只是Model层数据结构。这样进行设计架构，不仅仅对controller进行了瘦身，各个部分也进行了解耦，另外这么设计也有一个好处就是各个部分可以进行相应的复用，而且项目的维护（特别是新来的接手别人的“杰作”的时候，应该还算比较酸爽的，不至于像以前那样抱怨：这谁写的啊，看他代码我还不如删了自己重新写）起来也是比较方便的。



## 总结：

以上所展示的算是MVVM的一种改良吧，借鉴猿题库的架构思想。当然，每个人可能对MVVM都有自己的理解，可以根据自己的理解进行设计出合理的框架，我在这里只是做一个抛砖引玉，简单的展示下。大家可以自己发挥，如果有好的建议，可以留言进行探讨，另外理解错误的地方希望大家斧正～

这里稍微有点懒了，开年工作忙，所以贴了很多代码， 希望大家谅解。

