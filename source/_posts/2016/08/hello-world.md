---
title: MJExtesion源码解析
date: 2016-08-15 17:55:32
tags: 
---

<br>
<br>

# 前沿
> A fast, convenient and nonintrusive conversion between JSON and model. Your model class don't need to extend another base class. You don't need to modify any model file.

一个快速、方便、无侵入性的Json转模型框架，不需要扩展基类，也不需要修改任何的模型文件。


<br>
<br>

# 框架结构
![](https://ws1.sinaimg.cn/large/006tNbRwgy1fxzbj1lc8vj30n90jjq74.jpg)

### **MJExtension**(头文件)
```
#import "NSObject+MJCoding.h"
#import "NSObject+MJProperty.h"
#import "NSObject+MJClass.h"
#import "NSObject+MJKeyValue.h"
#import "NSString+MJExtension.h"
#import "MJExtensionConst.h"
```

```
// 辅助类
#import "MJFoundation.h"
#import "MJProperty.h"
#import "MJPropertyKey.h"
#import "MJPropertyType.h"
```

这些类的命名很规范，根据类的名字大概就能猜到是用来做什么事情的。
<br>

### **MJExtensionConst**
这里定义了整个库的常量和宏定义。类似于一个项目中的预编译文件。包含了:
* 方法弃用宏
* 自定义Log日志
* 自定义断言
* 快速打印所有属性
* 自定义类型编码字符串常量

前面几个都很简单，没什么好说的，看一下最后一个“自定义类型编码字符串常量”
```
NSString *const MJPropertyTypeInt = @"i";
NSString *const MJPropertyTypeShort = @"s";
NSString *const MJPropertyTypeFloat = @"f";
NSString *const MJPropertyTypeDouble = @"d";
NSString *const MJPropertyTypeLong = @"l";
NSString *const MJPropertyTypeLongLong = @"q";
NSString *const MJPropertyTypeChar = @"c";
NSString *const MJPropertyTypeBOOL1 = @"c";
NSString *const MJPropertyTypeBOOL2 = @"b";
NSString *const MJPropertyTypePointer = @"*";

NSString *const MJPropertyTypeIvar = @"^{objc_ivar=}";
NSString *const MJPropertyTypeMethod = @"^{objc_method=}";
NSString *const MJPropertyTypeBlock = @"@?";
NSString *const MJPropertyTypeClass = @"#";
NSString *const MJPropertyTypeSEL = @":";
NSString *const MJPropertyTypeId = @"@";
```
这里的类型编码就是按照下图苹果文档中的类型编码进行定义的，定义成const常量方便编码和阅读。
![](https://ws2.sinaimg.cn/large/006tNbRwgy1fxzf0c6djbj30ez0l5di4.jpg)

<br>

### **MJFoundation.h**
这个类做的事情就是利用**穷举法**判断类是不是属于`Foundation`框架里面的。进行判断的类放在classMap里面，源码如下：
```
foundationClasses = [NSSet setWithObjects:
                            [NSURL class],
                            [NSDate class],
                            [NSValue class],
                            [NSData class],
                            [NSError class],
                            [NSArray class],
                            [NSDictionary class],
                            [NSString class],
                            [NSAttributedString class], nil];
```
然后遍历循环判断是否是属于Foundation框架，源码如下：
```
__block BOOL result = NO;
[foundationClasses enumerateObjectsUsingBlock:^(Class foundationClass, BOOL *stop) {
    if ([c isSubclassOfClass:foundationClass]) {
        result = YES;
        *stop = YES;
    }
}];
```
* 这里有个细节需要注意:result是BOOL类型，属于基础数据类型，在Block内部修改，需要先进行包装，既利用__block进行修饰，把result包装成对象类型，然后在block内部进行修改。

#### 总结：其实就是一个工具分类,操作变量或者属性名
<br>



### **MJPropertyType**
MJPropertyType对属性的具体类型进行封装，简单来说就是讲类型编码转为我们熟知的类型。暴露给外面使用，其属性如下：
```
/**
*  包装一种类型
*/
@interface MJPropertyType : NSObject
/** 类型标识符 */
@property (nonatomic, copy) NSString *code;

/** 是否为id类型 */
@property (nonatomic, readonly, getter=isIdType) BOOL idType;

/** 是否为基本数字类型：int、float等 */
@property (nonatomic, readonly, getter=isNumberType) BOOL numberType;

/** 是否为BOOL类型 */
@property (nonatomic, readonly, getter=isBoolType) BOOL boolType;

/** 对象类型（如果是基本数据类型，此值为nil） */
@property (nonatomic, readonly) Class typeClass;

/** 类型是否来自于Foundation框架，比如NSString、NSArray */
@property (nonatomic, readonly, getter = isFromFoundation) BOOL fromFoundation;
/** 类型是否不支持KVC */
@property (nonatomic, readonly, getter = isKVCDisabled) BOOL KVCDisabled;

/**
*  获得缓存的类型对象
*/
+ (instancetype)cachedTypeWithCode:(NSString *)code;
```
通过类方法初始化`+ (instancetype)cachedTypeWithCode:(NSString *)code`.
**因为总共的类型就如上提到的那几种，所以这里用了一个静态的字典保存结果。下次直接从结果中取，如果有则不用依次判断了**

这里需要注意的是：参数类型转换
```
if ([code isEqualToString:MJPropertyTypeId]) {
    _idType = YES;
} else if (code.length == 0) {
    _KVCDisabled = YES;
} else if (code.length > 3 && [code hasPrefix:@"@\""]) {
    // 去掉@"和"，截取中间的类型名称
    _code = [code substringWithRange:NSMakeRange(2, code.length - 3)];
    _typeClass = NSClassFromString(_code);
    _fromFoundation = [MJFoundation isClassFromFoundation:_typeClass];
    _numberType = [_typeClass isSubclassOfClass:[NSNumber class]];

} else if ([code isEqualToString:MJPropertyTypeSEL] ||
    [code isEqualToString:MJPropertyTypeIvar] ||
    [code isEqualToString:MJPropertyTypeMethod]) {
    _KVCDisabled = YES;
}
```
传进来的`code`就是属性所对应的属性名称，比如`NSArray`对应`@"NSArray"`。自定义的类，签名都会有一个`@`符号，后面接上类名。形式如`@Classname`。
<br>




### **MJPropertyKey.h**

这个类是对属性进行分类。所有属性可以归为两类一种是字典，也就是键值对（`NSDictionary`）和数组（`NSArray`）。

外面通过调用`- (id)valueInObject:(id)object`从传进来的`id`类型中取值。
<br>



### **MJProperty.h**

对`objc_property_t`封装，存储一个属性值相关的详细信息，将`objc_property_t`转为能够直接用的对象。它是对`objc_property_t`类型的一次封装，便于我们使用。
同时它也依赖于上面所介绍的几种数据类型。从`.h`文件看到`#import "MJPropertyType.h" #import "MJPropertyKey.h"`。同样也依赖于`#import "MJFoundation.h" #import "MJExtensionConst.h"`

初始化入口函数`+ (instancetype)cachedPropertyWithProperty:(objc_property_t)property`。正如上面所说目的就是将`runtime`中的`objc_property_t `转为`MJProperty`方便我们的使用。

具体实现：
```
#pragma mark - 缓存
+ (instancetype)cachedPropertyWithProperty:(objc_property_t)property
{
    MJExtensionSemaphoreCreate
    MJExtensionSemaphoreWait
    MJProperty *propertyObj = objc_getAssociatedObject(self, property);
    if (propertyObj == nil) {
        propertyObj = [[self alloc] init];
        propertyObj.property = property;
        objc_setAssociatedObject(self, property, propertyObj, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }
    MJExtensionSemaphoreSignal
    return propertyObj;
}
```
` MJProperty *propertyObj = objc_getAssociatedObject(self, property);`这句话是动态给类添加属性，也就是说，给类添加了一个属性名为`property `的`MJProperty`类型属性。**注意这里是一个类方法，也就是这里的`self`代表什么。**

这里的缓存方法是通过懒加载的形式实现的，将需要缓存的属性动态添加到类上面。如果没有则新加属性有则直接返回属性。这样做的目的是为了一个类可能在项目中会用到很多次字典转模型。所以保存一份之后就不用每次都创建新的属性标识了。**以空间换时间**把这个方法执行完，就得到了如下三个属性的值：
```
/** 成员属性 /
@property (nonatomic, assign) objc_property_t property;
/* 成员属性的名字 */
@property (nonatomic, readonly) NSString *name;
/** 成员属性的类型 */
@property (nonatomic, readonly) MJPropertyType *type;
```
除了公开的属性还有两个私有属性：
```
@property (strong, nonatomic) NSMutableDictionary *propertyKeysDict;
@property (strong, nonatomic) NSMutableDictionary *objectClassInArrayDict;
```
他们分别保存了的类型是`MJPropertyKey`
通过
```
/**
* 设置object的成员变量值
*/
- (void)setValue:(id)value forObject:(id)object;
/**
* 得到object的成员属性值
*/
- (id)valueForObject:(id)object;
```
对特定的属性存取值
```
/**** 同一个成员属性 - 父类和子类的行为可能不一致（originKey、propertyKeys、objectClassInArray） ****/
/** 设置最原始的key */
- (void)setOriginKey:(id)originKey forClass:(Class)c;
/** 对应着字典中的多级key（里面存放的数组，数组里面都是MJPropertyKey对象） */
- (NSArray *)propertyKeysForClass:(Class)c;

/** 模型数组中的模型类型 */
- (void)setObjectClassInArray:(Class)objectClass forClass:(Class)c;
- (Class)objectClassInArrayForClass:(Class)c;
/**** 同一个成员变量 - 父类和子类的行为可能不一致（key、keys、objectClassInArray） ****/
```
<br>


### **NSObject+MJClass.h**
这个类的功能大致可以归为`遍历`、`属性白名单`、`属性黑名单`。所以可以重点来看看这三个部分。

外部通过`+ (NSMutableArray *)mj_totalIgnoredPropertyNames;`和`+ (NSMutableArray *)mj_totalAllowedCodingPropertyNames;`获得黑名单与白名单。
遍历就两个方法：
```
/**
*  遍历所有的类
*/
+ (void)mj_enumerateClasses:(MJClassesEnumeration)enumeration;
+ (void)mj_enumerateAllClasses:(MJClassesEnumeration)enumeration;
```
`mj_enumerateClasses `只要当前遍历的是`Foundatoin`框架的就会退出遍历，否则会一直沿着继承树遍历。`mj_enumerateAllClasses `会遍历继承树上所有的类。**为什么会存在遍历到`Foundatoin`框架就停止遍历了，因为我们自定义的模型大部分是继承至NSObject这中类型。这是为什么停止，那为什么要遍历。因为自定义的模型可能继承自己我们自定义的模型。为了保护所有的信息，比如属性信息，所以需要遍历。**

注意这里的参数是其实是一个`typedef void (^MJClassesEnumeration)(Class c, BOOL *stop);`Block。写法有点类似系统中数组遍历。这种写法值得学习，平时我们遍历都是在类中直接调用一个方法，而通过这样传递`Block`这样就更加解耦了。其实也可以通过`Target-Action`模式实现。注意这里的`Bool`类型传的是指针哦，就像`*stop`。

相关的实现：
```
+ (void)mj_enumerateClasses:(MJClassesEnumeration)enumeration
{
    // 1.没有block就直接返回
    if (enumeration == nil) return;

    // 2.停止遍历的标记
    BOOL stop = NO;

    // 3.当前正在遍历的类
    Class c = self;

    // 4.开始遍历每一个类
    while (c && !stop) {
        // 4.1.执行操作
        enumeration(c, &stop);

        // 4.2.获得父类
        c = class_getSuperclass(c);

        if ([MJFoundation isClassFromFoundation:c]) break;
    }
}
```
配置白名单是为了对属性进行过滤。只有在白名单中的属性名才会进行字典和模型的转换。来说一下这里涉及的配置方法。

外部调用者，通过类方法传入`Block`参数进行配置。`typedef NSArray * (^MJAllowedPropertyNames)();`这是传入的`Block`定义。可以看到返回的是一个数组。为什么不直接将白名单属性暴露处理出来给调用者直接使用呢？大概是遵循了设计模式中的`知道最少原则`。

在`.m`文件中定义了几个保存白名单、黑名单的静态数组。定义如下：
```
static NSMutableDictionary *allowedPropertyNamesDict;
static NSMutableDictionary *ignoredPropertyNamesDict;
static NSMutableDictionary *allowedCodingPropertyNamesDict;
static NSMutableDictionary *ignoredCodingPropertyNamesDict;
```
为了保存传入的Block信息，需要给分类动态添加属性
```
static const char MJAllowedPropertyNamesKey = '\0';
static const char MJIgnoredPropertyNamesKey = '\0';
static const char MJAllowedCodingPropertyNamesKey = '\0';
static const char MJIgnoredCodingPropertyNamesKey = '\0';
```
比如这里的`MJAllowedPropertyNamesKey `就是白名单传进来的属性名称了。

设置白名单的入口：最终调用的是`mj_setupBlockReturnValue`
```
#pragma mark - block和方法处理:存储block的返回值
+ (void)mj_setupBlockReturnValue:(id (^)(void))block key:(const char *)key
{
    if (block) {
        objc_setAssociatedObject(self, key, block(), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    } else {
        objc_setAssociatedObject(self, key, nil, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
    // 清空数据
    [[self classDictForKey:key] removeAllObjects];
}
```
可以很清楚的看到这里将block作为自身的属性。

**除了这种方式还有一种：**那就是在类中实现`mj_allowedPropertyNames`比如：
```
(NSArray *)mj_allowedPropertyNames {
return @[@"name",@"icon"];
}
```
后面获取可用属性的时候会对这两种方式都判断。

#### **设置黑名单**
设置黑名单的方式和设置白名单类似。

#### **最终可转换的数组**
调用这个方法`mj_totalIgnoredPropertyNames `就是返回经过过滤后的属性。一共有两种方式，一种是通过`Selecotr`一种是通过`Block`设置。
```
+ (NSMutableArray *)mj_totalObjectsWithSelector:(SEL)selector key:(const char *)key
{
    MJExtensionSemaphoreCreate
    MJExtensionSemaphoreWait

    NSMutableArray *array = [self classDictForKey:key][NSStringFromClass(self)];
    if (array == nil) {
        // 创建、存储
        [self classDictForKey:key][NSStringFromClass(self)] = array = [NSMutableArray array];

        if ([self respondsToSelector:selector]) {
    #pragma clang diagnostic push
    #pragma clang diagnostic ignored "-Warc-performSelector-leaks"
            NSArray *subArray = [self performSelector:selector];
    #pragma clang diagnostic pop
            if (subArray) {
                [array addObjectsFromArray:subArray];
            }
        }

        [self mj_enumerateAllClasses:^(__unsafe_unretained Class c, BOOL *stop) {
        NSArray *subArray = objc_getAssociatedObject(c, key);
        [array addObjectsFromArray:subArray];
        }];
    }

    MJExtensionSemaphoreSignal

    return array;
}
```

#### **通过Selector**
`selector`是使用者如果想添加白名单需要自定义实现类方法`mj_allowedPropertyNames`。返回的是一个白名单数组。黑名单原理类似。`key`是指定过滤的是白名单还是黑名单。

`selector`的方式需要给对应的类添加一个类方法如：
```
(NSArray *)mj_allowedPropertyNames {
    return @[@"name",@"icon"];
}
```


#### **通过Block**
设置block属性
```
if (block) {
    objc_setAssociatedObject(self, key, block(), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
} else {
    objc_setAssociatedObject(self, key, nil, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
```


获取利用属性关联的block属性：
```
[self mj_enumerateAllClasses:^(__unsafe_unretained Class c, BOOL *stop) {
    NSArray *subArray = objc_getAssociatedObject(c, key);
    [array addObjectsFromArray:subArray];
}];
```


#### **消除编译器警告**
```
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
NSArray *subArray = [self performSelector:selector];
#pragma clang diagnostic pop
```
这里的'-Warc-performSelector-leaks'是消除编译器报内存泄漏的警告
当我们确定编译器的警告对我们来说没用处的时候，为了避免出现警告，可以使用如下代码来忽略此警告：
```
#pragma clang diagnostic push
#pragma clang diagnostic ignored "xx要忽略的警告类型xx"
    xx要忽略警告的代码段xx
#pragma clang diagnostic pop
```

<br>



### **NSObject+MJCoding.h**

这个类是用于归档。只有实现了`MJCoding`协议的类才能够归档。
```
@optional
/**
*  这个数组中的属性名才会进行归档
*/
+ (NSArray *)mj_allowedCodingPropertyNames;
/**
*  这个数组中的属性名将会被忽略：不进行归档
*/
+ (NSArray *)mj_ignoredCodingPropertyNames;
```
在进行归档的时候，我们只需要在`Implemenion`中添加`MJExtensionCodingImplementation`。实际上是宏定义了`NSCode`进行归档，解档。
```
/**
归档的实现
*/
#define MJCodingImplementation \
- (id)initWithCoder:(NSCoder *)decoder \
{ \
if (self = [super init]) { \
[self mj_decode:decoder]; \
} \
return self; \
} \
\
- (void)encodeWithCoder:(NSCoder *)encoder \
{ \
[self mj_encode:encoder]; \
}

#define MJExtensionCodingImplementation MJCodingImplementation
```
至于归档解档的部分可以直接看.m文件中的实现。

<br>



### **NSObject+MJProperty.h**

这个类保存的属性的一些配置。大致可分为：

- 1.遍历属性
- 2.新值配置
- 3.key配置
- 4.array model class配置

#### **遍历属性**
```
/**
*  遍历所有的成员
*/
+ (void)mj_enumerateProperties:(MJPropertiesEnumeration)enumeration;
```
遍历所有属性的入口。这个方法在很多地方都存在过。属性会从缓存中取，`NSArray *cachedProperties = [self properties];`。`[self properties]`是缓存属性的部分。然后就直接遍历所有的属性。
```
+ (void)mj_enumerateProperties:(MJPropertiesEnumeration)enumeration
{
    // 获得成员变量
    NSArray *cachedProperties = [self properties];

    // 遍历成员变量
    BOOL stop = NO;
    for (MJProperty *property in cachedProperties) {
        enumeration(property, &stop);
        if (stop) break;
    }
}
```
这里遍历用了`block`,外部传递`block`进来遍历。`typedef void (^MJPropertiesEnumeration)(MJProperty *property, BOOL *stop);`这个库很多地方都用到了类似的遍历方式。

#### **新值配置**

新值配置是什么意思？：就是改变特定的属性的原有值，这样更加灵活。

同样有两种方式`Block`和`类方法`。

存取方法如下：
```
/**
*  用于过滤字典中的值
*
*  @param newValueFormOldValue 用于过滤字典中的值
*/
+ (void)mj_setupNewValueFromOldValue:(MJNewValueFromOldValue)newValueFormOldValue;
+ (id)mj_getNewValueFromObject:(__unsafe_unretained id)object oldValue:(__unsafe_unretained id)oldValue property:(__unsafe_unretained MJProperty *)property;
```
这里的`typedef id (^MJNewValueFromOldValue)(id object, id oldValue, MJProperty *property);`其实和下面方法参数形式是一样的。

看了一下这个方法，一次只支持一个属性新值配置。
```
+ (void)mj_setupNewValueFromOldValue:(MJNewValueFromOldValue)newValueFormOldValue
{
    objc_setAssociatedObject(self, &MJNewValueFromOldValueKey, newValueFormOldValue, OBJC_ASSOCIATION_COPY_NONATOMIC);
}
```

如何获取新？肯定需要兼容两种设置方式，一种`Block`,一种通过方法设置
```
+ (id)mj_getNewValueFromObject:(__unsafe_unretained id)object oldValue:(__unsafe_unretained id)oldValue property:(MJProperty *__unsafe_unretained)property{
    // 如果有实现方法
    if ([object respondsToSelector:@selector(mj_newValueFromOldValue:property:)]) {
    return [object mj_newValueFromOldValue:oldValue property:property];
    }
    // 兼容旧版本
    if ([self respondsToSelector:@selector(newValueFromOldValue:property:)]) {
        return [self performSelector:@selector(newValueFromOldValue:property:)  withObject:oldValue  withObject:property];
    }

    // 查看静态设置
    __block id newValue = oldValue;
    [self mj_enumerateAllClasses:^(__unsafe_unretained Class c, BOOL *stop) {
        MJNewValueFromOldValue block = objc_getAssociatedObject(c, &MJNewValueFromOldValueKey);
        if (block) {
            newValue = block(object, oldValue, property);
            *stop = YES;
        }
    }];
    return newValue;
}
```

#### **key配置**
`key`配置是解决属性名成需要重新定义的情况，这个配置统一通过`Block`设置。
这里注意到为什么在动态添加了属性之后需要将`cachedPropertiesDict_`字典里面的清空一次。如下：
```
#pragma mark - key配置
+ (void)mj_setupReplacedKeyFromPropertyName:(MJReplacedKeyFromPropertyName)replacedKeyFromPropertyName
{
[self mj_setupBlockReturnValue:replacedKeyFromPropertyName key:&MJReplacedKeyFromPropertyNameKey];

[[self propertyDictForKey:&MJCachedPropertiesKey] removeAllObjects];
}

+ (void)mj_setupReplacedKeyFromPropertyName121:(MJReplacedKeyFromPropertyName121)replacedKeyFromPropertyName121
{
objc_setAssociatedObject(self, &MJReplacedKeyFromPropertyName121Key, replacedKeyFromPropertyName121, OBJC_ASSOCIATION_COPY_NONATOMIC);

[[self propertyDictForKey:&MJCachedPropertiesKey] removeAllObjects];
}
```
这样做的目的是为了保证缓存数组中的数据是最新的。因为我们替换了属性的key，所以要用最新的。在获取所有属性中。有这么一段：
```
NSMutableArray *cachedProperties = [self propertyDictForKey:&MJCachedPropertiesKey][NSStringFromClass(self)];

if (cachedProperties == nil) {
    cachedProperties = [NSMutableArray array];

    [self mj_enumerateClasses:^(__unsafe_unretained Class c, BOOL *stop) {
    // 1.获得所有的成员变量
    
        ......
    }];
}
```
可以看到只有属性为空才会遍历类，获取最新属性。


#### **array model class配置**

这个方法是处理模型中包含一另一个模型数组。在实际运用比较多。
```
#pragma mark - array model class配置
+ (void)mj_setupObjectClassInArray:(MJObjectClassInArray)objectClassInArray
{
    [self mj_setupBlockReturnValue:objectClassInArray key:&MJObjectClassInArrayKey];

    [[self propertyDictForKey:&MJCachedPropertiesKey] removeAllObjects];
}
```
和上面的方式一样，会将最终会作为类的一个属性将模型数组字典保存下来。方便后面使用。

```

static const char MJReplacedKeyFromPropertyNameKey = '\0';
static const char MJReplacedKeyFromPropertyName121Key = '\0';
static const char MJNewValueFromOldValueKey = '\0';
static const char MJObjectClassInArrayKey = '\0';

static const char MJCachedPropertiesKey = '\0';
```
这是`NSObject+MJProperty.h`动态添加的所有属性。通过名字可以知道它的用途。

<br>



### **NSObject+MJKeyValue.h**

这个类中有一个很重要的协议`MJKeyValue`:
```
/**
*  只有这个数组中的属性名才允许进行字典和模型的转换
*/
+ (NSArray *)mj_allowedPropertyNames;

/**
*  这个数组中的属性名将会被忽略：不进行字典和模型的转换
*/
+ (NSArray *)mj_ignoredPropertyNames;

/**
*  将属性名换为其他key去字典中取值
*
*  @return 字典中的key是属性名，value是从字典中取值用的key
*/
+ (NSDictionary *)mj_replacedKeyFromPropertyName;

/**
*  将属性名换为其他key去字典中取值
*
*  @return 从字典中取值用的key
*/
+ (id)mj_replacedKeyFromPropertyName121:(NSString *)propertyName;

/**
*  数组中需要转换的模型类
*
*  @return 字典中的key是数组属性名，value是数组中存放模型的Class（Class类型或者NSString类型）
*/
+ (NSDictionary *)mj_objectClassInArray;

/**
*  旧值换新值，用于过滤字典中的值
*
*  @param oldValue 旧值
*
*  @return 新值
*/
- (id)mj_newValueFromOldValue:(id)oldValue property:(MJProperty *)property;

/**
*  当字典转模型完毕时调用
*/
- (void)mj_keyValuesDidFinishConvertingToObject;

/**
*  当模型转字典完毕时调用
*/
- (void)mj_objectDidFinishConvertingToKeyValues;
```
这里的协议规定了白名单、黑名单。之前分析过通过`block`的形式同样能够实现黑、白名单。我猜测这个文件是很久之前就有了为了兼容性，所以这种设置白、黑名单的方式一致保留着。

**这也是我们分析的最后一个类，可能也是最复杂的一个类**

分别是：

-  类方法
    - 错误定义
    - 转换字典`replace`设置
    - 模型转字典
        - 转所有属性
        - 制定部分属性转
        - 忽略部分属性转
        - 模型数组转字典数组
    - 字典转模型
        - 字典转为模型（可以是NSDictionary、NSData、NSString）
        - 字典转为模型（可以是NSDictionary、NSData、NSString）**CoreData支持**
    - 字典数组转模型数组
        - 字典数组来创建一个模型数组（可以是NSDictionary、NSData、NSString）
        - 字典数组来创建一个模型数组（可以是NSDictionary、NSData、NSString）**CoreData支持**
        - plist来创建一个模型数组
            - 仅限于mainBundle中的文件
            - 文件全路径
    -  模型转json、字典、数组
        - 转换为JSON Data
        - 转换为字典或者数组
        - 转换为JSON 字符串
-  对象方法
    -  字典转模型
        - 字典转为模型（可以是NSDictionary、NSData、NSString）
        - 字典转为模型（可以是NSDictionary、NSData、NSString）**CoreData支持**

上面所列的内容就是这个框架提供给使用者的所有功能了。这个部分就是把之前分析的内容串在一起。实现这些功能。

我们从最常用的`+ (instancetype)mj_objectWithKeyValues:(id)keyValues;`方法入手。从头到尾走一遍这个流程。

直接看最核心部分。入口就是`- (instancetype)mj_setKeyValues:(id)keyValues context:(NSManagedObjectContext *)context`这个方法。我们把外部的字典，json传入，最终把字典的`key`映射到对应的属性上，`value`成为这个属性的值。

#### **参数过滤**

第一步肯定是对参数合法性进行校验。
```
// 获得JSON对象
keyValues = [keyValues mj_JSONObject];

MJExtensionAssertError([keyValues isKindOfClass:[NSDictionary class]], self, [self class], @"keyValues参数不是一个字典");
```
经过转换后还不是字典类型就直接抛出异常。

#### **黑名单，白名单过滤**

接下来如果使用者配置了属性的白名单或者黑名单，则会对取出黑白名单。在遍历类的属性的时候过滤掉。
```
NSArray *allowedPropertyNames = [clazz mj_totalAllowedPropertyNames];
NSArray *ignoredPropertyNames = [clazz mj_totalIgnoredPropertyNames];
```
因为属性列表是在类对象上，所以自然去`NSObject+MJClass.h`调用。这个类主要功能就是提供了黑白名单的存储。
`NSObject+MJClass.h`中存储的黑白名单
```
static const char MJAllowedPropertyNamesKey = '\0'; // 白名单
static const char MJIgnoredPropertyNamesKey = '\0'; // 黑名单
static const char MJAllowedCodingPropertyNamesKey = '\0'; // 归档白名单
static const char MJIgnoredCodingPropertyNamesKey = '\0'; // 归档黑名单
```

#### **遍历类的所有属性**

接下来就是对类的每个属性处理，比如替换，忽略等。涉及到属性的是在`NSObject+MJProperty.h`类中完成的。比如遍历就是。

通过传入`block`,在遍历的同时对属性就行处理。形式就像通过`enumerateObjectsUsingBlock:`遍历。
```
[someArray enumerateObjectsUsingBlock:^(id _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
    ......
}]
```
遍历属性
```
[clazz mj_enumerateProperties:^(MJProperty *property, BOOL *stop) {
    ......
}];
```
如果属性在白名单或者黑名单中出现在则直接跳出这次循环。

```
if (allowedPropertyNames.count && ![allowedPropertyNames containsObject:property.name]) return;
if ([ignoredPropertyNames containsObject:property.name]) return;

```
取出属性对应的值，当然这里增加了对值得过滤，比如设置了新值替换旧的值。如果最终取出的结果中没有值，则直接返回。
```
// 0.检测是否被忽略
if (allowedPropertyNames.count && ![allowedPropertyNames containsObject:property.name]) return;
if ([ignoredPropertyNames containsObject:property.name]) return;

// 1.取出属性值
id value;
NSArray *propertyKeyses = [property propertyKeysForClass:clazz];
for (NSArray *propertyKeys in propertyKeyses) {
value = keyValues;
for (MJPropertyKey *propertyKey in propertyKeys) {
value = [propertyKey valueInObject:value];
}
if (value) break;
}

// 值的过滤
id newValue = [clazz mj_getNewValueFromObject:self oldValue:value property:property];
if (newValue != value) { // 有过滤后的新值
[property setValue:newValue forObject:self];
return;
}

// 如果没有值，就直接返回
if (!value || value == [NSNull null]) return;
```
接下里就是最终处理部分了，这时候属性的`key`和`value`都是合法的了。

- 1.剩下的就是处理属性。比如讲不可变数组转为可变数组。
- 2.如果不是`Foundation `框架的类，也就是继承至自定义模型的需要递归遍历。
- 3.对模型数组的处理
- 4.如果是`Foundation `框架中的。这个部分就可以直接给`value`赋值了。
- 5.最终`KVC`给属性赋值。`[property setValue:value forObject:self];`


<br>
<br>

# 总结
到这里基本MJExtension的源码也就解读完毕了。通过读源码，我们会学到很多平时没怎么注意的知识点，比如类型编码、属性的遍历、白名单配置、编译器警告的忽略等，另外就是开源框架的严谨业务逻辑
部分，很值得我们去学习的体会。
**阅读完源码，我也模仿着源码自己写了一个简单的字典转模型框架：**[**YBJson**](https://github.com/xxzizixx/YBJson)**：https://github.com/xxzizixx/YBJson**
如果你也阅读了源码，觉得体会很多，那么你也可以试着自己动手去模仿着写一下，印象会更深刻哟～










