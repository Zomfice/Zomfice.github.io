---
title: __bridge，__bridge_transfer和__bridge_retained详解
date: 2018-04-23 17:17:48
tags:
	- bridge
	- bridge_transfer
categories:
	- OC
---

### [Objective-C和Core Foundation 对象相互转换的内存管理总结](http://www.cnblogs.com/HypeCheng/p/4686503.html)

iOS允许Objective-C 和 Core Foundation 对象之间可以轻松的转换，拿 NSString 和 CFStringRef 来说，直接转换毫无压力： [cpp] view plaincopyprint? 01. CFStringRef aCFString = (CFStringRef)aNSString; 02. NSString *aNSS


iOS允许Objective-C 和 Core Foundation 对象之间可以轻松的转换，拿 NSString 和 CFStringRef 来说，直接转换毫无压力：

```
    [cpp] view plaincopyprint?
01. CFStringRef aCFString = (CFStringRef)aNSString;  
02. NSString *aNSString = (NSString *)aCFString;  
```

针对内存管理问题，ARC 可以帮忙管理 Objective-C 对象, 但是不支持 Core Foundation 对象的管理，所以转换后要注意一个问题：谁来释放使用后的对象。 本文重点总结一下类型转换后的内存管理。


##### 一、非ARC的内存管理

倘若不使用ARC，手动管理内存，思路比较清晰，使用完，release转换后的对象即可。

```
    [cpp] view plaincopyprint?
01. //NSString 转 CFStringRef   
02. CFStringRef aCFString = (CFStringRef) [[NSString alloc] initWithFormat:@"%@", string];  
03. //...   
04. CFRelease(aCFString);  
05.  
06.  
07. //CFStringRef 转 NSString   
08. CFStringRef aCFString = CFStringCreateWithCString(kCFAllocatorDefault,  
09.                                                  bytes,  
10.                                                  NSUTF8StringEncoding);  
11. NSString *aNSString = (NSString *)aCFString;  
12. //...   
13. [aNSString release]; 
```

##### 二、ARC下的内存管理

ARC的诞生大大简化了我们针对内存管理的开发工作，但是只支持管理 Objective-C 对象, 不支持 Core Foundation 对象。Core Foundation 对象必须使用CFRetain和CFRelease来进行内存管理。那么当使用Objective-C 和 Core Foundation 对象相互转换的时候，必须让编译器知道，到底由谁来负责释放对象，是否交给ARC处理。只有正确的处理，才能避免内存泄漏和double free导致程序崩溃。

根据不同需求，有3种转换方式

*  __bridge                                           （不改变对象所有权）
*  __bridge_retained 或者 CFBridgingRetain()           （解除 ARC 所有权）
*  __bridge_transfer 或者 CFBridgingRelease()          （给予 ARC 所有权） 

**1. __bridge_retained 或者 CFBridgingRetain()**

__bridge_retained 或者 CFBridgingRetain()  将Objective-C对象转换为Core Foundation对象，把对象所有权桥接给Core Foundation对象，同时剥夺ARC的管理权，后续需要开发者使用CFRelease或者相关方法手动来释放对象。

例子：

```
    [cpp] view plaincopyprint?
01. - (void)viewDidLoad  
02. {  
03.     [super viewDidLoad];  
04.      
05.     NSString *aNSString = [[NSString alloc]initWithFormat:@"test"];  
06.     CFStringRef aCFString = (__bridge_retained CFStringRef) aNSString;  
07.      
08.     (void)aCFString;  
09.      
10.     //正确的做法应该执行CFRelease   
11.     //CFRelease(aCFString);    
12.} 

```

程序没有执行CFRelease，造成内存泄漏：


```
CFBridgingRetain()  是 __bridge_retained 的宏方法，下面两行代码等价：

    [cpp] view plaincopyprint?
01. CFStringRef aCFString = (__bridge_retained CFStringRef) aNSString;  
02. CFStringRef aCFString = (CFStringRef) CFBridgingRetain(aNSString); 

```

**2. __bridge_transfer 或者 CFBridgingRelease()**

__bridge_transfer 或者 CFBridgingRelease()  将非Objective-C对象转换为Objective-C对象，同时将对象的管理权交给ARC，开发者无需手动管理内存。

接着上面那个内存泄漏的例子，再转成OC对象交给ARC来管理内存，无需手动管理，也不会出现内存泄漏：

```
    [cpp] view plaincopyprint?
01. - (void)viewDidLoad  
02. {  
03.     [super viewDidLoad];  
04.      
05.     NSString *aNSString = [[NSString alloc]initWithFormat:@"test"];  
06.     CFStringRef aCFString = (__bridge_retained CFStringRef) aNSString;  
07.     aNSString = (__bridge_transfer NSString *)aCFString;  
08. }  

```

CFBridgingRelease() 是__bridge_transfer的宏方法，下面两行代码等价：

```
    [cpp] view plaincopyprint?
01. aNSString = (__bridge_transfer NSString *)aCFString;  
02. aNSString = (NSString *)CFBridgingRelease(aCFString);

```

**3. __bridge**

__bridge 只做类型转换，不改变对象所有权，是我们最常用的转换符。

从OC转CF，ARC管理内存：

```
    [cpp] view plaincopyprint?
01. - (void)viewDidLoad  
02. {  
03.     [super viewDidLoad];  
04.      
05.     NSString *aNSString = [[NSString alloc]initWithFormat:@"test"];  
06.     CFStringRef aCFString = (__bridge CFStringRef)aNSString;  
07.      
08.     (void)aCFString;  
09. }  

```

从CF转OC，需要开发者手动释放，不归ARC管：

```
    [cpp] view plaincopyprint?
01. - (void)viewDidLoad  
02.   
03.     [super viewDidLoad];  
04.      
05.     CFStringRef aCFString = CFStringCreateWithCString(NULL, "test", kCFStringEncodingASCII);  
06.     NSString *aNSString = (__bridge NSString *)aCFString;  
07.      
08.     (void)aNSString;  
09.      
10.     CFRelease(aCFString);  
11. } 

```


#### 参考:

* [Objective-C和Core Foundation 对象相互转换的内存管理总结](http://www.cnblogs.com/HypeCheng/p/4686503.html)