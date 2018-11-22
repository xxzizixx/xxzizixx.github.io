---
title: Copy修饰和深浅复制以及NSMutableObject的修饰词
date: 2015-07-07 17:44:04
tags: 
    - 求学之路
---

## 先说结论

1：对于不可变对象，copy都是浅复制，即指针复制。mutableCopy 都是alloc一个新对象返回。
2：对于可变对象，copy和mutableCopy都是alloc新对象返回。
3：不论是可变还是不可变对象，copy返回的对象都是不可变的，mutableCopy返回的对象都是可变的。
4：容器类对象，不论是可变的还是不可变的，copy，mutableCopy返回的对象里所包含的对象的地址和之前都是一样 的，即容器内对象都是浅拷贝。

## 举个栗子

#### 1、不可变字符串的拷贝
```
NSString *string = @"string";

NSString *str1 = [string copy];                // 没有产生新对象 【浅拷贝】
NSMutableString *str2 = [string mutableCopy];  // 产生新对象     【深拷贝】

NSLog(@"%p %p %p", string, [string copy], [string mutableCopy]);
```
```
打印结果如下：
0x100001a50 0x100001a50 0x100201a00
```
根据打印结果解析：
1⃣️当不可变字符串调用copy的时候，指针地址没有发生改变，也就意味着没有产生新的对象，所以属于浅拷贝；
2⃣️当不可变字符串调用mutableCopy的时候，指针地址发生了改变，意味着产生新的对象，所以属于深拷贝。



#### 2、可变字符串的拷贝
```
NSMutableString *string = [NSMutableString string];
[string appendString:@"text1"];

NSString *str1 = [string copy];              // 产生新对象,变为不可变字符串【深拷贝】
NSMutableString *str2 = [string mutableCopy];// 产生新对象,还是可变字符串  【深拷贝】

[str2 appendString:@"text2"];

NSLog(@"%p %p %p", string, [string copy], [string mutableCopy]);
NSLog(@"%@ %@ %@", string, str1, str2);
```
```
打印结果如下：
0x100300280 0x317478657233 0x1003004f0
text1 text1 text1text2
```
根据打印结果解析：
1⃣️当可变字符串调用copy的时候，指针地址发生了改变，也就意味着产生新的对象，所以属于深拷贝；
2⃣️当可变字符串调用mutableCopy的时候，指针地址发生了改变，意味着产生新的对象，所以属于深拷贝。




#### 3、自定义对象的拷贝

###### 由于自定义对象不考虑可变，所以忽略mutableCopy

首先，当对象需要调用copy的时候，需要遵守遵守NSCopying协议和调用copyWithZone：这个方法

```
@interface Dog : NSObject
/** 姓名 */
@property (nonatomic, copy) NSString *name;
/** 年龄 */
@property (nonatomic, assign) int age;

@end


// 需要遵守 NSCopying 协议
@interface Dog () <NSCopying>

@end

@implementation Dog
// 当对象需要调用 copy 的时候，需要调用 copyWithZone：这个方法
- (id)copyWithZone:(NSZone *)zone
{
Dog *dog = [[Dog allocWithZone:zone] init];
dog.name = self.name;
dog.age  = self.age;
return dog;
}
@end
```
```
Dog *dog = [[Dog alloc] init]; // [object copyWithZone:zone]
dog.name = @"huahua";
dog.age  = 1;

Dog *newDog = [dog copy]; // 产生新对象【深拷贝】
NSLog(@"%@ %@",dog,newDog);
NSLog(@"%@ %@",dog.name,newDog.name);
```
```
打印结果如下：
<Dog: 0x100600350> <Dog: 0x100600520>
huahua huahua
```
根据打印结果解析：
当自定义对象调用copy的时候，指针地址发生了改变，也就意味着产生新的对象，所以属于深拷贝；



#### 4、属性中的copy 和 strong

在平时定义属性的时候，对于NSString和block我们经常用copy来修饰
数组和字典等类型用strong来修饰；
当使用copy修饰属性的时候，属性的setter方法会调用[object copy]产生新的对象，
这样，当原object对象的值发生改变时，并不影响新对象值；
```
// 定义NSString
@property(nonatomic, copy) NSString *name;

// 当调用上面的copy的时候，等价于下面的代码
- (void)setName:(NSString *)name {
if (_name != name) {
[_name release];
_name = [name copy];
}
}
```

当使用strong修饰属性的时候，属性的setter方法会直接强引用该对象，这样，当原object对象的值发生改变时，新对象的属性也改变；
例如：我们平时使用strong修饰的NSMutableArray，这个可变数组在当前文件中只有一个，而且是可变的;
```
/** 数组 */
@property(nonatomic,strong)NSMutableArray *array;

// 当调用上面的strong的时候，等价于下面的代码
-(void)setArray:(NSMutableArray *)array{
_array = array;
}
```

##### 以前面的自定义Dog对象进行举例：
```
// 定义一个可变的字符，作为小狗的name
NSMutableString *dogName = [NSMutableString stringWithString:@"huahua"];   // dogName == "huahua"

Dog *dog = [[Dog alloc] init];
// 将字符赋值给dog的name属性
dog.name = dogName;
// 当小狗的name值发生改变时
[dogName appendString:@"lvlv"];   // dogName == "huahualvlv"
// 小狗的名还是原来的姓名
NSLog(@"%@",dog.name);            // 打印结果：huahua
```

分析：
当给dog.name赋值时，会将[dogName copy]后的结果赋值给dog.name，这样当dogName字符的值发生改变后，不会影响dog.name的值；

### 总结：

当不可变类型对象调用copy拷贝后，不会产生新的对象，属于浅拷贝，其他类型对象不管调用copy亦或是mutableCopy，都会产生新的对象，属于深拷贝！
![](https://ws2.sinaimg.cn/large/006tNc79gy1fpp1m1k7lwj30mm0b277t.jpg)
