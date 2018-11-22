---
title: Xcode Template：工程模版、子工程模版、类文件模版
date: 2018-11-16 14:57:41
tags: 
    - 求学之路
---

# 一、前序
一直准备写一篇关于Xcode Template模版制作的文章，昨天听同事提到Xcode Template，并决定抽时间写一篇关于Xcode Template制作的文章，文章包括三部分：
+ 为什么要定义这些模版
+ Xcode工程模版和子工程模版
+ Xcode的类文件模版


# 二、为什么要定义这些模版
遵守代码规范可以提高代码可读性, 降低后期维护成本. 当我们定下了一个团队都认同的代码规范, 如我们要求所有的viewController的代码都得按照下面来组织:
```
#pragma mark - define / static
#pragma mark - override
#pragma mark - api request
#pragma mark - view config
#pragma mark - model event 
#pragma mark - private
#pragma mark - datasource
#pragma mark - delegate
#pragma mark - getter / setter

```
那么，在查找的时候,你就会发现会多么的爽 <br/>
那么，以后看同事的代码的时候，再也不用偷偷跟小伙伴吐槽又接了一个烂尾楼了
![](https://user-gold-cdn.xitu.io/2018/11/16/1671b8d8a8417490?w=972&h=939&f=jpeg&s=188407)
看起来是不是很清晰明了？
那么下面的常用类模版制作就是解决这些问题的; <br>
另外,在阅读别人代码的时候，你是不是也常常找不到某个类在那个文件夹里面？<br> 别方，下面的工程模版和子工程模版就是为了解决这个问题的。


# 三 Xcode工程模版和子工程模版

Xcode自带的工程创建和子工程创建：
![](https://user-gold-cdn.xitu.io/2018/11/16/1671b9684edb2d7f?w=929&h=588&f=jpeg&s=57718)
自定义模版之后的创建和子工程创建：
![](https://user-gold-cdn.xitu.io/2018/11/16/1671b95c8297b8da?w=927&h=597&f=jpeg&s=54494)

看到下面这两个“皮卡丘”了没，没错，这就是自定义模版之后的App工程创建和ChildProject子工程创建，以后创建的时候就可以直接选择这两个，不用在选择系统Xcode自带的了，那么怎么创建呢？<br> 下面我们要先来看一下几张图，然后再去制作模版。<br>
这里是我创建的两个模版，这个文件在你安装的Xcode里面，路径是：
>/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Xcode/Templates/Project Templates/iOS/Application 

 注：这里有两种方式去制作模版：1、可以直接修改系统的相应模版；2、复制系统的模版，然后修改模版名和绑定的ID，制作属于自己的模版。  <br>
建议：复制模版出来，然后自己修改，另外直接在xcode里面修改文件是不方便的，有些可能是只读权限，所以吧模版复制出来修改完后替换Xcode自带的就行。

![](https://user-gold-cdn.xitu.io/2018/11/16/1671b9ebf0a633a0?w=1366&h=444&f=jpeg&s=74942)

图中除了SMT-Application之外，其他的都是Xcode自带的模版，里面能对应上创建工程时操作面板上的所有图标。这里我们只以Single View APP为例说明。

![](https://user-gold-cdn.xitu.io/2018/11/16/1671bae1dae42865?w=1127&h=726&f=jpeg&s=162071)

+ 首先要修改下“Identifier”字段，以防冲突。
> Xcode模板是支持继承的，或者叫做Import，这里Ancestors字段以数组的方式列出了要继承的对象，上面只继承com.apple.dt.unit.cocoaTouchApplicationBase，这个是和Xcode6一起的Empty工程类似的。
而Signal View工程则继承了com.apple.dt.unit.storyboardApplication"和com.apple.dt.unit.coreDataCocoaTouchApplication.而com.apple.dt.unit.storyboardApplication又是继承了上面的com.apple.dt.unit.cocoaTouchApplicationBase。 具体其定义了什么，可以对比着看下，在系统的工程模板位置，比如“Main.storyboard”是如何别加入工程的。

+ OC的用Objective-C字段:每个Node是一个新工程中的具体物理文件位置，目录自动创建，如果没有在后面的Definitions中指定，则吧路径中的最后的的文件加入到工程的代码跟目录下。
+ Definitions.Xxx :这里Xxx就是上面的Node的值，其实一个字典类型，主要有两个键：
Path: 模板文件（也就是要被拷贝的文件）所在工程模板中的位置
Group: 新工程中，这个文件所在的Group

Xcode模版的继承结构：
![](https://user-gold-cdn.xitu.io/2018/11/16/1671bb0cb4554e52?w=1527&h=922&f=jpeg&s=92718)


文件夹配置模版结构：
![](https://user-gold-cdn.xitu.io/2018/11/16/1671bc1e0b2b7bfb?w=740&h=456&f=jpeg&s=65976)

占位符含义：

占位符                          |   意义
--------                        |   ---
___FILENAME___                  |   当前的文件名
___PROJECTNAME___               |   当前工程名，在创建工程时设置的
___FULLUSERNAME___              |   当前登录用户
___DATE___                      |  当前日期 ，格式为MM/DD/YY
___FILEBASENAMEASIDENTIFIER___  |   不带后缀的文件名


当然这里的工程目录文件结构如何配置，看个人喜好来决定，你可以配置.gitignore、info.plist、子工程的Resource放资源文件 等等， 看你的需求。这样创建出来的工程就是你配置后期望的工程目录以及一些你希望做的初始化配置。

简单配置模版之后创建的子工程：
![](https://user-gold-cdn.xitu.io/2018/11/16/1671bd30b7fa0dbf?w=447&h=446&f=jpeg&s=29339)

其实如果能有更多的时间，做一份通用模版出来，对项目开发尤其是多人的组件化开发，创建子工程和文件，统一文件夹和代码格式，将变得非常便利。


# 四、Xcode的类文件模版

可能我们平常用得最多的就是创建文件了， 那么如何去制作文件模版呢？ 其实制作文件模版的过程和子工程的基本一样，下面只说一下不同的部分；
子工程模版位置：
> /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Xcode/Templates/File Templates/Source

模版目录：

![](https://user-gold-cdn.xitu.io/2018/11/16/1671bd83fa19afb1?w=1121&h=682&f=jpeg&s=107283)

模版文件：

![](https://user-gold-cdn.xitu.io/2018/11/16/1671bda6fe368f73?w=1015&h=914&f=jpeg&s=86135)

这里的模版是只写了几个#pragma mark，你可以随意定制一些代码规范以及想调用的方法都可以，看个人定制，只要你愿意，你甚至可以在每个文件里面写“我最帅”...haha

文件模版可以不做太多的新增文件，Xcode默认提供的类型我们修改下就基本足够了，当然如果你想有更多的姿势，可以新增然后修改下TemplateInfo.plist文件

然后在工程里面，当我们要创建文件的时候，选择自己的Cocoa Touch Class， 创建出来的就是你期待的文件模版：

![](https://user-gold-cdn.xitu.io/2018/11/16/1671be15d5859c67?w=746&h=542&f=jpeg&s=45222)

<br>
<br>
<br>

### 好了，到这里基本操作就说完了，还是需要自己去操作一边，或者团队有大佬创建一份作为规范模版，分发给我们，然后直接拖入Xcode，那就真的是爽歪歪了～  
