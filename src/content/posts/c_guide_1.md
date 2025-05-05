---
title: C语言教程(1)-常变量、关键字和流程控制
published: 2025-04-28
description: '做项目时候作为字典的笔记'
image: ''
tags: [Cookbook,C]
category: 'Cookbook'
draft: false 
lang: ''
---

# 基础语法

## 关键字

C语言关键字如下：
| 关键字分类 | 例子 |
|:---|:---|
| 数据类型声明 | `int`, `float`, `char`, `void`, `struct`, `union`, `enum` |
| 控制语句 | `if`, `else`, `switch`, `case`, `default`, `for`, `while`, `do`, `break`, `continue`, `goto`, `return` |
| 存储类型 | `auto`, `static`, `extern`, `register`, `const`, `volatile` |
| 其他 | `sizeof`, `typedef` |

:::important
- `auto`关键字在C语言中不用显式声明
- `static`比较重要，具体用法如下：
    1. **私有函数或变量**：在`.c`文件中声明和定义static变量或函数，类似于Java或Go中的私有函数或变量的用法，避免对外暴露
    ```c
    // module.c
    static void helper() { ... }      // 内部使用
    int public_api() { helper(); }    // 提供公开接口

    // module.h
    int public_api();   // 只暴露接口
    ```
    2. **防止符号污染**：由于是各`.c`文件私有的变量或函数，因此不存在调用不同库时命名撞车的问题，也因此static变量不要在`.h`文件中定义
    ```c
    // moduleA.c
    int count = 5;

    // moduleB.c
    int count = 6;          //Error 重复定义
    static int count = 6；  //OK
    ```
    3. **局部静态变量**：在函数体中定义，可以使该局部变量保留上次的值，不因为函数初始化或结束调用而被初始化或销毁，通常用于设计`计数器`或`状态机`。局部静态变量在全局数据区分配内存，且如果没有显式初始化的话会被初始化为`0`.
    ```c
    void counter_func() {
        static int count = 0; // 只初始化一次
        count++;              // 每次调用加一
        printf("This function has been called %d times.\n", count);
    }

    int main() {
        counter_func();  // 输出 1
        counter_func();  // 输出 2
        counter_func();  // 输出 3
        return 0;
    }
    ```
    4. 搭配`inline`关键字实现静态内联函数，用宏定义作全局替换，一般拿来写工具函数：
    ```c
    // math_utils.h
    #ifndef MATH_UTILS_H
    #define MATH_UTILS_H

    static inline int max(int a, int b) {
        return a > b ? a : b;
    }

    #endif
    ```
- `extern关键字也很特殊`,主要作用如下:
    1. C语言中一般不用`static`关键字修饰的变量都是`extern`的
    2. 用于在.h文件中声明并暴露出去，保证变量唯一，控制全局重名，防止修改
    ```c
    global.h：
    extern const int MAX_SIZE;
    global.c：
    const int MAX_SIZE = 1024;
    main.c：
    #include "global.h"
    printf("%d", MAX_SIZE); // OK
    ```
    这里其实是因为const修饰的变量或函数默认为`局部只读变量`，需要再用extern暴露出去
    3. 在C++中使用`extern "C" void c_function();`来调用C代码的函数

- `register`关键字现在一般不使用了，它的作用是手动把变量放寄存器里提高访问速度，
适合修饰临时的循环变量，在单片机裸机编程中可能会遇到

- `volatile`关键字表示告诉编译器该变量的值可能随时发生改变，不要优化其读写操作(**阻止编译器优化**)：
  
  - 一般用于**中断**或者**多线程共享变量**的处理，
    - 也可能**随外部变化的硬件寄存器的值**（如 UART 收发缓冲区）
    - 示例如下，若不加volatile，编译器可能认为 UART0_DR 永远不会变，从而优化掉访问操作，导致程序无法与外设通信。
    ```c
    #define UART0_DR (*(volatile unsigned int *)0x101f1000)
    void uart_send_char(char c) {
        UART0_DR = c;  // 写入外设寄存器
    }
    ```
    - 即使使用 volatile，某些架构如ARM仍然可能指令重排序(Out-of-Order Execution)，会破坏时序依赖。
    因此需要使用`内存屏障（memory barrier`或`编译器屏障(compiler barrier)`
    
    C语言内联汇编屏障示例(GCC)：`#define COMPILER_BARRIER() __asm__ __volatile__("" ::: "memory")`
    
    ARM架构下完整屏障:`_asm__ __volatile__("dmb ish" ::: "memory");// 内存屏障：确保前面的操作在后面的操作前完成`
    
    因此它还有个作用是**强制每次读写都访问真实内存**，也因此它不是原子性的

    | 场景 | 示例 |
    |------|------|
    | 中断标志 | `volatile int irq_flag;` |
    | 多核/多线程共享变量 | `volatile int status;` |
    | 外设状态寄存器 | `#define UART_TX (*(volatile uint8_t *)0x40011000)` |
    | 系统时钟等周期更新变量 | `volatile unsigned long system_tick;` |

  ```c
  volatile int flag;

  void isr_handler() {
      flag = 1;  // 中断中修改 flag
  }

  void main_loop() {
      while (flag == 0);  // 不会被优化成死循环
      // 执行后续逻辑
  }
  ```
:::

## 常量

这里主要是强调下字符串常量：

C语言中对于字符串常量的存储主要有两种方式：字符串指针和字符数组

```c
    char *string = "Hello, World!";  //字符串指针方式
    char string[] = "Hello, World!"; //字符数组方式
```

- 一般指针形式用于存储固定的只读字符串,也就是写成`const char *`方式,可以改变指针指向的字符串，但不可以更改字符串本身
- 数组方式的字符串是可以通过下标访问的方式修改字符串的某些位，因为此时字符串相当于复制到了栈上
- 字符串数组使用`strlen`来获取长度
:::important
- 无论何种方式，**字符串本身是不可更改的**，你能更改的只有他的副本
- 无论何种方式，操作字符串之前先拷贝一份是好习惯
- 不要忘记字符串最后隐藏的`\0`
:::



## 变量

### 变量类型和注意点
以标准Linux x86_64和32位arm架构为例，在使用gcc编译的情况下，C语言全部类型关键字和占用大小如下:

| 类型 | 说明 | 32位 | 64位| 备注
|-------------|---------------------------|-------|---------|---
| `char`      | 字符型                    | 1字节 | 1字节   |-
| `short`     | 短整型                    | 2字节 | 2字节   |`short int`
| `int`       | 整型                      | 4字节 | 4字节   |16位机器上占2字节
| `long`      | 长整型                    | 4字节 | 8字节   |`long int`
| `long long` | 长长整型                  | 8字节 | 8字节   |-
| `float`     | 单精度浮点型              | 4字节 | 4字节   |-
| `double`    | 双精度浮点型              | 8字节 | 8字节   |-
| `void`      | 空类型，无实际存储大小    | -     | -       |-

注意以下几个点：
- 对于整型变量，`short`至少占用2字节，但是int和long视系统和编译器而定，但是依次大于等于对方，建议不清楚的时候用`sizeof`看下
- 关于各类型的取值范围,记不住的话查下`limits.h`头文件就可以了
- 所有变量都包括`signed`和`unsigned`两类，默认都是`signed`，也就是带符号的。这里面char比较特殊：
  - char本质上是一个8位的小整数，8位就是1个字节
  - 因此有无符号位导致这个小整数代表的实际的数不一样，因此当执行强制类型转换后，由于`符号扩展`和`零扩展`的区别，`char`执行符号扩展，`unsigned char`执行零扩展，导致最终的整数不一样，因此需要格外注意char类型的字符转为整数的问题
```c
#include<stdio.h>

int main() {
    char c_pos = 1;
    char c_neg = -1;
    unsigned char uc_pos = 1;
    unsigned char uc_neg = -1;

    int i_pos = (int) c_pos;
    int i_neg = (int) c_neg;
    int ui_pos = (unsigned int) uc_pos;
    int ui_neg = (unsigned int) uc_neg;

    printf("----- 均为非负整数 -----\n");
    printf("有符号字符1=%d, 无符号字符1=%u\n", c_pos, uc_pos);

    printf("----- 均为负整数 -----\n");
    printf("有符号字符-1=%d, 无符号字符-1=%u\n", c_neg, uc_neg);
}
结果：
----- 均为非负整数 -----
有符号字符1=1, 无符号字符1=1
----- 均为负整数 -----
有符号字符-1=-1, 无符号字符-1=255
```

- 额外补充下`char`和`char *`
    1. 上文提过，char本质是1个字节8位的小整数，因此`char c = 'A'`或者`char c = 96`都是合法的，前者是语法糖
    2. 参考指针遍历数组的规则，定义一个字符串的过程其实就是创建字符数组的过程,因此可以按照如下方法遍历
    ```c
    #include<stdio.h>
    #include<string.h>

    int main() {
        char *string = "Hello,World!";
        const char *p = string; // 顺手保存下，好习惯 
        printf("Size of string is : %zu\n", strlen(p));

        while(*p != '\0') {
            printf("当前字符: %c,  ASCII码为 %zu\n", *p, *p);
            p++;
        }

        return 0;
    }

    Size of string is : 12
    当前字符: H,  ASCII码为 72
    当前字符: e,  ASCII码为 101
    当前字符: l,  ASCII码为 108
    当前字符: l,  ASCII码为 108
    当前字符: o,  ASCII码为 111
    当前字符: ,,  ASCII码为 44
    当前字符: W,  ASCII码为 87
    当前字符: o,  ASCII码为 111
    当前字符: r,  ASCII码为 114
    当前字符: l,  ASCII码为 108
    当前字符: d,  ASCII码为 100
    当前字符: !,  ASCII码为 33
    ```

### Integer Promotion（整数提升）

所谓整数提升，指的是小整数类型比如`short`和`char`会被自动转换为`int`或`unsigned int`参与运算。

具体规则：

| 小类型 | 提升到 |
|:---|:---|
| `char` | `int` 或 `unsigned int` |
| `short` | `int` 或 `unsigned int` |
| `bool`（C99）| `int` |
| `enum`（C99）| `int` 或 `unsigned int`（取决于枚举值范围） |

:::important[我的note]
- 如果`int`范围不足以表示原有类型，则使用`unsigned int`
- `char`和`short`直接提升成`int`非常常见
:::

示例:
```c
char c = -1;
int result = c+2;

result值为1
```

### Strict Aliasing Rule（严格别名规则）

所谓`严格别名规则`，是指编译器默认不同类型的指针**一般**不会指向同一块内存，这是出于优化编译器的目的设计的。

区别于强制类型转换规则，如果违反了严格别名规则，编译器不会报错，但是可能会得到意外的结果。
```c
float f = 3.14;
int *p = (int *) &f; //&取地址，*表示指针打过去取这个地址的内容
printf("%d \n", *p); // 这里违反了严格别名规则，会导致未定义行为
```

有两种情况算是允许的：
1. 使用`union`关键字声明的共享内存块，允许各自成员的指针去访问对应的成员
注意这里技术
```c
    union Data {
        int i;
        float f;
        char str[20];
    };

    int main() {
        union Data data;
        data.i = 10;
        data.f = 220.5;

        int *int_ptr = &data.i;
        *int_ptr = 20;

        float *float_ptr = &data.f;
        *float_ptr = 30.5f;    //注意即使是union内存操作，也不能用不同类型的指针去访问

        return 0;
    }
```

2. `char *`和`unsigned char *`也是例外，它被设计允许访问任意类型的内存，比如`memcpy`函数的实现
- 以下示例展示了使用`char *`指针按字节访问int类型数据
```c
    int num = 123456;
    char *ptr = (char *)&num;  // 把 num 的地址作为 char 类型的指针来访问
    for (int i = 0; i < sizeof(int); ++i) {
        printf("%x ", *(ptr + i));  // 按字节遍历并打印
    }
```

### 精度损失/溢出

强制类型转换或者是将错误的值赋值给了错误的变量类型可能导致最终结果被截断，或者是溢出

```c
    // 精度丢失的情况
    int a = 100000000;
    short b = a;
    printf("a: %d\n", a);//100000000
    printf("b: %d\n", b);//-7936

    // 溢出的情况
    void uint32_to_bin(uint32_t val, char *out) {
        for (int i = 31; i >= 0; i--) {
            *out++ = (val & (1U << i)) ? '1' : '0';
        }
        *out = '\0';
    }

    int main() {
        uint32_t x = -5;
        char bin[33] = {0};

        uint32_to_bin(x, bin);

        printf("x binary: %s\n", bin);//11111111111111111111111111111011
        printf("x : %u\n", x);  // 4294967291，由于按照无符号数的规则补1了，所以变成了很大的正数
        return 0;
    }
```

# 流程控制

## 预处理指令

- `#include`通常用来包括头文件：
  - `<...>`表示优先搜索系统目录
  - `"..."`表示优先搜索当前目录(可以带相对路径)，再搜索系统目录

- `#define`关键字有三个作用：
    1. 宏定义常数：`#define BUFFER_SIZE 1024`
    2. 宏定义函数：`#define MAX(a, b) ((a) > (b) ? (a) : (b))`
    3. 条件编译
   ```c
   #define DEBUG
    #ifdef DEBUG
        printf("Debug mode\n");
    #endif
   ```
   4. 宏展开防止重复包含
    ```c
    #ifndef MY_HEADER_H
    #define MY_HEADER_H
    // 内容
    #endif
    ```
    5. 安全宏函数:

        在C中有：

        ```c
        #define SAFE_FREE(p) \
        do {             \
            if (p) {     \
                free(p); \
                p = NULL;\
            }            \
        } while (0)

        char *p = malloc(100);

        SAFE_FREE(p);  // p 被释放且设为 NULL
        if (p) {
            // 这里不会进入，因为 p 是 NULL，避免了悬空指针访问
        }
        ```

        又或者在C++中可以有：

        ```c++
        #define SAFE_DELETE(ptr)     \
            do {                     \
                if (ptr) {           \
                    delete ptr;      \
                    ptr = nullptr;   \
                }                    \
            } while (0)

        #define SAFE_DELETE_ARRAY(ptr) \
            do {                       \
                if (ptr) {             \
                    delete[] ptr;      \
                    ptr = nullptr;     \
                }                      \
            } while (0)
        ```

- `#pragma`并不是C标准语法的一部分,属于**编译器指令**,不同编译器有不同的指令
  - **消除编译警告(Visual Studio特有)**：`#pragma warning(disable : 4996)`
  - **内存对齐设置**
    ```c
    #pragma pack(1)   // 1字节对齐
    struct S {
        char a;
        int b;
    };
    #pragma pack()    // 恢复默认
    ```
  - **防止重复包含**: `#pragma once` 这个和上面的宏定义是等价的  



## if指令常见的坑

|坑点分类|示例|错误原因|正确做法|备注|
|:---|:---|:---|:---|:---|
|**漏写括号**|`if (a) if (b) x=1; else x=2;`|else只匹配到最近的if|用`{}`包住每一层|**常见隐蔽错误!!**|
|**短路误解**|`if (p != NULL && *p == 0)`|忘了短路特性直接解引用空指针|确保`p != NULL`在前|逻辑与运算`&&`有短路特性,条件顺序不能乱|
|**赋值和比较混淆**|`if (a = 5)`| `=`赋值，不是`==`比较，永远为真|写成`if (a == 5)`|面试高频出|
|**优先级错误**|`if (a & b == 0)`|`==`优先级比`&`高，变成`a & (b==0)`|应写成`if ((a & b) == 0)`|位运算常踩坑|
|**宏展开误坑**|`#define IS_VALID(x)x>0`<br>`if(IS_VALID(a)&&IS_VALID(b))`|展开成`if(a>0&&b>0)`，无问题;但如果复杂就出事|加括号:`#define IS_VALID(x)((x)>0)`|宏要加括号保护|
|**逗号表达式误用**|`if(a=1,b=2,c==3)`|最后一个表达式决定if结果，不是逐个判断|拆开成多个if||
|**负数作为条件**|`if(-1)`| 在C里非零常数都是真，不是负就假|了解语义，写注释提醒||
|**数组越界读**|`if (arr[i])`|没检查`i`是否合法，直接访问|注意数组越界||
|**浮点数比较**|`if(a==0.1)`| 浮点精度误差导致意外结果|用误差范围比较`fabs(a-0.1)<epsilon`|**常见隐蔽错误!!**|
|**忘了初始化变量**|`if (flag)`| flag未初始化，值不确定|定义时赋初值|**空指针、未赋值都是雷**|
|**条件里带副作用**|`if(i++<10)`| i发生变化，注意下次使用i的值|最好不要这么写||
|**==与!=连用陷阱**|`if(x == 1\|\|2)`| 实际是`(x == 1)\|\|2`，恒为真|正确写法：`if (x == 1\|\| x == 2)`|**常见隐蔽错误!!**|

# 运算符

C语言中常见的运算符和优先级如下所示：

|优先级|运算符|描述
|:---|:---|:---
|1|`()` `[]` `->` `.`|括号，数组下标，结构体指针成员访问，结构体成员访问
|2|`!` `~` `++` `--` `+` `-` `*` `&` `(type)` `sizeof`|单目运算符(右到左)
|3|`*` `/` `%`|乘、除、取余|
|4|`+` `-`| 加、减
|5|`<<` `>>`| 左移、右移
|6|`<` `<=` `>` `>=`| 大小比较
|7|`==` `!=`| 等于、不等于
|8|`&`| 按位与
|9|`^`| 按位异或
|10|`\|`| 按位或
|11|`&&`| 逻辑与
|12|`\|\|`|逻辑或|
|13|`?:`| 条件运算(三目运算)
|14|`=` `+=` `-=` `*=` `/=` `&=` `\|=` `^=` `>>=` `<<=`|赋值及复合赋值(右到左)|
|15|`,`| 逗号运算符

:::important[注意点]
1. 优先级相同时，默认从左到右结合（除了单目运算符和赋值运算符是右到左结合）
2. 可以用括号`()`强制改变优先级
:::

## 位运算符

C语言常用的位运算符如下:

|运算符|描述|规则|用途
|:---|:---|:---|:---
|`~`|按位取反|注意`无符号类型`反转为`负数`溢出问题|
|`&`|按位与|乘法,有0则0;按位&0是屏蔽，&1是不变|清零;取指定位;&1判断最后一位奇偶
|`\|`|按位或|有1则1|开关,让第`x`位打开就是和`2的(x-1)次方`按位或(关闭还是按位与0)
|`^`|按位异或|不同为1|`^0`还是自己;批量翻转比如后四位(^15);交换两个变量
|`<<`|左移|左移`x`位，右侧补`0`|**左移N位的本质是乘以2的N次方**
|`>>`|右移|右移`x`位|**右移N位的本质是除以2的N次方**

:::note[异或操作交换两个变量]
```c
int swap(int *a,int *b)
{
    if (*a!=*b)
    {
        *a=*a^*b;
        *b=*b^*a;
        *a=*a^*b;
    }
    return 0;
}
```
:::

## 逻辑运算符

C语言常用逻辑表达式运算符如下:

|运算符|描述|规则|用途
|:---|:---|:---|:---
|`&&`|逻辑与|同时为真则为真|**短路原则**:左表达式为假则直接判定为假
|`\|\|`|逻辑或|有一个条件为真则为真|

### 逻辑表达式的化简