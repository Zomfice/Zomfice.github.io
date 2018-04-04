---
title: C 语言中结构体的定义和使用
date: 2018-04-04 14:19:16
tags:
- Struct
categories:
- C语言
---

### 1. 声明

一般形式

struct 结构名{ 

成员列表 

}; 
struct关键字表示接下来是一个结构

如：声明一个学生的结构

```
struct Student{         //声明结构体
    char name[20];      //姓名
    int num;            //学号
    float score;        //成绩
};
```

### 2. 定义

struct 结构体名 结构体变量名; 

```
struct Student stu1;    //定义结构体变量
```

#### 1. 结构体变量的定义可以放在结构体的声明之后:

```
struct Student{         //声明结构体
    char name[20];      //姓名
    int num;            //学号
    float score;        //成绩
};
struct Student stu1;    //定义结构体变量
```

#### 2. 结构体变量的定义也可以与结构体的声明同时

```
struct Student{        
    char name[20];       
    int num;             
    float score;         
}stu1;                  //在定义之后跟变量名
```

#### 3. 可以使用匿名结构体来定义结构体变量

```
struct {                //没有结构名
    char name[20];       
    int num;            
    float score;         
}stu1;  
```

### 3. 访问


虽然结构类似一个数组，只是数组元素的数据类型是相同的，而结构中元素的数据类型是可以不同的。但结构不能像数组那样使用下标去访问其中的各个元素，而应该用结构成员运算符点（.）。即访问成员的一般形式是： 
结构变量名 . 成员名 
如 stu1 . name 表示学生stu1的姓名。

```
struct Birthday{                //声明结构体 Birthday
    int year;
    int month;
    int day;
};
struct Student{                 //声明结构体 Student
    char name[20];              
    int num;                    
    float score;                 
    struct Birthday birthday;   //生日
}stu1;
```
则用 stu1.birthday.year 访问出生的年份。

### 4. 初始化

```
struct Student stu1, stu2;      //定义结构体变量
strcpy(stu1.name, "Jack");
stu1.num = 18;
stu1.score = 90.5;
```
详细初始化过程查看参考1

### 5. 结构体数组

结构类型作为一种数据类型，也可以像基本数据类型那样，作为数组的元素的类型。元素属于结构类型的数组成为结构型数组。

* 定义

```
struct 结构名 { 
成员列表 
} 数组名[数组长度]; 
```

```
struct Student{                 //声明结构体 Student
    char name[20];
    int num;
    float score;
}stu[5];                        //定义一个结构结构数组stu，共有5个元素
```

* 初始化

```
struct Student stu[2] = {{"Mike", 27, 91},{"Tom", 15, 88.0}}; //定义结构数组的同时进行初始化

stu[2] = (struct Student){"Jack", 12, 85.0};//先定义后初始化
```


### 6. 结构体与指针

当一个指针变量用来指向了一个结构变量，这个指针就成了结构指针变量。 
结构指针变量中的值是所指向的结构变量的首地址。可以通过指针来访问结构变量。

1）定义结构指针变量的一般形式： 
struct 结构名 * 结构指针变量名 

```
struct Student *pstu;       //定义了一个指针变量，它只能指向Student结构体类型的结构体变量
```
结构指针变量的定义也可以与结构体的定义同时。而且它必须先赋值后使用。 
数组名表示的是数组的首地址，可以直接赋值给数组指针。但结构变量名只是表示整个结构体变量，不表示结构体变量的首地址，所以不能直接赋值给结构指针变量，而应该使用 & 运算符把结构变量的的地址赋值给结构指针变量。即：

```
pstu = stu1;//false
pstu = &stu2; //OK
```
注意：结构名、结构变量名、结构体体指针的区别


2）通过结构指针间接访问成员值

访问的一般形式： 
（*结构指针变量）. 成员名 或 结构指针变量 -> 成员名 
如：

```
(*pstu).name
pstu->name
```
注意(pstu)的小括号不能省略，因为成员符“.”优先级为1，取地址符“”优先级为2，去掉括号就相当于*(pstu.name)了。



### 参考：

* [1. C语言中结构体指针的定义和引用](https://blog.csdn.net/marlene0312/article/details/4746951)
* [2. C语言中的结构体(struct)](https://blog.csdn.net/yanggangclcsdn/article/details/49718131)
* [3. C语言typedef：给类型起一个别名](http://c.biancheng.net/cpp/html/100.html)