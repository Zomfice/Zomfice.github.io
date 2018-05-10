---
title: protocol属性 category属性 覆盖属性
date: 2018-05-10 17:37:21
tags:
	- protocol属性
	- category属性
categories:
	- OC
---

### protocol属性

> protocol属性只有对应的set/get方法声明,没有对应成员变量,因为协议中只可以声明方法，分类中只能声明方法和对应的实现。

详解: 在protocol中,通过查看runtime源码,没有Ivar实例变量,@property只是对set/get方法声明

```
@interface Protocol : Object
{
@private
    char *protocol_name OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocol_list OBJC2_UNAVAILABLE;
    struct objc_method_description_list *instance_methods OBJC2_UNAVAILABLE;
    struct objc_method_description_list *class_methods OBJC2_UNAVAILABLE;
}
```

由于遵守协议的类中，存取器的生成只是根据成员变量而生成的。而在协议中没有成员变量存在，所有就没有对应的set/get方法实现。而synthesize 实例变量 = _实例变量,系统会自动生成实例变量,也会生成set/get方法。

注:(解释为什么分类和协议中无法自动生成实例变量和set/get方法)

如果已经手动实现了get和set方法的话Xcode不会再自动生成带有下划线的私有成 员变量了
因为xCode自动生成成员变量的目的就是为了根据成员变量而生成get/set方法的
但是如果get和set方法缺一个的话都会生成带下划线的变量[详情查看](https://www.jianshu.com/p/cee2e058ce82)

1. 定义一个mmNum属性,重写get/set方法,如果写@synthesize mmNum = _mmNum;将会 Use of undeclared identifier '_mmNum';did you mean 'mmNum'

通过clang将.m文件转成cpp.之后看到有set/get方法,但是没有实例变量存在

```
clang -rewrite-objc MM.m
```

```

static void _I_MM_setMmNum_(MM * self, SEL _cmd, int mmNum) {

}

static int _I_MM_mmNum(MM * self, SEL _cmd) {
    return 0;
}

```

当加上@synthesize mmNum = _mmNum;可以看到实例变量生成,同时@synthesize将set/get方法作用实例变量

```
// @interface MM ()<MMProtocol>
// @property (nonatomic,assign) int mmNum;
/* @end */

// @implementation MM
// @synthesize mmNum = _mmNum;

struct MM_IMPL {
	struct NSObject_IMPL NSObject_IVARS;
	int _mmNum;
};


static struct /*_ivar_list_t*/ {
	unsigned int entsize;  // sizeof(struct _prop_t)
	unsigned int count;
	struct _ivar_t ivar_list[1];
} _OBJC_$_INSTANCE_VARIABLES_MM __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	sizeof(_ivar_t),
	1,
	{{(unsigned long int *)&OBJC_IVAR_$_MM$_mmNum, "_mmNum", "i", 2, 4}}
};
```

2. 在Protocol定义属性,只是对set/get方法声明,不加synthesize的时候,类中是没有对应的实例变量和方法实现的。当我们添加对应的@synthesize 属性 = _属性的时候,发现方法实现

```
extern "C" unsigned long OBJC_IVAR_$_MM$_proNum;
struct MM_IMPL {
	struct NSObject_IMPL NSObject_IVARS;
	int _mmNum;
	int _proNum;
};
```

```
static int _I_MM_proNum(MM * self, SEL _cmd) { return (*(int *)((char *)self + OBJC_IVAR_$_MM$_proNum)); }
static void _I_MM_setProNum_(MM * self, SEL _cmd, int proNum) { (*(int *)((char *)self + OBJC_IVAR_$_MM$_proNum)) = proNum; }

static struct /*_ivar_list_t*/ {
	unsigned int entsize;  // sizeof(struct _prop_t)
	unsigned int count;
	struct _ivar_t ivar_list[2];
} _OBJC_$_INSTANCE_VARIABLES_MM __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	sizeof(_ivar_t),
	2,
	{{(unsigned long int *)&OBJC_IVAR_$_MM$_mmNum, "_mmNum", "i", 2, 4},
	 {(unsigned long int *)&OBJC_IVAR_$_MM$_proNum, "_proNum", "i", 2, 4}}
};
```

结论:推断结论是,属性的set/get方法是根据实例变量来实现,由于protocol没有ivars实例变量对象,所以,protocol中的属性,只是相当于set/get方法声明,而要实现set/get方法,需要指定相应的实例变量,以此来让编译器去生成get/set方法。


### category属性

分类中添加属性,由于分类中只能声明方法和实习方法,不能生成实例变量,所以分类中不能自动生成set/get方法,而且需要运行时去关联方法和对象,runtime源码中

```
struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
    // Fields below this point are not always present on disk.
    struct property_list_t *_classProperties;

    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }

    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
};

```

分类中添加属性举例:
```
@interface NSObject (Extension)
@property (nonatomic,strong) NSString *myTitle;
@end

#import <objc/runtime.h>
@implementation NSObject (Extension)
- (NSString *)myTitle {
    return objc_getAssociatedObject(self, @selector(myTitle));   
}
- (void)setMyTitle:(NSString *)myTitle {
    objc_setAssociatedObject(self, @selector(myTitle), myTitle,OBJC_ASSOCIATION_RETAIN);
}
```

### 属性覆盖

* 首先看一个例子：ZZYObject2是ZZYObject1的字类，它们都有一个同样的属性myName

类ZZYObject1的代码：

```
#import <Foundation/Foundation.h>

@interface ZZYObject1 : NSObject

@property(nonatomic,copy)NSString* myName;

@end

------------------------------------------------------------------

#import "ZZYObject1.h"

@implementation ZZYObject1

-(void)setMyName:(NSString *)myName{
    _myName = @"我是ZZYObject1的set 方法";
}

@end
```

类ZZYObject2的代码：

```
#import "ZZYObject1.h"

@interface ZZYObject2 : ZZYObject1

@property(nonatomic,copy)NSString* myName;

@end

------------------------------------------------------------------

#import "ZZYObject2.h"

@implementation ZZYObject2


@end
```
在ViewController调用代码如下：

```
#import "ViewController.h"
#import "ZZYObject2.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    ZZYObject2*z2 = [[ZZYObject2 alloc]init];
    z2.myName = @"dsada";
    NSLog(@"我set了ZZYObject2 %@",z2.myName);
}

```
* 我们发现：ZZYObject2中的@property(nonatomic,copy)NSString* myName;这句代码有警告：“ Auto property synthesis will not synthesize property 'myName'; it will be implemented by its superclass, use @dynamic to acknowledge intention”。上面打印的结果为：

```
2017-02-24 11:02:14.389732 FugaiShuXing[10620:3285829] 我是ZZYObject1的set 方法
```

* 上面打印的并不是我们想要的结果，上面走的是父类ZZYObject1的set方法。原来在子类覆盖父类的属性时，编译器不会为子类合成带下划线的实例变量以及setter和getter方法，则需要自己来实现这些东西，否则这个属性将由父类实现，也就是说如果子类没有手写set和get方法，声明的@property相当于没写。

* 我们在ZZYObject2实现文件的代码加上一句@synthesize myName = _myName;（或者用“@dynamic完全接管property的方式”也可以，当然正如上面的警告系统也是推荐的@dynamic）就可以了。

注：@dynamic完全接管property的方式，可参考应用场景1或2更正后的写法

```
#import "ZZYObject2.h"

@implementation ZZYObject2

@synthesize myName = _myName;


@end
```


### 参考

* [属性覆盖](https://www.jianshu.com/p/94fb8b816147)
