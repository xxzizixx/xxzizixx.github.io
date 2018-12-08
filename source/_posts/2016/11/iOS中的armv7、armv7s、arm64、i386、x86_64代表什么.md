---
title: iOS中的armv7、armv7s、arm64、i386、x86_64等指令集的作用
date: 2016-11-08 10:17:12
tags: 
    - 求学之路
---

<br/>
<br/>

# 前序
我们常常在开发中看到一些熟悉的字眼armv7、armv7s、arm64、i386、x86_64，但是又都不太清楚他们具体表达的什么，还时常会弄混淆。那么，他们到底是什么呢？我们在Xcode中又该如何选择？
<br>
<br>



# 概念
### arm
arm处理器，特点是体积小、低功耗、低成本、高性能，所以几乎所有手机处理器都基于arm，在嵌入式系统中应用广泛。


### arm处理器指令集
armv6｜armv7｜armv7s｜arm64都是ARM处理器的指令集，这些指令集都是向下兼容的，例如armv7指令集兼容armv6，只是使用armv6的时候无法发挥出其性能，无法使用armv7的新特性，从而会导致程序执行效率没那么高。


### Mac处理器的指令集
i386｜x86_64 是Mac处理器的指令集，i386是针对intel通用微处理器32架构的。x86_64是针对x86架构的64位处理器。所以当使用iOS模拟器的时候会遇到i386｜x86_64，iOS模拟器没有arm指令集。


### iOS移动设备指令集
**arm64  ：** iPhone5S｜ iPad Air｜ iPad mini2(iPad mini with Retina Display)
**armv7s：** iPhone5｜iPhone5C｜iPad4(iPad with Retina Display)
**armv7  ：** iPhone3GS｜iPhone4｜iPhone4S｜iPad｜iPad2｜iPad3(The New iPad)｜iPad mini｜iPod Touch 3G｜iPod Touch4
**armv6  ：** iPhone, iPhone2, iPhone3G, 第一代、第二代 iPod Touch（一般不需要去支持）
> **注：arm64是真机64位处理器，armv7s、armv7、armv6是真机32位处理器**
> iPhone5s的cpu是64位，但是可以兼容32位代码，即可以运行32位代码。但是这会降低iPhone5s的性能。 其余的iPhone对32位代码包更没问题， 而32位代码包，对系统也几乎也没什么限制。

<!--|   指令集           |     设备                 |  处理器 |-->
<!--| :-------------: | :-------------------: |  :-------------:  | -->
<!--|   arm64   |    iPhone5S｜ iPad Air｜ iPad mini2(iPad mini with Retina Display)     |   64位     |-->
<!--| armv7s   |    iPhone5｜iPhone5C｜iPad4(iPad with Retina Display)  |     32位     |-->
<!--| armv7     |   iPhone3GS｜iPhone4｜iPhone4S｜iPad｜iPad2｜iPad3(The New iPad)｜iPad mini｜iPod Touch 3G｜iPod Touch4          |   32位     |-->
<!--| armv6     |    iPhone, iPhone2, iPhone3G, 第一代、第二代 iPod Touch（一般不需要去支持）|       |-->





### iOS模拟器设备指令集
**x86_64：** iPhone6以上的模拟器
**i386：** iPhone5、iPhone5s以下的模拟器
> **注：x86_64是模拟器64位处理器，i386是模拟器32位处理器**



<br>
<br>

# Xcode中指令集相关选项
### Architectures
> Space-separated list of identifiers. Specifies the architectures (ABIs, processor models) to which the binary is targeted. When this build setting specifies more than one architecture, the generated binary may contain object code for each of the specified architectures.  
 
指定工程被编译成可支持哪些指令集类型，而支持的指令集越多，就会编译出包含多个指令集代码的数据包，对应生成二进制包就越大，也就是ipa包会变大。

### Valid Architectures
> Space-separated list of identifiers. Specifies the architectures for which the binary may be built. During the build, this list is intersected with the value of ARCHS build setting; the resulting list specifies the architectures the binary can run on. If the resulting architecture list is empty, the target generates no binary.   

限制可能被支持的指令集的范围，也就是Xcode编译出来的二进制包类型最终从这些类型产生，而编译出哪种指令集的包，将由Architectures与Valid Architectures（因此这个不能为空）的交集来确定



<!--原因解释如下： -->
<!--使用 standard architectures (including 64-bit)(armv7,arm64) 参数，则打的包里面有32位、64位两份代码，在iPhone5s（ iPhone5s的cpu是64位的 ）下，会首选运行64位代码包， 其余的iPhone（ 其余iPhone都是32位的,iPhone5c也是32位 ），只能运行32位包，但是包含两种架构的代码包，只有运行在ios6，ios7系统上。 -->
<!--这也就是说，这种打包方式，对手机几乎没要求，但是对系统有要求，即ios6以上。 -->
<!--而使用 standard architectures (armv7,armv7s) 参数， 则打的包里只有32位代码， iPhone5s的cpu是64位，但是可以兼容32位代码，即可以运行32位代码。但是这会降低iPhone5s的性能。 其余的iPhone对32位代码包更没问题， 而32位代码包，对系统也几乎也没什么限制。 -->
<!--所以总结如下：  -->
<!--要发挥iPhone5s的64位性能，就要包含64位包，那么系统最低要求为ios6。 如果要兼容ios5以及更低的系统，只能打32位的包，系统都能通用，但是会丧失iPhone5s的性能。-->



### Build Active Architecture Only
> In the Build settings for the target there is an additional option in the Architectures section named “Build Active Architecture Only” which influences how the binary is built.
YES - If set to yes, then Xcode will detect the device that is connected, and determine the architecture, and build on just that architecture alone.
NO - If set to no, then it will build on all the architectures.

影响二进制文件的构建方式：指定是否只对当前连接设备所支持的指令集编译。当其值设置为YES，这个属性设置为yes，是为了debug的时候编译速度更快，它只编译当前的architecture版本，而设置为no时，会编译所有的版本。 编译出的版本是向下兼容的，连接的设备的指令集匹配是由高到低（arm64 > armv7s > armv7）依次匹配的。比如你设置此值为yes，用iphone4编译出来的是armv7版本的，iphone5也可以运行，但是armv6的设备就不能运行。  所以，一般debug的时候可以选择设置为yes，release的时候要改为no，以适应不同设备。


### Architectures和Valid Architectures结合实例

(1)
Architectures:  armv7
Valid Architectures:  armv7、armv7s、arm64
生成二进制包支持的指令集： armv7s
<br>

(2)
Architectures:  armv7、armv7s
Valid Architectures:  armv7s、arm64
生成二进制包支持的指令集： armv7s
<br>

(3)
Architectures:  armv7, armv7s, arm64
Valid Architectures:  armv6, armv7s, arm64
生成二进制包支持的指令集： arm64
<br>

(4)
Architectures: armv6, armv7, armv7s
Valid Architectures:  armv6, armv7s, arm64
生成二进制包支持的指令集： armv7s 
<br>

(5)
Architectures: armv7, armv7s, arm64
Valid Architectures: armv7，armv7s
这种情况是报错的，因为允许使用指令集中没有arm64。
<br>

原因解释如下： 
使用 standard architectures (including 64-bit)(armv7,arm64) 参数，则打的包里面有32位、64位两份代码，在iPhone5s（ iPhone5s的cpu是64位的 ）下，会首选运行64位代码包， 其余的iPhone（ 其余iPhone都是32位的,iPhone5c也是32位 ），只能运行32位包，但是包含两种架构的代码包，只有运行在ios6，ios7系统上。 
这也就是说，这种打包方式，对手机几乎没要求，但是对系统有要求，即ios6以上。 
而使用 standard architectures (armv7,armv7s) 参数， 则打的包里只有32位代码， iPhone5s的cpu是64位，但是可以兼容32位代码，即可以运行32位代码。但是这会降低iPhone5s的性能。 其余的iPhone对32位代码包更没问题， 而32位代码包，对系统也几乎也没什么限制。 
所以总结如下：  
**要发挥iPhone5s的64位性能，就要包含64位包，那么系统最低要求为ios6。如果要兼容ios5以及更低的系统，只能打32位的包，系统都能通用，但是会丧失iPhone5s的性能。**

> 在Xcode里的 Valid Architectures  设置里， 默认为 Standard architectures(armv7,armv7s,arm64),如果你想改的话，自己在other中更改。
> 如果你对ipa安装包大小有要求，可以减少安装包的指令集的数量，这样就可以尽可能的减少包的大小。当然这样做会使部分设备出现性能损失，当然在普通应用中这点体现几乎感觉不到，至少不会威胁到用户体检。

<br>
<br>


<!--# 制作静态库指令集选择-->
<!--如何制作一个兼容性良好的.a静态库？-->
<!--通过以上信息了解到，当我们做App的时候，为了追求高效率，并且减小包的大小，Build Active Architecture Only设置成YES，Architectures按Xcode默认配置就可以，**因为arm64向前兼容**。-->
<!--但制作.a静态库就不同了，因为要保证兼容性，包括不同iOS设备以及模拟器运行不出错，所以结合当前行业情况，要做到最大的兼容性。-->
<!---->
<!--**ValidArchitectures设置为：armv7｜armv7s｜arm64｜i386｜x86_64**-->
<!---->
<!--**Architectures设置不变（或根据你需要）:  armv7｜arm64**-->
<!---->
<!--然后分别选择iOS设备和模拟器进行编译，最后找到相关的.a进行合包，使用lipo -create 真机库.a的路径 模拟器库.a的的路径 －output 合成库的名字.a。这样就制作了一个通用的静态库.a-->



