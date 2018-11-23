---
title: CocoaPods建立自己的仓库
date: 2018-10-14 19:11:15
tags: 
    - 求学之路
---

<br/>
<br/>

# 前序
作为iOS开发，无论你是小白还是远古时代MRC的大神，相信没有人不知道CocoaPods，如果不知道的话，那么你应该去面壁十分钟怀疑自己了～ 
github几乎所有（或者说全部）优秀的iOS开源框架都提供了Cocoapods Installation方式，由此可见，CocoaPods的使用为我们开发带来了极大的便利。
那么问题来了：**我们改如何去构建自己的Cocoapods仓库呢？**
<br>
<br>

# CocoaPods的本地目录
先在终端运行下面命令显示隐藏文件
```
$ defaults write com.apple.finder AppleShowAllFiles -boolean true ; killall Finder
```
然后输入命令查看CocoaPods本地目录
```
$ cd ~/.cocoapods/repos/master
// 打开~/.cocoapods/repos/master文件夹
$ open ../
```
然后你就会看到如下面的图：
![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxhs4t2hs0j31gg0k1q9v.jpg)
图中的master文件夹是github上面的仓库的库的检索文件；另外领个是公司内部gitLab上面自建的库文件：
* 一个是gitmirror，他是项目中使用到github上面的库文件的镜像podspec检索文件，关于git mirror的相关知识可以自行google。
* 另一个是为组件化开发自建的Cocoapods库，目的是封装相关功能模块方便其他人pod开发。可见在组件化开发中Cocoapods自建仓库已经是基本操作了。    


**Specs目录**：Specs目下并不是直接是以库的名称命名的文件夹，而是分了3层目录，分别以0-f来命名。比如AFNetworking的位置是: a/7/5/AFNetworking/xx/AFNetworking.podspec。
后面的xx就是版本号了，也就是我们打的tags。

三层文件夹的排列方式就是使用MD5库名，然后取MD5后的前三位分别做为三层目录；这样做的目的是提高库的检索速度来提高Cocoapods的效率。
如下命令可以输出MD5根据规则在相对应文件夹下查找对应的库podspec文件：
```
$ md5 -s "AFNetworking"
// 输出
MD5 ("AFNetworking") = a75d452377f3996bdc4b623a5df25820
```
```
md5 -s "SDWebImage"
// 输出
MD5 ("SDWebImage") = 1173b6117a2cf4a6756f761aedae9d2c
```
```
md5 -s "YYKit"
// 输出
MD5 ("YYKit") = a408542eb2fcd3c332bf83b515cdcf03
```

#### pod搜索Specs文件夹中的框架输出框架信息
```
$ pod search AFNetworking
```
会看到iTerm输出以下信息：

可以点击查看[AFNetworking/AFNetworking.podspec](https://github.com/AFNetworking/AFNetworking/blob/master/AFNetworking.podspec)进行对比：
![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxht2cy2d8j30sl0rwafi.jpg)

<br/>
我们在 CocoaPods 发布我们的框架时，就是要在 master 仓库中添加我们的仓库描述信息，然后push到远程仓库中。不过这个过程不用我们手动去操作，只需要通过pod命令进行操作即可。

<br/>
<br/>

# 构建github上的Cocoapods公有仓库
### 注册 CocoaPods 账号
想创建开源的Pod库，就要注册一个CocoaPods账号，我们使用终端注册, email 用你的 GitHub 邮箱
```
$ pod trunk register GitHub_email 'user_name' --verbose
```
等终端出现下面文字，CocoaPods 会发一个确认邮件到你的邮箱上，登录你的邮箱进行确认。
```
[!] Please verify the session by clicking the link in the verification email that has been sent to you_email@163.com
```
![](https://ws2.sinaimg.cn/large/006tNbRwgy1fxht8jhwbwj30r10h30ud.jpg)
注册成功！

确认后再终端输入
```
pod trunk me
```
![](https://ws1.sinaimg.cn/large/006tNbRwgy1fxht9rw5ihj30oq0d0wge.jpg)

<br/>
### 创建Git仓库
在 GitHub 上创建一个公开项目，项目中必须包含这几个文件:
* LICENSE:开源许可证
* README.md:仓库说明
* 你的代码
* **PAImagePickerModule.podspec: CocoaPods 的描述文件，这个文件非常重要**
<br>

### 创建.podspec
.podspec 是用 Ruby 的配置文件，描述你项目的信息。

在你的仓库目录下，使用终端命令创建
```
$ pod spec create PAImagePickerModule
```
这时就会在你的仓库下生成 PAImagePickerModule.podspec 文件
![](https://ws4.sinaimg.cn/large/006tNbRwgy1fxhtk4u9jxj30pw0lwn0f.jpg)
<br/>


根据要求修改配置文件，里面都有注视，但是很多配置都不是必须的，我们只填写必要的信息就行了。
我这里列出了主要的配置，可以参考这个进行修改：
![](https://ws4.sinaimg.cn/large/006tNbRwgy1fxhtmm2c6mj30me0isgp0.jpg)

<br/>
**下面就是最关键的一步了：验证 .podspec 文件的格式是否正确**

```
$ pod lib lint 
```
如果验证通过会显示：
```
-> PAImagePickerModule (0.0.4)

PAImagePickerModule passed validation.    
```
但是很多情况下并没有这么顺利，比如:
```
-> PAImagePickerModule (0.0.4)
- WARN  | url: There was a problem validating the URL http://xxzizixx.github.io.

[!] PAImagePickerModule did not pass validation, due to 1 warning (but you can use `--allow-warnings` to ignore it) and all results apply only to public specs, but you can use `--private` to ignore them if linting the specification for a private pod.
...
...
You can use the `--no-clean` option to inspect any issue.
```
提示我们需要加--allow-warnings这么一句话，意思就是允许警告，命令改为:
```
$ pod lib lint --allow-warnings
```
### 给仓库打标签
验证成功后，将仓库提交到远程，然后给仓库打上标签并将标签也推送到远程。

标签相当于将你的仓库的一个压缩包，用于稳定存储当前版本。标签号与你在 s.version = "0.0.4"的版本号一致 0.0.4
```
查看标签
$ git tag
创建标签
$ git tag -a 0.0.4 -m '标签说明' 
删除标签
$ git tag -d 0.0.4
推送到远程
$ git push origin --tags
删除远程tag
git push origin --delete tag 0.1.0
```
<br>

### 发布.podspec
到这里说明到最后一步了，就是发布项目的描述的文件PAImagePickerModule.podspec

在仓库目录下执行：
```
pod trunk push PAImagePickerModule.podspec
```
如果像之前那样因为警告等原因报错了，那就在后面加参数重新执行：
```
pod trunk push PAImagePickerModule.podspec --allow-warnings
```

将你的 PAImagePickerModule.podspec 发布到公有的speecs上,这一步其实做了很多操作,包括:
* 更新本地 pods 库 ~/.cocoaPods.repo/master
* 验证.podspec格式是否正确 
* 将 .podspec 文件转成 JSON 格式
* 对 master 仓库 进行合并、提交.master仓库地址

成功后将会出现下列信息：
```
Updating spec repo `master`
Validating podspec
-> PAImagePickerModule (0.0.4)

Updating spec repo `master`

--------------------------------------------------------------------------------
🎉  Congrats

🚀  PAImagePickerModule (0.0.4) successfully published
📅  Oct. 14th, 01:39
🌎  https://cocoapods.org/pods/PAImagePickerModule
👍  Tell your friends!
```
说明发布成功，你就可以通过上面的URL: https://cocoapods.org/pods/PAImagePickerModule 进入的Pods查看自己的仓库信息了.
![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxhwmhkhfnj30no0hu0u4.jpg)
<br>

### 使用仓库
发布到Cocoapods后，在终端更新本地pods仓库信息
```
$ pod setup
```

然后搜索刚刚提交的仓库:
```
$ pod search PAImagePickerModule ---
-> PAImagePickerModule (0.0.4)
A delightful TextField of PhoneNumber
pod 'PAImagePickerModule', '~> 0.0.4'
- Homepage: https://github.com/xxzizixx/PAImagePickerModule
- Source:   https://github.com/xxzizixx/PAImagePickerModule.git
- Versions: 0.0.4, 0.0.1 [PAImagePickerModule repo]
(END)
```

若出现仓库信息说明已经成功了，这时候你就可以在 Podfile 添加:pod 'PAImagePickerModule', '~> 0.0.4',愉快的使用自己的仓库了。
<br>
<br>


# 在内网创建私有CocoaPods仓库
前面已经说了，在多人团队开发采用组件化方案的时候，就免不了的在公司内网创建私有的CocoaPods仓库，其实在内网gitlab上创建私有的pod库和在github上面常见的步骤基本差不多，这里只说几个注意点。
首先在iTerm中输入：
```
pod repo
```
会列出当前Cocoapods关联的仓库，如下图：
![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxhy9ki2lmj30oq0eg76r.jpg)
然后我们在写**内网specs文件**的时候，别忘了相关地方替换下：
```
把这里的https://github.com/xxzizixx都换成内网地址
s.homepage     = "https://github.com/xxzizixx/PAImagePickerModule"
s.source       = { :git => 'https://github.com/xxzizixx/PAImagePickerModule.git', :tag => s.version.to_s }
```
<br/>

**在发布.podspec描述文件的时候，内网选择使用：**
```
pod repo push "内网仓库名" PAImagePickerModule.podspec --allow-warnings
```
其实这个理解可以跟git branch分支推送差不多的意思，cocoapods默认的trunk是github的master，我们在push .podspec描述文件的时候需要指定推送的仓库名。
<br>
<br>


# Cocoapod库的更新：
关于pod库的更新，按照下面三个步骤就行：
* 修改pod库完毕后（**.podspec也属于pod库的一部分，.podspec的version版本号一定要记得修改，版本号要和后面的tags标签对应**），push到目标仓库
* 打标签，对push到目标仓库的版本进行tags版本标记，tags一定要和.podspec里面的version抑制，然后将打好的tags推送到目标仓库
* 发布.podspec文件
注：如果对整个流程不熟悉，避免多次发布.podspec失败，导致多次新增tag版本越来越大，可以先进行.podspec描述文件的验证，通过后再push到目标仓库。
<br>



# 总结
看完了是不是觉得so easy？ 没错，就是这么简单，学会了就赶紧把你私藏的好用的“类库”分享到github上给大家享用吧。
<br/>

**CocoaPods自建仓库是每个iOSer都应该会的基本操作**，如果你还不会，那么赶紧去试试吧～

