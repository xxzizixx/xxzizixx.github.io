---
title: nil、Nil、NULL、NSNull区别以及给空对象发送消息
date: 2015-08-13 14:37:03
tags:  
    - 求学之路
---

首先，OC中向nil发消息，程序是不会崩溃的。因为OC的函数调用都是通过objc_msgSend进行消息发送来实现的，相对于C和C++来说，对于空指针的操作会引起Crash的问题，而objc_msgSend会通过判断self来决定是否发送消息，如果self为nil，那么selector也会为空，直接返回，所以不会出现问题。视方法返回值，向nil发消息可能会返回nil(返回值为对象)、0（返回值为一些基础数据类型）或0X0（返回值为id）等。
但是对[NSNull null]对象发送消息时，是会crash的，因为这个NSNull类只有一个null方法。
当然，如果一个对象已经被释放了（引用计数为0了），那么这个时候再去调用方法肯定是会Crash的，因为这个时候这个对象就是一个野指针（指向僵尸对象（对象的引用计数为0，指针指向的内存已经不可用）的指针）了，安全的做法是释放后将对象重新置为nil，使它成为一个空指针，大家可以在关闭ARC后手动release对象验证一下。

## nil

nil的定义是null pointer to object-c object，指的是一个OC对象指针为空，本质就是(id)0，是OC对象的字面0值。

不过这里有必要提一点就是OC中给空指针发消息不会崩溃的语言特性，原因是OC的函数调用都是通过objc_msgSend进行消息发送来实现的，相对于C和C++来说，对于空指针的操作会引起Crash的问题，而objc_msgSend会通过判断self来决定是否发送消息，如果self为nil，那么selector也会为空，直接返回，所以不会出现问题。

这里补充一点，如果一个对象已经被释放了，那么这个时候再去调用方法肯定是会Crash的，因为这个时候这个对象就是一个野指针了，安全的做法是释放后将对象重新置为nil，使它成为一个空指针，大家可以在关闭ARC后手动release对象验证一下。
```
NSString *name = @"Allen";

if (name != nil && [name isEqualToString:@"Allen"]) {
NSLog(@"name: %@", name);
} else {
NSLog(@"name is nil");
}

//or
if ([name isEqualToString:@"Allen"]) {
NSLog(@"name: %@", name);
} else {
NSLog(@"name is nil");
}
```
上面的两种判断都是正确的，我们不必担心当name为nil时调用isEqualToString会出现Crash，但是我还是想说，在使用一个对象之前判断它是否为nil是一个很好的习惯，个人觉得有两个原因:

降低时间复杂度（感觉可以这么说吧），如果你增加了nil的判断，那么不需要对空指针发送消息了，发消息其实是件费时的操作。详情可以看这里
把判断为空养成习惯其实是好事，这样在你切换语言时也不容易出错。



## Nil

Nil的定义是null pointer to object-c class，指的是一个类指针为空。本质就是(class)0，OC类的字面零值。
```
Class class = [NSString class];
if (class != Nil) {
NSLog(@"class name: %@", class);
}
```


## NULL
NULL的定义是null pointer to primitive type or absence of data，指的是一般的基础数据类型为空，可以给任意的指针赋值。本质就是(void *)0，是C指针的字面0值。
```
NSInteger *pointerA = NULL; 
NSInteger pointerB = 10; 
pointerA = &pointerB; 
NSLog(@”%ld”, *pointerA);
```
我们要尽量不去将NULL初始化OC对象，可能会产生一些异常的错误，要使用nil，NULL主要针对基础数据类型。



## NSNull
NSNull好像没有什么具体的定义（懵），它包含了唯一一个方法+(NSNull*)null，[NSNull null]是一个对象，用来表示零值的单独的对象。
```
NSMutableDictionary *dictionary = [[NSMutableDictionary alloc] init]; 
NSString *nameOne = @”Allen”; 
NSString *nameTwo = [NSNull null]; // 不用使用 nil，nil在字典，数组中有特殊含义–元素结束标记
```
```
NSString *nameThree = @"Tom";
[dictionary setObject:nameOne forKey:@"nameOne"];
[dictionary setObject:nameTwo forKey:@"nameTwo"];
[dictionary setObject:nameThree forKey:@"nameThree"];
NSLog(@"names: %@", dictionary);

NSMutableArray *array = [[NSMutableArray alloc] init];
[array addObject:nameOne];
[array addObject:nameTwo];
[array addObject:nameThree];
NSLog(@"names : %@", array);
```
NSNull主要用在不能使用nil的场景下，比如NSMutableArray是以nil作为数组结尾判断的，所以如果想插入一个空的对象就不能使用nil, NSMutableDictionary也是类似，我们不能使用nil作为一个object，而要使用NSNull。



## 向nil发送消息不Crash的原因：
向nil发送消息:
在Objective-C中向nil发送消息是完全有效的——只是在运行时不会有任何作用。Cocoa中的几种模式就利用到了这一点。发向nil的消息的返回值也可以是有效的:
• 如果一个方法返回值是一个对象，那么发送给nil的消息将返回0(nil)。例如：Person _ motherInlaw = [ aPerson spouse] mother]; 如果spouse对象为nil，那么发送给nil的消息mother也将返回nil。
• 如果方法返回值为指针类型，其指针大小为小于或者等于sizeof(void_)，float，double，long double 或者long long的整型标量，发送给nil的消息将返回0。
• 如果方法返回值为结构体，正如在《Mac OS X ABI 函数调用指南》，发送给nil的消息将返回0。结构体中各个字段的值将都是0。其他的结构体数据类型将不是用0填充的。
• 如果方法的返回值不是上述提到的几种情况，那么发送给nil的消息的返回值将是未定义的。    




## 理解一下nil，NULL和[NSNull null]的区别：
nil用来给对象赋值（Objective-C中的任何对象都属于id类型），NULL则给任何指针赋值，NULL和nil不能互换，nil用于类指针赋值（在Objective-C中类也是一个对象，是类的meta-class的实例，有关meta-class参见资料[【译】Objective-C 中的 Meta-class 是什么？）](http://www.cocoachina.com/industry/20131210/7508.html)， 而NSNull则用于集合操作，用在数组或字典中要添加某个内容为空的情况。
虽然它们表示的都是空值，但使用的场合完全不同。所以在编码时严格按照变量类型来赋值，将正确的空值赋给正确的类型，使代码易于阅读和维护，也不易引起错误。
示例如下：
```
id object = nil;  
// 判断对象不为空  
if (object) {  
}       
// 判断对象为空  
if (object == nil) {  
}  

// 数组初始化，空值结束  
NSArray *array = [[NSArray alloc] initWithObjects:@"First", @"Second", nil];  
// 判断数组元素是否为空  
NSString *element = [array objectAtIndex:2];  
if ((NSNull *)element == [NSNull null]) {  
}  

//做项目时遇到，要判断数组元素是否为空，以下写法，都无效
if(!element)
if([element length]>0)
if(element == NULL)
if(element == nil)

// 判断字典对象的元素是否为空  
NSDictionary *dictionary = [NSDictionary dictionaryWithObjectsAndKeys:  
@"iPhone", @"First", @"iPad", @"Second", nil];  
NSString *value = [dictionary objectForKey:@"First"];  
if ((NSNull *)value == [NSNull null]) {  
}
```


#### 总结一下：
1.  nil：一般赋值给空对象；
2.  NULL：一般赋值给nil之外的其他空值。如SEL等，如：
```
[NSApp beginSheet:sheet modalForWindow:mainWindow
modalDelegate:nil     // 使用nil指向一个空对象
didEndSelector:NULL    // 指向空
　　contextInfo:NULL];
```
3.  当向nil发送消息时，不会有异常，程序将继续执行下去；
4.  而向NSNull的对象发送消息时会收到异常，因为NSNull只有一个方法：+ (NSNull *) null;，会“没有这样的selector”的错误。 [NSNull null]是一个对象，他用在不能使用nil的场合。因为在NSArray和NSDictionary中nil有特殊的含义。但是在有些时候，确实需要用到这样的空值，比如在字典中，电话簿中”Jack”关键字下有电话号码、家庭住址、Email等等信息，但是现在只知道他的电话号码，这种不知道其他信息的情况下为了消除一些歧义，有必要将它们设置为空，所以Cocoa提供了NSNull类。
```
[dictionary setObject:[NSNull null], forKey:"Email"];
if(EmailAdress == [NSNull null]) {
　　//to do something...
　　}
```
因为Object-C的集合对象，如NSArray、NSDictionary、NSSet等，都有可能包含NSNull对象，所以，如果以下代码中的item为NSNull，则会引起程序崩溃：
```
NSString *item=[NSArray objectAtIndex:i];
if([item isEqualToString:@"TestNumber"]) {
// do someThing
}
// 以下代码是常见的错误，release对象没有设置为nil，从而引起程序崩溃。
id someObject=[[Object alloc] init];
//...
[someObject release];
//...
if(someObject) {
//crash here
}
```

上面会发生常见的“EXC_BAD_ACCESS”错误，也就是野指针错误。因为someObject指针指向的那块内存的引用计数已经为0了，所以那块内存已经不可以访问了，但是someObject指针并没有设为nil，所以会报野指针错误，那块内存地址中的僵尸对象已经无法使用。
