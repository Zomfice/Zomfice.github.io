---
title: '@synthesize @dynamic详解'
date: 2018-05-10 17:27:21
tags:
	- synthesize
	- dynamic
categories:
	- OC
---

### @synthesize @dynamic

> OC最初设定@property和@synthesize的作用：
</br>
</br> 1. @property的作用是定义属性，声明getter,setter方法。(注意：属性不是变量)
</br>
</br> 2. @synthesize的作用是实现属性的,如getter，setter方法。

在声明属性的情况下如果重写setter,getter,方法，就需要把未识别的变量在@synthesize中定义，把属性的存取方法作用于变量。如：.h文件中

后来因为使用@property灰常频繁，就简略了@synthesize的表达

从Xcode4.4以后@property已经独揽了@synthesize的功能主要有三个作用：

### @synthesize三个作用：

1. 生成了成员变量get/set方法的声明
2. 生成了私有的带下划线的的成员变量因此子类不可以直接访问，但是可以通过. get/set方法访问。那么如果想让定义的成员变量让子类直接访问那么只能在.h文件. 中. 定义成员变量了，因为它默认是@protected
3. 生成了get/set方法的实现

用@property声明的成员属性,相当于自动生成了setter getter方法,如果重写了set和get方法,与@property声明的成员属性就不是一个成员属性了,是另外一个实例变量,而这个实例变量需要手动声明。所以会报错误。

总结：一定要分清属性和变量的区别，不能混淆。@synthesize 声明的属性=变量。意思是，将属性的setter,getter方法，作用于这个变量。


### [@synthesize：自动合成](https://www.jianshu.com/p/294e9285361e)

* 自动合成 getter, setter：注意名字编译器会自动检查，如果没有手动实现，会自动添加 getter，setter，如果实现了，则不做处理
* 具体合成那些其实还和 ready write 相关修饰符相关的，默认为 readwrite
* 指定变量名字：@synthesize firstName = _myFirstName;
* 合成规则：假设声明为 @property NSString *firstName;
* 变量 ivar 名字ivar 变量名称为 _firstName
* getter, setter：getter 变量名称为 firstName， setter 为 setFirstName

那些情况下自动合成会失效

* 同时重写了 setter 和 getter 时
* 重写了只读属性的 getter 时
* 使用了 @dynamic 时
* 在 @protocol 中定义的所有属性
* 在 category 中定义的所有属性
* 重载的属性

### [@dynamic：禁止自动合成](https://www.jianshu.com/p/294e9285361e)

* 禁止自动合成getter, setter：告诉编译器属性的 setter 与 getter 方法由用户自己实现，不自动生成。（当然对于 readonly 的属性只需提供 getter 即可）

* 注意：属性被声明为 @dynamic var，然后你没有提供 @setter方法和 @getter 方法，编译的时候没问题，但是当程序运行到 instance.var = someVar，由于缺 setter 方法会导致程序崩溃； 或者当运行到 someVar = var 时，由于缺 getter 方法同样会导致崩溃。编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定

* 使用场景：

在CoreData的NSManagedObject类使用的某些。如果你想这些情况下，声明和使用属性，但要避免缺少方法在编译时的警告，你可以使用@dynamic动态指令，而不是@synthesize合成指令。



### 使用场景

* 场景1

```
ViewController.h

#import <UIKit/UIKit.h>  
  
@interface ViewController : UIViewController{
    NSMutableDictionary* _myDic;
}  
  
@property(nonatomic,strong)NSMutableDictionary*myDic; 
  
@end  
---------------------------------------------
ViewController.m

#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

@dynamic myDic;//告诉别人set&get都是自己重写了的

- (void)viewDidLoad {
    [super viewDidLoad];
}

-(NSMutableDictionary *)myDic{
    return _myDic;
}

-(void)setMyDic:(NSMutableDictionary *)myDic{
    _myDic = myDic;
}
```

* 场景2

```
ViewController.h

#import <UIKit/UIKit.h>  
  
@interface ViewController : UIViewController{
    NSMutableDictionary* _myDic;
}  
  
@property(nonatomic,strong,readonly)NSMutableDictionary*myDic; //这时候外部不能调用set方法，只能子类或该类内部才能调用
  
@end  
---------------------------------------------
ViewController.m

#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

@dynamic myDic;//告诉别人set&get都是自己重写了的

- (void)viewDidLoad {
    [super viewDidLoad];
}

-(NSMutableDictionary *)myDic{
    return _myDic;
}

-(void)setMyDic:(NSMutableDictionary *)myDic{
    _myDic = myDic;
}
```

* 场景3 @synthesize还可以用来自定义Property所对应的ivar的名称。例子如下：

```
#import <UIKit/UIKit.h>

@interface ZZYObject : UIView

@property(nonatomic,strong)NSMutableDictionary*myDic;

@end

-----------------------------

#import "ZZYObject.h"

@interface ZZYObject()

@end

@implementation ZZYObject

@synthesize myDic = myDicssss;


-(NSMutableDictionary *)myDic{
    return myDicssss;
}

-(void)setMyDic:(NSMutableDictionary *)myDic{
    myDicssss = myDic;
}

@end
```

* 场景4:在protocol中声明属性

* 场景5 [覆盖属性参考：](http://blog.csdn.net/jeffasd/article/details/50475608)


### 参考：

* [1. synthesize详解](https://www.cnblogs.com/handsomeBoys/p/5672352.html)
* [2. @synthesize应用场景详解](https://www.jianshu.com/p/94fb8b816147)
* [3. 自动合成和禁止](https://www.jianshu.com/p/294e9285361e)
* [4. @synthesize的妙用](https://www.jianshu.com/p/cee2e058ce82)
* [5. @synthesize 的作用](http://nextcocoa.com/synthesize-de-zuo-yong/)
