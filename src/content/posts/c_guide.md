---
title: C语言教程-常量和变量
published: 2025-04-28
description: '做项目时候作为字典的笔记'
image: ''
tags: [Cookbook,C]
category: 'Cookbook'
draft: false 
lang: ''
---

# 基础语法

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
