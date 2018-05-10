---
title: '@property详解'
date: 2018-05-10 17:35:33
tags:
	- property
categories:
	- OC 
---

### 目录:

* 本质
* 修饰符
* synthesize, dynamic
* KVO 与 Property

### @property 本质：

> 概述: @property = ivar + getter + setter
</br>
</br>@property 其实是属性的声明标识符，提供成员变量的访问方法的声明、控制成员变量的访问权限、控制多线程时成员变量的访问环境
</br>
“属性” (property)作为 Objective-C 的一项特性，主要的作用就在于封装对象中的数据。 Objective-C 对象通常会把其所需要的数据保存为各种实例变量。实例变量一般通过“存取方法”(access method)来访问。其中，“获取方法” (getter)用于读取变量值，而“设置方法” (setter)用于写入变量值。这个概念已经定型，并且经由“属性”这一特性而成为 
</br>Objective-C 2.0 的一部分。 而在正规的 Objective-C 编码风格中，存取方法有着严格的命名规范。 正因为有了这种严格的命名规范，所以 Objective-C 这门语言才能根据名称自动创建出存取方法。

#### ivar，getter，setter 是什么？

* [ivar](http://blog.sunnyxx.com/2015/09/13/class-ivar-layout/)：作为变量存放的载体，是真正存放变量的（不完全正确，可能是指针），和其他语言的类下的变量含义一样

* getter：变量访问方法。变量的访问方法都可以称之为 getter。@property 自动的生成的 getter 是根据变量名有关的。

* setter：变量的修改/设置方法。变量的修改设置方法都可以称之为 setter。@property 自动的生成的 getter 是根据变量名有关的。


#### 代码 @property NSString *firstName 做了什么？

前面提到 @perperty 是 ivar，getter，setter 的集合，他们的命名是有规则的。

* 命名规则/规范： ivar，getter，setter
* ivar 名： _firstName
* getter 名：firstName
* setter 名： setFirstName
* 其他默认行为：
其他的默认行为和修饰符相关的，见修饰符的影响

#### 修饰符的影响：

* atomic：setter，getter 中添加 spinlock 来确定操作的原子性。atomaic 这个是默认行为。nonatomic 则不会添加 spinlock 相关代码
* readwrite：同时生成 getter，setter 方法，
* readonly：仅仅生成 getter
* strong，weak，assign：控制编译器自动添加 arc 相关代码。细节见修饰符
* retain, copy, assign：控制 setter 对于入参的处理。细节见修饰符

property 与 protocol，category：

1. 是否可以使用？

答：在 protocol 和 category 都可以使用

2. @perperty 究竟做了什么？

答：在 protocol 中只是声明了 setter，getter 方法；在 category 也只是声明了 setter 和 getter 方法，没有添加 ivar。

3. 如何在 category 中添加 变量？

答：通过 objc_setAssociatedObject，objc_getAssociatedObject 实现添加关联变量。

### 修饰符：

修饰符处理的对象其实是 getter，setter，定义了编译器对于 getter，setter 的合成相关操作

#### atomic与nonatomic

* atomic：默认是有该属性的，这个属性是为了保证程序在多线程情况，编译器会自动生成一些互斥加锁代码，避免该变量的读写不同步问题。

* nonatomic：如果该对象无需考虑多线程的情况，请加入这个属性，这样会让编译器少生成一些互斥加锁代码，可以提高效率。

注意：

* 针对的是指 getter，setter，而不是说 ivar

* 是通过自旋锁 SpinLock 来实现原子的。iOS 上默认是 atomic。

#### readwrite与readonly

* readwrite：这个属性是默认的情况，会自动为你生成存取器。
* readonly：只生成getter不会有setter方法。
* readwrite、readonly这两个属性通常是用来控制成员变量的访问权限。

#### strong与weak：

* strong：强引用，也是我们通常说的引用，其存亡直接决定了所指向对象的存亡。如果不存在指向一个对象的引用，并且此对象不再显示在列表中，则此对象会被从内存中释放。对于引用计数而言，引用计数会增加

* weak：弱引用，不决定对象的存亡。即使一个对象被持有无数个弱引用，只要没有强引用指向它，那么还是会被清除。引用计数不会改变，但是会把指针添加到对象的 weak table 中，为 nil 的时候将本指针也设置为 nil。

    weak runtime 实现：属性 ivar 的具体设置函数，我们假定为objc_storeWeak(&a, b) 函数
    
    objc_storeWeak 函数把第二个参数--赋值对象（b）的内存地址作为键值key，将第一个参数--weak修饰的属性变量（a）的内存地址（&a）作为value，注册到 weak 表中。如果第二个参数（b）为0（nil），那么把变量（a）的内存地址（&a）从weak表中删除，
    
    你可以把objc_storeWeak(&a, b)理解为：objc_storeWeak(value, key)，并且当key变nil，将value置nil。
    
    在b非nil时，a和b指向同一个内存地址，在b变nil时，a变nil。此时向a发送消息不会崩溃：在Objective-C中向nil发送消息是安全的。
    
* strong与retain功能相似；weak与assign相似，只是当对象消失后weak会自动把指针变为nil;

#### assign、copy、retain：

* assign：对于基本类型，默认是该修饰符（非基本类型默认是 strong）。setter方法直接赋值，不进行任何retain操作，不改变引用计数。一般用来处理基本数据类型。

* retain：释放旧的对象（release），将旧对象的值赋给新对象，再令新对象引用计数为1。我理解为指针的拷贝，拷贝一份原来的指针，释放原来指针指向的对象的内容，再令指针指向新的对象内容。

* copy：与retain处理流程一样，先对旧值release，再copy出新的对象，retainCount为1.为了减少对上下文的依赖而引入的机 制。我理解为内容的拷贝，向内存申请一块空间，把原来的对象内容赋给它，令其引用计数为1。对copy属性要特别注意：被定义有copy属性的对象必须要 符合NSCopying协议，必须实现- (id)copyWithZone:(NSZone *)zone方法。

* 拷贝有浅拷贝，深拷贝：oc 中主要表现为的 copy，mutableCopy。关于 copy 更多信息见[iOS 集合的深复制与浅复制](https://www.zybuluo.com/MicroCai/note/50592)

<div align=center><img width="60%" src="https://ws1.sinaimg.cn/large/ad3a9ce5gy1fr67tmzavbj20ne067ab1.jpg"/></div>

* 直接使用：

* 使用assign: 对基础数据类型 （NSInteger，CGFloat）和C数据类型（int, float, double, char, 等等）

* 使用copy： 对NSString

* 使用retain： 对其他NSObject和其子类

#### getter setter：

* getter：是用来指定get方法的方法名
* setter：是用来指定set访求的方法名
在@property的属性中，如果这个属性是一个BOOL值，通常我们可以用getter来定义一个自己喜欢的名字，例如：

* @property (nonatomic, assign, getter=isValue) boolean value;
* @property (nonatomic, assign, setter=setIsValue) boolean value;

#### retain, copy, assign区别

在 OC 中的内存管理的本质其实是通过引用计数来完成的，每个对象都有一个记录当前对象引用的次数，当引用次数为0的时候，才会去释放对象。 下面假设有新产生一个对象，a 作为指向这个对象的变量（指针），现将 a 使用不同的方式复制给 变量 b，

* assign： a 和 b指向同一块内存。assign 指示引用计数不增加，当前对象的引用计数依然为 1。而当 a 设置为 nil 的时候，由于引用计数变为 0，对象已经被释放，而 b 完全不知道，已经指向原来对象的地址，而原来的对象已经被是释放了，b 成为野指针。那么b在使用这块内存的时候会引起程序crash掉。
* retain： 在将 a 赋值给 b 的时候，a 和 b 都是指向同一个对象，该对象引用计数会 +1，变为 2。所以即使在 a = nil 的时候，对象的引用计数为1，所以依然不会释放，所以访问不会出现问题。在 b = nil 的时候，引用计数才会 0，才会被释放。
* copy：和上面 assign 和 retain 完全不同。assign 和 retain都是指向同一个对象，也就是共享一块内存。而 copy 指示，在复制的时候，并不是简单的赋值，而是拷贝原来的对象，然后将 b，指向这个拷贝生成的对象。由于 a 和 b 是两个不同的对象，仅仅是在值/内容相等。a = nil，对于 b 完全没有影响。


#### nonnull，nullablenull_resettable，_Null_unspecified：

* nonnull：不能为空
* nullable：表示可以为空
* null_resettable:
* get:不能返回空
* set可以为空
* 注意：如果使用null_resettable,必须 重写get方法或者set方法,处理传递的值为空的情况
* _Null_unspecified：不确定是否为空

#### assign 与 weak 比较：

1. 修饰变量类型的区别

* weak 只可以修饰对象。如果修饰基本数据类型，编译器会报错-“Property with ‘weak’ attribute must be of object type”。
* assign 可修饰对象，和基本数据类型。当需要修饰对象类型时，MRC时代使用unsafe_unretained。当然，unsafe_unretained也可能产生野指针，所以它名字是”unsafe_”

2. 是否产生野指针的区别

* weak 不会产生野指针问题。因为weak修饰的对象释放后（引用计数器值为0），指针会自动被置nil，之后再向该对象发消息也不会崩溃。 weak是安全的。
* assign 如果修饰对象，会产生野指针问题；如果修饰基本数据类型则是安全的。修饰的对象释放后，指针不会自动被置空，此时向对象发消息会崩溃。


#### @synthesize和@dynamic

property 是有 ivar ，getter，setter 构成。而这两个关键字处理的的是这三者间的关系的。编译器在编译期间完成。

#### @synthesize：自动合成

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


#### @dynamic：禁止自动合成

* 禁止自动合成getter, setter：告诉编译器属性的 setter 与 getter 方法由用户自己实现，不自动生成。（当然对于 readonly 的属性只需提供 getter 即可）

* 注意：属性被声明为 @dynamic var，然后你没有提供 @setter方法和 @getter 方法，编译的时候没问题，但是当程序运行到 instance.var = someVar，由于缺 setter 方法会导致程序崩溃； 或者当运行到 someVar = var 时，由于缺 getter 方法同样会导致崩溃。编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定

* 使用场景：

在CoreData的NSManagedObject类使用的某些。如果你想这些情况下，声明和使用属性，但要避免缺少方法在编译时的警告，你可以使用@dynamic动态指令，而不是@synthesize合成指令。


### KVO：

#### 什么是KVO？

KVO（Key Value Observing, 键值观察）是Objective-C对观察者模式的实现，每次当被观察对象的某个属性值发生改变时，注册的观察者便能获得通知。
使用：三个基本步骤：

1. 注册观察者，指定被观察对象的属性：

```
[obj addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionOld | NSKeyValueObservingOptionNew context:nil];
```
其中，person即为被观察对象，它的name属性即为被观察的属性。

2. 在观察者中实现以下回调方法：

```
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
```

3. 移除观察者

#### KVO 触发：

概述：触发依赖与以下三个函数：执行顺序如下

1. willChangeValueForKey: 在一个被观察属性发生改变之前调用，记录旧的值
2. observeValueForKeyPath:ofObject:change:context: 当改变发生后，调用该函数
3. didChangeValueForKey: 在 2 完成后执行

触发方式分类：

* 自动触发：注册KVO，在 value 发生变化的时候自动触发。property 相关属性属性默认都是自动触发的
* 手动触发： value 的变化，通过代码控制触发。

#### 手动触发 KVO：

* 手动触发 KVO，就是通知 value 变化。主要有两种情况

property 变量：默认情况下 property 的是自动触发，手动触发方法与步骤：

    * 重载 + (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key： key 为手动触发的 key 的返回 NO，其他调用父方法
    * 重写 setter 方法：在修改值前后分别调用，willChangeValueForKey: 和 didChangeValueForKey:

* 注意：

    * 上面步骤缺一不可
    * 如果步骤 1 缺失, 会导致 KVO 代码执行两次

* 非 property 变量：对于 property 变量，编译器会自动合成 getter 和 setter，而KVO 是运行期间的，依赖与 runtime。所以KVO 对于名字有严格的要求（setter）
* 方法：只需要在自己的 setter 中调用 willChangeValueForKey: 和 didChangeValueForKey: 即可。


#### 手动触发 KVO 使用场景：

* 自定义 setter 实现，setter 使用其他名字

* category 分类的添加的属性

* 使用：objc_setAssociatedObject，objc_getAssociatedObject 添加关联变量

关于 category 详情见 [objc category的秘密](http://blog.sunnyxx.com/2014/03/05/objc_category_secret/)


#### KVO 实现原理（Apple的实现）

核心：isa + setter

当你观察一个对象时，一个新的类会被动态创建。这个类继承自该对象的原本的类，并重写了被观察属性的 setter 方法。重写的 setter 方法会负责在调用原 setter 方法之前和之后，通知所有观察对象：值的更改。最后通过 isa 混写（isa-swizzling） 把这个对象的 isa 指针 ( isa 指针告诉 Runtime 系统这个对象的类是什么 ) 指向这个新创建的子类，对象就神奇的变成了新创建的子类的实例


<div align=center><img width="60%" src="https://ws1.sinaimg.cn/large/ad3a9ce5gy1fr68bazzl1j20qh0fvmzp.jpg"/></div>

### 参考：

* [iOS 基础知识回顾——关于 property](https://www.jianshu.com/p/294e9285361e)
