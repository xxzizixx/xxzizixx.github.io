---
title: LLVM编译器之Clang前端
date: 2018-09-28 16:24:17
tags: 
    - 求学之路
---
# 前沿

> ### 瞻仰大佬

![](https://user-gold-cdn.xitu.io/2018/9/30/166281ae1b5a9f6a?w=314&h=328&f=jpeg&s=39409)
**<center>Chris Lattner</center>**

### 三大杰作：
>* **Clang**
>* **LLVM**
>* **Swift**

2010年开始编写 Swift语言,而且一个人实现了Swift的大部分基础架构；他也是 LVVM 以及 Clang的主要开发者。  
<br></br>

# 什么是LLVM
[LLVM官网](https://llvm.org/)

* The LLVM Project is a collection of modular and reusable compiler and toolchain technologies.
* LLVM是一个模块化和可重用的编译器和工具链技术的集合。

**作用**：用于优化以任意程序语言编写的程序的**编译时间(compile-time)**、**链接时间(link-time)**、**运行时间(run-time)**以及**空闲时间(idle-time)**.在2000年,Chris Lattner开发了这一套编译器工具库套件.后来随着LLVM的发展,LLVM可以用于常规编译器,JIT编译器,汇编器,调试器,静态分析工具等一系列跟编程语言相关的工作。  

2012年，LLVM 获得**美国计算机学会 ACM** 的软件系统大奖，和 **UNIX，WWW，TCP/IP，Apache，JAVA, Eclipse**等齐名。

**注**：LLVM工程包含了一组模块化，可复用的编辑器和工具链。同其名字原意(Low Level Virtual Machine)不同的是，LLVM不是一个首字母缩写，而是工程的名字。
<br></br>

## Xcode版本的相对应编译器的变迁

|   Xcode版本      |    编译器版本       |
|    :--------:         |    :--------:  |
|   Xcode3之前    |   GCC   |
|   Xcode3           |   GCC与 LLVM混合编译器     |
|   Xcode4           |   LLVM-GCC 成为默认编译器    |
|   Xcode4.2        |   LLVM3.0成为默认编译器          |
|   Xcode5           |   LLVM5.0, 完成 GCC到LLVM的过渡      |

### GCC -> LLVM 简介 
>GCC是 Xcode早期使用的一个强大的编译器.这个编译器被移植到各种系统中,其中就是 Mac OSX 操作系统,所以这就反映在 Xcode中,在早期的 Xcode 调试代码的一个工具就是 GDB,它是GNU调试器.  


### 为什么从GCC变迁到LLVM？
Apple(包括中后期的NeXT)一直使用GCC作为官方的编译器。GCC作为开源世界的编译器标准一直做得不错,但Apple对编译工具会提出更高的要求。
一方面,是Apple对Objective-C语言(甚至后来对C语言)新增很多特性,但GCC开发者并不买Apple的帐——不给实现,因此索性后来两者分成两条分支分别开发,这也造成Apple的编译器版本远落后于GCC的官方版本。另一方面,GCC的代码耦合度太高,不好独立,而且越是后期的版本,代码质量越差,但Apple想做的很多功能(比如更好的IDE支持)需要模块化的方式来调用GCC,但GCC一直不给做。甚至最近,[《GCC运行环境豁免条款(英文版)》](https://blog.delphij.net/2009/01/gcc.html)从根本上限制了LLVM-GCC的开发。 所以,这种不和让Apple一直在寻找一个高效的、模块化的、协议更放松的开源替代品.
<br></br>

### 目前LLVM包含的主要子项目包括:
>1. LLVM Core:包含一个现在的源代码/目标设备无关的优化器，一集一个针对很多主流(甚至于一些非主流)的CPU的汇编代码生成支持。
>2. Clang:一个C/C++/Objective-C编译器，致力于提供令人惊讶的快速编译，极其有用的错误和警告信息，提供一个可用于构建很棒的源代码级别的工具.
>3. dragonegg: gcc插件，可将GCC的优化和代码生成器替换为LLVM的相应工具。
>4. LLDB:基于LLVM提供的库和Clang构建的优秀的本地调试器。
>5. libc++、libc++ ABI: 符合标准的，高性能的C++标准库实现，以及对C++11的完整支持。
>6. compiler-rt:针对__fixunsdfdi和其他目标机器上没有一个核心IR(intermediate representation)对应的短原生指令序列时，提供高度调优过的底层代码生成支持。
>7. OpenMP: Clang中对多平台并行编程的runtime支持。
>8. vmkit:基于LLVM的Java和.NET虚拟机实
>9. polly: 支持高级别的循环和数据本地化优化支持的LLVM框架。
>10. libclc: OpenCL标准库的实现
>11. klee: 基于LLVM编译基础设施的符号化虚拟机
>12. SAFECode:内存安全的C/C++编译器
>13. lld: clang/llvm内置的链接器



<br></br>
# LLVM编译架构
传统的静态编译器分为三个阶段：前端、优化和后端。
![](https://user-gold-cdn.xitu.io/2018/9/30/166296aeeb3c3a13)

**典型例子：GCC编译器， 如何做到解耦？**
![](https://user-gold-cdn.xitu.io/2018/10/8/166528b84aef2c59?w=860&h=74&f=jpeg&s=14953)

### LLVM Three-Phase 编译器器架构:
![](https://user-gold-cdn.xitu.io/2018/9/30/166296b4f9fb0116)
#### 架构优点：
>  不同的前端后端使用统一的中间代码LLVM Intermediate Representation (LLVM IR) 
<br></br>
>  如果需要支持一种新的编程语言，那么只需要实现一个新的前端
<br></br>
>  如果需要支持一种新的硬件设备，那么只需要实现一个新的后端
<br></br>
>  优化阶段是一个通用的阶段，它针对的是统一的LLVM IR，不论是支持新的编程语言，还是支持新的硬件设备，都不需要对优化阶段做修改
<br></br>
>  LLVM现在被作为实现各种静态和运行时编译语言的通用基础结构(GCC家族、Java、.NET、Python、Ruby、Scheme、Haskell、D等)  

<br>  

### Clang/Swift - LLVM 编译器器架构:
![](https://user-gold-cdn.xitu.io/2018/10/8/166528aee2e388d5?w=854&h=487&f=jpeg&s=95419)


**Frontend:前端**
- 词法分析、语法分析、语义分析、生成中间代码

**Optimizer:优化器**
- 中间代码优化

**Backend:后端**
- 生成机器码
<br></br>

作为LLVM提供的编译器前端，clang可将用户的源代码(C/C++/Objective-C)编译成语言/目标设备无关的IR(Intermediate Representation)实现。其可提供良好的插件支持，容许用户在编译时，运行额外的自定义动作。
<br></br>


# Xcode

![](https://user-gold-cdn.xitu.io/2018/10/23/166a03c909b56aad?w=1200&h=579&f=jpeg&s=116720)

<br></br>

# 编译过程
在列出完整步骤之前可以先看个简单例子。看看是如何完成一次编译的。
```
#import <Foundation/Foundation.h>
#define DEFINEEight 8

int main(){
@autoreleasepool {
int eight = DEFINEEight;
int six = 6;
NSString* site = [[NSString alloc] initWithUTF8String:"starming"];
int rank = eight + six;
NSLog(@"%@ rank %d", site, rank);
}
return 0;
}
```

- 查看oc的c实现：
```
$ xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc -fobjc-arc -fobjc-runtime=ios-8.0.0 main.m -o main-arm64.cpp
```
生成的c++文件如下
```
int main(){
/* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
int eight = 8;
int six = 6;
NSString* site = ((NSString * _Nullable (*)(id, SEL, const char * _Nonnull))(void *)objc_msgSend)((id)((NSString *(*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("NSString"), sel_registerName("alloc")), sel_registerName("initWithUTF8String:"), (const char *)"starming");
int rank = eight + six;
NSLog((NSString *)&__NSConstantStringImpl__var_folders_c__8jb7vhc96p1bhvf5gl7zw_9sj925cz_T_main_9c278d_mi_0, site, rank);
}
return 0;
}
static struct IMAGE_INFO { unsigned version; unsigned flag; } _OBJC_IMAGE_INFO = { 0, 2 };
```
<br>  

##  Clang 命令
- Clang 在概念上是编译器前端，同时，在命令行中也作为一个“黑盒”的 Driver。  
- 封装了了编译管线、前端命令、LLVM 命令、Toolchain 命令等，一
个 Clang ⾛走天下。  
- ⽅便从 gcc 迁移过来。

![](https://user-gold-cdn.xitu.io/2018/10/8/1665299812d333a9?w=480&h=220&f=jpeg&s=15434)
例如上面的查看oc的c语言实现，可以利用clang重写objc：
```
$ clang -rewrite-objc mian.m
```

### 利用clang命令查看整个编译过程
```
$ clang -ccc-print-phases main.m
```
- 可以看到编译源文件需要的几个不同的阶段
```
0: input, "main.m", objective-c
1: preprocessor, {0}, objective-c-cpp-output // 预处理    
2: compiler, {1}, ir    // 编译生成IR(中间代码)
3: backend, {2}, assembler  // 汇编器生成汇编代码
4: assembler, {3}, object   // 生成机器码(目标文件)
5: linker, {4}, image   // 链接
6: bind-arch, "x86_64", {5}, image  // 根据运行平台，生成镜像文件(Image)，也就是最后的可执行文件
```

想看清clang前端的全部过程？接下来可以继续通过clang命令查看各阶段都做了哪些处理。

<br>

### 1⃣ Preprocess -预处理
```
$ clang -E main.m
```
```
/*
... 头文件

# 1 "/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.13.sdk/System/Library/Frameworks/Foundation.framework/Headers/FoundationLegacySwiftCompatibility.h" 1 3
# 185 "/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.13.sdk/System/Library/Frameworks/Foundation.framework/Headers/Foundation.h" 2 3
# 2 "main.m" 2

int main(){
@autoreleasepool {
int eight = 8;
int six = 6;
NSString* site = [[NSString alloc] initWithUTF8String:"starming"];
int rank = eight + six;
NSLog(@"%@ rank %d", site, rank);
}
return 0;
}
```

这个过程的处理包括宏的替换，头文件的导入，以及类似#if的处理。

<br>

### 2⃣ Lexical Analysis - 词法分析
- 词法分析，也作 Lex 或者 Tokenization  
- 将预处理理过的代码⽂文本转化成 Token 流
- 不校验语义  
>预处理完成后就会进行词法分析，这里会把代码切成一个个 Token，比如大小括号，等于号还有字符串等。
```
$ clang -fmodules -fsyntax-only -Xclang -dump-tokens main.m
```

<br>

### 3⃣ Semantic Analysis - 语法分析
- 语法分析，在 Clang 中由 Parser 和 Sema 两个模块配合完
成
- 验证语法是否正确
- 根据当前语⾔言的语法，⽣生成语意节点，并将所有节点组合成 抽象语法树(AST)  

如下例子：
```
int testAST(int a, int b) {
while (b != 0) {

if (a > b) {
a = a - b;
} else {
b = b - a;
}
}
return a;
}
```
验证语法是否正确，然后将所有节点组成抽象语法树 AST 。
```
$ clang -fmodules -fsyntax-only -Xclang -ast-dump main.m
```

生成如下语法树：
```
`-FunctionDecl 0x7ffd8f84d678 <main.m:3:1, line:13:1> line:3:5 test 'int (int, int)'
|-ParmVarDecl 0x7ffd8f84d4f8 <col:10, col:14> col:14 used a 'int'
|-ParmVarDecl 0x7ffd8f84d570 <col:17, col:21> col:21 used b 'int'
`-CompoundStmt 0x7ffd8f84dba8 <col:24, line:13:1>
|-WhileStmt 0x7ffd8f84db30 <line:4:5, line:11:5>
| |-<<<NULL>>>
| |-BinaryOperator 0x7ffd8f84d7d8 <line:4:12, col:17> 'int' '!='
| | |-ImplicitCastExpr 0x7ffd8f84d7c0 <col:12> 'int' <LValueToRValue>
| | | `-DeclRefExpr 0x7ffd8f84d778 <col:12> 'int' lvalue ParmVar 0x7ffd8f84d570 'b' 'int'
| | `-IntegerLiteral 0x7ffd8f84d7a0 <col:17> 'int' 0
| `-CompoundStmt 0x7ffd8f84db10 <col:20, line:11:5>
|   `-IfStmt 0x7ffd8f84dad8 <line:6:9, line:10:9>
|     |-<<<NULL>>>
|     |-<<<NULL>>>
|     |-BinaryOperator 0x7ffd8f84d880 <line:6:13, col:17> 'int' '>'
|     | |-ImplicitCastExpr 0x7ffd8f84d850 <col:13> 'int' <LValueToRValue>
|     | | `-DeclRefExpr 0x7ffd8f84d800 <col:13> 'int' lvalue ParmVar 0x7ffd8f84d4f8 'a' 'int'
|     | `-ImplicitCastExpr 0x7ffd8f84d868 <col:17> 'int' <LValueToRValue>
|     |   `-DeclRefExpr 0x7ffd8f84d828 <col:17> 'int' lvalue ParmVar 0x7ffd8f84d570 'b' 'int'
|     |-CompoundStmt 0x7ffd8f84d9a0 <col:20, line:8:9>
|     | `-BinaryOperator 0x7ffd8f84d978 <line:7:13, col:21> 'int' '='
|     |   |-DeclRefExpr 0x7ffd8f84d8a8 <col:13> 'int' lvalue ParmVar 0x7ffd8f84d4f8 'a' 'int'
|     |   `-BinaryOperator 0x7ffd8f84d950 <col:17, col:21> 'int' '-'
|     |     |-ImplicitCastExpr 0x7ffd8f84d920 <col:17> 'int' <LValueToRValue>
|     |     | `-DeclRefExpr 0x7ffd8f84d8d0 <col:17> 'int' lvalue ParmVar 0x7ffd8f84d4f8 'a' 'int'
|     |     `-ImplicitCastExpr 0x7ffd8f84d938 <col:21> 'int' <LValueToRValue>
|     |       `-DeclRefExpr 0x7ffd8f84d8f8 <col:21> 'int' lvalue ParmVar 0x7ffd8f84d570 'b' 'int'
|     `-CompoundStmt 0x7ffd8f84dab8 <line:8:16, line:10:9>
|       `-BinaryOperator 0x7ffd8f84da90 <line:9:13, col:21> 'int' '='
|         |-DeclRefExpr 0x7ffd8f84d9c0 <col:13> 'int' lvalue ParmVar 0x7ffd8f84d570 'b' 'int'
|         `-BinaryOperator 0x7ffd8f84da68 <col:17, col:21> 'int' '-'
|           |-ImplicitCastExpr 0x7ffd8f84da38 <col:17> 'int' <LValueToRValue>
|           | `-DeclRefExpr 0x7ffd8f84d9e8 <col:17> 'int' lvalue ParmVar 0x7ffd8f84d570 'b' 'int'
|           `-ImplicitCastExpr 0x7ffd8f84da50 <col:21> 'int' <LValueToRValue>
|             `-DeclRefExpr 0x7ffd8f84da10 <col:21> 'int' lvalue ParmVar 0x7ffd8f84d4f8 'a' 'int'
`-ReturnStmt 0x7ffd8f84db90 <line:12:5, col:12>
`-ImplicitCastExpr 0x7ffd8f84db78 <col:12> 'int' <LValueToRValue>
`-DeclRefExpr 0x7ffd8f84db50 <col:12> 'int' lvalue ParmVar 0x7ffd8f84d4f8 'a' 'int'
```

![](https://user-gold-cdn.xitu.io/2018/10/8/16651a2c89c25804?w=728&h=674&f=jpeg&s=59588)

<br>

### 4⃣ Static Analysis - 静态分析
- 通过语法树进行代码静态分析，找出非语法性错误
- 模拟代码执行路径，分析出 control-flow graph (CFG)【MRC下会分析出引用计数的错误】
- 预置了常用 Checker（检查器）

### 5⃣ 中间代码生成
- CodeGen 负责将语法树从顶至下遍历，翻译成 LLVM IR
- LLVM IR 是 Frontend 的输出，也是 LLVM Backend 的输 入，前后端的桥接语言
- 与 Objective-C Runtime 桥接
```
$ clang -S -fobjc-arc -emit-llvm main.m -o main.ll
```

到这一步，LLVM前段编译器clang的工作已经基本做完了。


<br>
<br>


# LLVM IR
- LLVM IR有3种表示形式,但本质是等价的: 
- text:便于阅读的文本格式，类似于汇编语言，拓展名.ll
```
$ clang -S -emit-llvm main.m -o main.ll
```
- memory:内存格式
- bitcode:二进制格式，拓展名.bc， 
```
$ clang -c -emit-llvm main.m -o main.bc
```

<br>


## Optimizer优化器
### SSA(Static Single Assignment)静态单一赋值优化
#### 概念
> In compiler design, static single assignment form (often abbreviated as SSA form or simply SSA) is a property of an intermediate representation (IR), which requires that each variable is assigned exactly once, and every variable is defined before it is used.
<br>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; – From Wikipedia

从上面的描述可以看出，SSA 形式的 IR 主要特征是每个变量只赋值一次。相比而言，非SSA形式的IR里一个变量可以赋值多次。


#### 优点
 可以简化很多编译优化方法的过程；<br>
 对很多编译优化方法来说，可以获得更好的优化结果,<br>

下面给出一个例子：
```
int main() {
int x, y;
x = 1;
x = 2;
y = x;
}
```
- 非SSA
```
y := 1
y := 2
x := y
```
> 显然，我们一眼就可以看出，上述代码第一行的赋值行为是多余的，第三行使用的 y 值来自于第二行中的赋值。对于采用非 SSA 形式 IR 的编译器来说，它需要做数据流分析（具体来说是到达-定义分析）来确定选取哪一行的 y 值。但是对于 SSA 形式来说，就不存在这个问题了。如下所示：

- SSA
```
y1 := 1
y2 := 2
x1 := y2
```
> 我们不需要做数据流分析就可以知道第三行中使用的y来自于第二行的定义，这个例子很好地说明了SSA的优势。除此之外，还有许多其他的优化算法在采用SSA形式之后优化效果得到了极大提高。甚至，有部分优化算法只能在SSA上做。



- 这里 LLVM 会去做些优化工作，在Xcode的编译设置里也可以设置优化级别-01，-03，-0s，还可以写些自己的 Pass，官方有比较完整的 Pass 教程： [Writing an LLVM Pass — LLVM 5 documentation](http://llvm.org/docs/WritingAnLLVMPass.html)  

优化IR：(级别-03)
```
clang -O3 -S -fobjc-arc -emit-llvm main.m -o main.ll
```
- Pass 是 LLVM 优化工作的一个节点，一个节点做些事，一起加起来就构成了 LLVM 完整的优化和转化。

如果开启了 bitcode 苹果会做进一步的优化，有新的后端架构还是可以用这份优化过的 bitcode 去生成。  

生成字节码：
```
clang -emit-llvm -c main.m -o main.bc
```
注：xx.bc文件是位流格式，由于是二进制的，所以直接看就是一堆乱码，查看bitcode最好的方式是用hexdump工具。

<br>


###  6⃣ Assemble -生成Target相关汇编
```
clang -S -fobjc-arc main.m -o main.s
```
###  7⃣ Assemble - 生成 Target 相关机器码 Object (Mach-O)
```
clang -fmodules -c main.m -o main.o
```
###  8⃣  Link 生成 Executable 可执行文件
```
clang main.o -o main
执行
./main
输出
starming rank 14
```
<br>

## 总结：Clang-LLVM 下，一个源文件的编译过程:

![](https://user-gold-cdn.xitu.io/2018/10/8/16652bcd7a26e0a5?w=904&h=489&f=jpeg&s=72140)


#### 整个工作流程犹如：

![](https://user-gold-cdn.xitu.io/2018/10/23/166a04a63cb2dbf4?w=1470&h=482&f=jpeg&s=139239)


## 完整的编译过程
1.优先编译cocopods里面的所有依赖文件
![](https://user-gold-cdn.xitu.io/2018/10/8/16652d974654bb57?w=884&h=817&f=jpeg&s=195377)
2.编译信息写入辅助信息，创建编译后的文件架构
![](https://user-gold-cdn.xitu.io/2018/10/8/16652d4c28a29575?w=1041&h=139&f=jpeg&s=66257)
3.处理打包信息。例如development环境下处理xxxx.entitlements的打包信息
![](https://user-gold-cdn.xitu.io/2018/10/8/16652e83854f69cb?w=1013&h=257&f=jpeg&s=85015)
4.执行cocopods编译前脚本 checkPods Manifest.lock
![](https://user-gold-cdn.xitu.io/2018/10/8/16652dc5d6876eae?w=1005&h=118&f=jpeg&s=67883)
5.编译包内所有m文件 （使用Compile和Clang的几个主要命令）
![](https://user-gold-cdn.xitu.io/2018/10/8/16652ddb40d2ea98?w=1026&h=124&f=jpeg&s=46049)
6.链接需要的framework，例如AFNetworking.framework，Masonry.framework等信息
![](https://user-gold-cdn.xitu.io/2018/10/8/16652e0442493795?w=1013&h=104&f=jpeg&s=70843)  
7.编译xib文件  
8.copy Xib文件，图片等资源文件放到结果目录  
![](https://user-gold-cdn.xitu.io/2018/10/8/16652eeb1656d29f?w=1018&h=45&f=jpeg&s=32721)
9.编译imageAsserts  
![](https://user-gold-cdn.xitu.io/2018/10/8/16652ee5d7e9f1c4?w=1013&h=26&f=jpeg&s=12456)
10.处理infoplist  
![](https://user-gold-cdn.xitu.io/2018/10/8/16652ede99e38ebd?w=1000&h=29&f=jpeg&s=20039)
11.执行Cocoapods脚本  
12.copy标准库  
13.创建.app文件和签名  
![](https://user-gold-cdn.xitu.io/2018/10/8/16652ebfdacf7e40?w=1013&h=205&f=jpeg&s=116721)
<br>

**我的深圳编译流程：**
![](https://user-gold-cdn.xitu.io/2018/10/8/16652ea8ff0b2227?w=1380&h=985&f=jpeg&s=546283)

<br>

## clang插件开发
### 准备工作
- 安装brew
> brew是一个软件包管理工具,类似于centos下的yum或者ubuntu下的apt-get,非常方便,免去了自己手动编译安装的不便  
brew 安装目录 /usr/local/Cellar  
brew 配置目录 /usr/local/etc  
brew 命令目录 /usr/local/bin  
注:homebrew在安装完成后自动在/usr/local/bin加个软连接，所以平常都是用这个路径
```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" 
```
- 安装cmake
> CMake是一个跨平台的编译(Build)工具,可以用简单的语句来描述所有平台的编译过程。
```
$ brew install cmake
```
- 安装ninja
> Ninja 是一个构建系统，与 Make 类似。作为输入，你需要描述将源文件处理为目标文件这一过程所需的命令。 Ninja 使用这些命令保持目标处于最新状态。
Ninja 的主要设计目标是速度。
```
$ brew install ninja
```

<br>

### 下载编译工具
- 下载LLVM
```
// 大小648.2M
$ git clone https://git.llvm.org/git/llvm.git/
```
- 下载clang
```
// 大小240.6M
$ cd llvm/tools
$ git clone https://git.llvm.org/git/clang.git/
```
-  在LLVM源码同级目录下新建一个【llvm_build】目录(最终会在【llvm_build】目录下生成【build.ninja】)
```
$ cd llvm_build
$ cmake -G Ninja ../llvm -DCMAKE_INSTALL_PREFIX=LLVM的安装路径
```

- 依次执行编译、安装指令
```
$ ninja
```
编译完毕后， 【llvm_build】目录大概 21.05 G(仅供参考)
```
$ ninja install
```
安装完毕后，安装目录大概 11.92 G(仅供参考)

- 在llvm同级目录下新建一个【llvm_xcode】目录
```
$ cd llvm_xcode
$ cmake -G Xcode ../llvm
```

<br>


## 开始开发插件
- 在【llvm/tools/clang/tools】源码目录下新建一个插件目录，假设叫做【yb-plugin】
![](https://user-gold-cdn.xitu.io/2018/9/30/166290b28aa24291?w=1023&h=478&f=jpeg&s=159389)

- 在【llvm/tools/clang/tools/CMakeLists.txt】最后加入内容: add_clang_subdirectory(yb-plugin)，小括号里是插件目录名
```
# libclang may require clang-tidy in clang-tools-extra.
add_clang_subdirectory(libclang)
add_clang_subdirectory(yb-plugin)
```
- 在【yb-plugin】目录下新建一个【YBPlugin.cpp】和【CMakeLists.txt】，并在【CMakeLists.txt】文件里面添加如下内容：
```
add_llvm_loadable_module(YBPlugin YBPlugin.cpp)
```
> MJPlugin是插件名，MJPlugin.cpp是源代码文件

<br>


## 编译插件
>### 生成xcode项目
- 利用cmake生成的Xcode项目来编译插件(第一次编写完插件，需要利用cmake重新生成一下Xcode项目)
>### 编写插件
- 插件源代码在【Sources/Loadable modules】目录下可以找到，这样就可以直接在Xcode里编写插件代码
>### 编译插件生成动态库文件
- 选择MJPlugin这个target进行编译，编译完会生成一个动态库文件，将动态库文件存放在桌面。

<br>


## 加载插件

- 在Xcode项目中指定加载插件动态库:BuildSettings > OTHER_CFLAGS
```
-Xclang -load -Xclang 动态库路径 -Xclang -add-plugin -Xclang 插件名称
```

![](https://user-gold-cdn.xitu.io/2018/9/30/166291e5febbc175?w=2048&h=986&f=jpeg&s=247798)

<br>


## 使用插件
- 首先要对Xcode进行Hack，才能修改默认的编译器
> 下载【XcodeHacking.zip】，解压，修改【HackedClang.xcplugin/Contents/Resources/HackedClang.xcspec】的内容，设
置一下自己编译好的clang的路径
> ![](https://user-gold-cdn.xitu.io/2018/9/30/16629249474c657f?w=2048&h=795&f=jpeg&s=214089)
- 然后在XcodeHacking目录下进行命令行，将XcodeHacking的内容剪切到Xcode内部
```
$ sudo mv HackedClang.xcplugin `xcode-select-print- path`/../PlugIns/Xcode3Core.ideplugin/Contents/SharedSupport/Developer/Library/Xcode/Plug-ins
$ sudo mv HackedBuildSystem.xcspec `xcode-select-print- path`/Platforms/iPhoneSimulator.platform/Developer/Library/Xcode/Specifications
```

- 配置项目
![](https://user-gold-cdn.xitu.io/2018/9/30/16629293f12beaa9?w=1966&h=734&f=jpeg&s=204046)

<br>

## 插件效果
![](https://user-gold-cdn.xitu.io/2018/9/30/166291f2bacc5e05?w=2022&h=1262&f=jpeg&s=425240)


<br>

<br>

# 应用场景
 [clang插件编写]()

 [代码混淆](https://github.com/obfuscator-llvm/obfuscator/wiki)

 [APP包瘦身](https://mp.weixin.qq.com/s?__biz=MzUxMzcxMzE5Ng==&mid=2247488360&amp;idx=1&amp;sn=94fba30a87d0f9bc0b9ff94d3fed3386&source=41#wechat_redirect)

 [iOS动态化方案——OCS](https://blog.csdn.net/guojin08/article/details/54310858)

 [开发新的编程语言](https://llvm-tutorial-cn.readthedocs.io/en/latest/index.html) 


<br>

<br>

# 推荐书记

![](https://user-gold-cdn.xitu.io/2018/9/30/166294c4bc43de48?w=1154&h=559&f=jpeg&s=154084)
