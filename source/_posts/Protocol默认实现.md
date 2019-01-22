---
title: Protocol默认实现
date: 2019-01-22 17:11:40
tags:
    - Protocol
categories: 
    - OC
---

# Protocol默认实现


需求：

Protocol作为方法声明必须实现后才能才能进行操作处理，有没有办法可以实现协议不用实现就可以有默认值？

libextobjc 是一个提供语言级别的各种小功能的库。

### 用法

```
MyProtocol.h

@protocol MyProtocol <NSObject>

@concrete
- (void)testProtocol;

@end

MyProtocol.m
@concreteprotocol(MyProtocol)

- (void)testProtocol{
NSLog(@"%s",__func__);
}

@end

```

在`EXTConcreteProtocol.h` 中找到 `concreteprotocol` `concrete` 。
##### concrete

`concrete` 其实是对Optional的宏定义。作用1、是使用Optional的特性，声明协议方法，不实现没有⚠️2、是可以直观区分那些协议方法是默认实现的

```
#define concrete \
optional
```
##### concreteprotocol

`concreteprotocol` 传入Protocol的名称，和ProtocolMethodContainer拼接成一个类名,创建一个类去包含用在`Protocol` 中所有方法

宏定义内容如下：

```
#define concreteprotocol(NAME) \
interface NAME ## _ProtocolMethodContainer : NSObject < NAME > {} \
@end \
@implementation NAME ## _ProtocolMethodContainer \
+ (void)load { \
if (!ext_addConcreteProtocol(objc_getProtocol(metamacro_stringify(NAME)), self)) \
fprintf(stderr, "ERROR: Could not load concrete protocol %s\n", metamacro_stringify(NAME)); \
} \
__attribute__((constructor)) \
static void ext_ ## NAME ## _inject (void) { \
ext_loadConcreteProtocol(objc_getProtocol(metamacro_stringify(NAME))); \
}
```

传入NAME翻译如下：

```
@interface MyProtocol_ProtocolMethodContainer : NSObject <MyProtocol> {
}

@implementation MyProtocol_ProtocolMethodContainer

+ (void)load {
ext_addConcreteProtocol(objc_getProtocol”MyProtocol”,self);
}

__attribute__((constructor))
Static void ext_MyProtocol_inject (void) {
ext_loadConcreteProtocol(ojbc_getProtocol("MyProtocol"));
}

- (void)testProtocol{
NSLog(@"%s",__func__);
}

@end
```

### 分析

#### __attribute__((constructor))

`__attribute__`􏰊􏰋􏱋􏰚􏰛􏰱􏱶􏱷是一套编译器指令，被GNU和LLVM所支持，允许对__attribute__增加一些参数，做一些高级检查和优化。

`__attribute__` 用法是，在后面加两个括号，然后写属性列表，属性列表以逗号分隔。

`__attribute__((attribute1,attribute2))`;


#### constructor/destructor

`constructor` 属性表示在main函数执行之前，执行一些操作。`destructor` 表示在main函数执行之后做一些操作。`constructor`执行实际是在所有`load` 方法都执行完之后，才执行所有`constructor` 属性修饰的函数

```
__attribute__((constructor)) static void beforeMain(){
NSLog(@"brforeMian");
}
__attribute__((destructor)) static void afterMain(){
NSLog(@"afterMian");
}
int main(int argc, char * argv[]) {
@autoreleasepool {
NSLog(@"Mian");
}
}

```

有关`constructor` 的更多用法请参考[RuntimePDF](https://github.com/DeveloperErenLiu/RuntimePDF)，其中还有多个`constructor` 设置优先级的用法

#### ext_addConcreteProtocol 

添加待处理的Protocol

```
BOOL ext_addConcreteProtocol (Protocol *protocol, Class containerClass) {

return ext_loadSpecialProtocol(protocol, ^(Class destinationClass){

ext_injectConcreteProtocol(protocol, containerClass, destinationClass);

});

}
```
Protocol:默认实现的Protocol

```
BOOL ext_loadSpecialProtocol (Protocol *protocol, void (^injectionBehavior)(Class destinationClass)) {
// 默认实现的 protocol 个数是否大于 SIZE_MAX
// specialProtocolCount 默认实现的 protocol 的计数
if (specialProtocolCount == SIZE_MAX) {
return NO;
}

// specialProtocolCapacity 为数组总容量
if (specialProtocolCount >= specialProtocolCapacity) {
// 如果未超过 SIZE_MAX 则进行动态扩容
// 将动态扩容后的数组头指针交给 specialProtocols
}

// 将参数的 block copy 到堆，并赋值给 copiedBlock
ext_specialProtocolInjectionBlock copiedBlock = [injectionBehavior copy];

// 将 protocol, block, 和 NO 组装成 struct
// 然后将 struct 追加到数组中
specialProtocols[specialProtocolCount] = (EXTSpecialProtocol){
.protocol = protocol,
.injectionBlock = (__bridge_retained void *)copiedBlock,
.ready = NO
};

// 默认实现 protocol 的个数自增
++specialProtocolCount;

// success!
return YES;
}
```
整个函数走下来，作用就是将 {protocol, block} 追加到数组中。




#### ext_loadConcreteProtocol 

方法注入

```
void ext_loadConcreteProtocol (Protocol *protocol) {

ext_specialProtocolReadyForInjection(protocol);

}

```
确保 +load 中加入数组的所有 protocol 都能找到：

```
void ext_specialProtocolReadyForInjection (Protocol *protocol) {
// 循环遍历数组
for (size_t i = 0;i < specialProtocolCount;++i) {
// 如果数组的 protocol 是当前 protocl
if (specialProtocols[i].protocol == protocol) {
// 并且这个 protocol 还未被遍历过(也就是 ready 标识)
if (!specialProtocols[i].ready) {
// 则进行标记
specialProtocols[i].ready = YES;
// ready 标识计数自增
// !!! 当所有的 protocol 均 ready 之后
// 再调用 ext_injectSpecialProtocols
if (++specialProtocolsReady == specialProtocolCount)
ext_injectSpecialProtocols();
}

break;
}
}
}
```

+load 与 `__attribute__((constructor))` 的优先级能使得所有 protocol 加入完成以后，再进行处理。
ready 计数 specialProtocolsReady 使得所有默认实现均判断无误后，再进行注入。

#### ext_injectSpecialProtocols

优先级问题：如果 protocolA <ProtocolB>，也就是 protocolA 遵循 protocolB，那么谁的优先级更高呢？除此之外，如果遵循 protocol 的 class，自己也实现了默认方法呢？

```
static void ext_injectSpecialProtocols (void) {

qsort_b(specialProtocols, specialProtocolCount, sizeof(EXTSpecialProtocol), ^(const void *a, const void *b){
// 根据 a 是否 comform b，对整个数组进行排序 (protocol_conformsToProtocol)
});

// 通过 objc_getClassList 获得所有类列表

// 两个 for 循环嵌套
// 对类列表以及 protocol 列表进行遍历
// 如果 class comform protocol
// 则调用之前 struct 中的注入 block，进行注入
for (size_t i = 0;i < specialProtocolCount;++i) {
Protocol *protocol = specialProtocols[i].protocol;

for (unsigned classIndex = 0;classIndex < classCount;++classIndex) {
Class class = allClasses[classIndex];

if (!class_conformsToProtocol(class, protocol))
continue;

// 遵循 protocol 的 class 即为注入的目标 class
injectionBlock(class);
}
}
}
```

所以，整个方法的任务也很清晰：

对 protocol 进行优先级排序，给出具体注入的先后顺序，防止方法覆盖或无法注入。

获取全部 class 列表。

两层循环遍历，将 class 与其遵循的 protocol 进行匹配。

调用 struct 中的 block，并将目标 class 传出，进行注入。

#### ext_injectConcreteProtocol (block作用)

block 是在 +load 中就已经赋值了，而 block 的实现，就是直接调用了 ext_injectConcreteProtocol 函数：

```
// 函数有三个参数
// protocol
// containerClass: 实现了 protocol 方法的 容器类
// class: 两层循环嵌套中，找到的要注入的目标 class
static void ext_injectConcreteProtocol (Protocol *protocol, Class containerClass, Class class) {

// 获取 容器类 中的实例方法列表
// 获取 容器类 meta class 中的类方法列表

// 循环注入实例方法
for (unsigned methodIndex = 0;methodIndex < imethodCount;++methodIndex) {
// 获取方法 SEL
// 获取方法 IMP
// 判断 目标类 是否存在该方法
// 进行注入
}
// 循环注入类方法同理
}
```

### 小结

1. 注入实现思路的重点在于，使用宏为 protocol 扩展了一个容器类。
容器类中，利用 +load 与 __attribute__((constructor)) 的特性，将注入流程分为了两个部分。

2. 在 +load 中，将 protocol，执行注入的 block 打包成 struct，然后将 struct 装进数组。

3. 当执行到 __attribute__((constructor)) 时，也就表示所有类的 +load 都已经执行过了，再对数组进行优先级排序。

4. 排序完成后，两层循环嵌套，查找遵循了 protocol 的 class。
调用 block 执行注入。
