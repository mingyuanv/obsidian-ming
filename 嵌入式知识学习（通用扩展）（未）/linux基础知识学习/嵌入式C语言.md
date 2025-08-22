---
title: 镜像烧录 | 油炸鸡开源硬件
aliases: 
tags: 
description:
---
# 备注(声明)：

- 3 要做到： when to do? how to do? why to do?

- 1 双引号一定要把它理解成`const char `
# 一、编译和预处理

## 二进制
### 1 、进制转换
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628893.png|Open: Pasted image 20250706135257.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628893.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628893.png)
### 2 、2的n次方（二进制）

| 2的n次方 |       |     |     |
| ----- | ----- | --- | --- |
| 1     | 2     |     |     |
| 2     | 4     |     |     |
| 3     | 8     |     |     |
| 4     | 16    |     |     |
| 5     | 32    |     |     |
| 6     | 64    |     |     |
| 7     | 128   |     |     |
| 8     | 256   |     |     |
| 9     | 512   |     |     |
| 10    | 1024  |     |     |
| 16    | 65535 | 2B  | int |
|       |       |     |     |

### 3 、进制表示
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628981.png|Open: Pasted image 20250706152603.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628981.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628981.png)

#### ​进制表示法总结​​

| ​**​进制​**​   | ​**​标识符（前缀）​**​ | ​**​后缀（可选）​**​         | ​**​示例​**​            | ​**​注意事项​**​                                             |
| ------------ | --------------- | ---------------------- | --------------------- | -------------------------------------------------------- |
| ​**​十进制​**​  | 无               | `L`（长整型）  <br>`U`（无符号） | `123`、`456U`、`789L`   | 直接书写数字，不可添加前缀。`U`和 `L`可组合（如 `123UL`）<br><br>。            |
| ​**​八进制​**​  | `0`             | 同十进制                   | `0123`（十进制 83）        | 必须​**​以 `0`开头​**​，后跟 0-7 的数字（如 `077`合法，`089`非法）<br><br>。 |
| ​**​十六进制​**​ | `0x`或 `0X`      | 同十进制                   | `0x1A`（十进制 26）、`0XFF` | 后跟 0-9 及 A-F/a-f（不区分大小写）<br><br>。                        |
| ​**​二进制​**​  | `0b`或 `0B`      | 同十进制                   | `0b1010`（十进制 10）      | ​**​需 C99 或更高标准支持​**​（如 GCC、Clang 支持，旧版编译器可能不支持）         |

```c
    int a=0b10;
    int b=10;
    int c=010;
    int d=0x10;
    printf("a=%d, b=%d, c=%d, d=%d\n", a, b, c, d);
```
> (base) topeet@ubuntu:~/test/c$ ./app
`a=2, b=10, c=8, d=16`

### 后缀 `L`（或 `l`）和 `U`（或 `u`）
- 1 显式指定整型常量的数据类型，主要解决数值范围、符号性和跨平台兼容性问题

> `long int`通常占用 ​**​4字节​**​（32位系统）或 ​**​8字节​**​（64位系统）

#### ​后缀总结表​​

| ​**​后缀​**​ | ​**​类型​**​      | ​**​典型场景​**​ | ​**​示例​**​       |
| ---------- | --------------- | ------------ | ---------------- |
| `L`        | `long int`      | 大整数（>2.1亿）   | `2147483648L`    |
| `U`        | `unsigned int`  | 位操作、非负索引     | `0xFFFFU`        |
| `UL`       | `unsigned long` | 极大无符号数（>40亿） | `4294967296UL`   |
| `LL`       | `long long`     | 超大数据（需C99支持） | `123456789012LL` |

### 二进制中的正负数表示
- 1 二进制数的<span style="background:#b1ffff">最高位作为符号位</span>，**0 表示正数，1 表示负数**。

#### 补码
> 正数的补码与原码相同；**负数的补码是反码加 1。**
> **负数的反码是原码的符号位不变，其余位取反。**
> 
> **加法和减法可统一为补码加法**（无需单独处理符号位）。

```c
    int a=-5;

    print_binary(a); 
```
> (base) topeet@ubuntu:~/test/c$ ./app
`11111111111111111111111111111011`



#### 补码的运算规则

> - **加法**：直接按二进制加法计算，**忽略溢出位**。
>     - 示例：`1 + (-2)`
>         - 补码：`0000 0001` + `1111 1110` = `1111 1111`（即 `-1`）。
> - **减法**：转换为加法（被减数 + 负减数的补码）。
>     - 示例：`5 - 3`
>         - 补码：`0000 0101` + `1111 1101` = `0000 0010`（即 `2`）。
> 



## 对c语言的本质了解
### 1 、C语言工具的特性:内存操作
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629106.png|Open: Pasted image 20250706135850.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629106.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629106.png)

### 2 、对c语言的想法
> 什么时候用?
> 怎么用?
> 为什么要这样设计?

- 1 人和人之间交流的一种什么高级语言

- 3 在学完c语言过后，把这些关键字和常用符号啊，都能够呃看到过后立刻想到诶，我在什么场景下用过这样的符号，而且呢确实很好地解决了这样的问题。

### 3 、推荐教材
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629166.png|Open: Pasted image 20250706135758.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629166.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629166.png)

### 4、c操作对象 - 资源/内存(内存类型的资源,LCD缓存,LED灯)



## gcc概述
### 1 、翻译官
- 1 高级语言向机器语言翻译的一个工具

### 2 、查看gcc版本（ gcc -v）
[“02 gcc概述”页上的图片](onenote:#02%20gcc概述&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={F20956CC-824F-47BF-B415-2AC1FC61B17E}&object-id={1A24562D-03F8-423C-9B59-162B1B98FBD7}&65&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 3 、gcc -o 输出 输入 （gcc编译）
[“02 gcc概述”页上的图片](onenote:#02%20gcc概述&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={F20956CC-824F-47BF-B415-2AC1FC61B17E}&object-id={1A24562D-03F8-423C-9B59-162B1B98FBD7}&6A&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

- 1 -o后面跟输出就可。

### 4 、gcc编译时增加-v选项
[“02 gcc概述”页上的图片](onenote:#02%20gcc概述&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={F20956CC-824F-47BF-B415-2AC1FC61B17E}&object-id={1A24562D-03F8-423C-9B59-162B1B98FBD7}&B7&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

- 2 为我们服务，那么这个时候它就会打开一个开关，把我们gcc中设计的所有组织，都进行一个展现啊。
> (base) topeet@ubuntu:~/test/c$` gcc main.c -o app -v`
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/9/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none:hsa
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 9.4.0-1ubuntu1~20.04.1' --with-bugurl=file:///usr/share/doc/gcc-9/README.Bugs --enable-languages=c,ada,c++,go,brig,d,fortran,objc,obj-c++,gm2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-9 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-vtable-verify --enable-plugin --enable-default-pie --with-system-zlib --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-werror --wit




### 5、return 0；（代表成功）
- 2 非零值都代表失败




## C语言编译过程介绍
- 3 后面的命令在运行的时候会自动运行完前面要用的命令
### 1 、预处理（gcc -E）（生成.i文件）
- 3 预处理：去掉注释、加载头文件、替换宏定义，注意预处理不会进行语法检查

- 2 gcc -E -o app.i app.c

### 2 、编译（gcc -S）（生成.s文件）
- 2 gcc -S -o app.s app.c

### 3 、汇编（gcc -c）（生成.o文件）
- 2 gcc -c -o app.o app.c

### 4 、链接（gcc -o）（生成可执行文件）
- 2 gcc -o app app.c

> (base) topeet@ubuntu:~/test/c$ `gcc -E -o main.i main.c`
(base) topeet@ubuntu:~/test/c$ `gcc -S -o main.s main.c`
(base) topeet@ubuntu:~/test/c$ `gcc -c -o main.o main.c`
(base) topeet@ubuntu:~/test/c$ `gcc main.c -o app`




## C语言编译常见错误举例
### 1 、预处理错误（not find）（-I）
```
#include <stdio.h> 
#include "stdio.h"
```
- 3 尖括号都是系统库
- 2 直接在我们系统的环境变量中去寻找

- 3 双引号都是我们自定义的头文件
- 2 在我们的当前目录下开始去寻找名字


- 1 `-I` 选项用于指定头文件（header files）的搜索路径



### 2 、编译错误（语法问题）
- 3 有多个原材料的时候，先把每个原材料编译（汇编）过后(.o)，再进行链接。


### 3 、链接错误（原材料多或少了）
- 3  undefined reference to ‘fun'（原材料不够）
- 2 寻找标签是否实现了，链接时是否加入一起链接

- 3 multiple definition of fun （原材料多了）
- 2 多次实现了标签，只保留一个标签实现

### 4 、




## C语言预处理介绍
### 1 、 include - 在当前的位置上进行一个展开

### 2 、#define 宏名 宏体
- 1 宏名一般都是用大写字母来写，尽量加括号。

[“05 C语言预处理介绍”页上的图片](onenote:#05%20C语言预处理介绍&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={01B164D4-DF74-4C42-A279-6DD0D76F0013}&object-id={D93DADC8-79BE-4405-9113-C27067B25944}&53&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 3 、宏函数
`#define ABC(x) (5+(x))`

### 4 、预定义宏（gcc自带的）
```
__FUNCTION__      函数名

__LINE__           行号

__FINE__           文件名

```
[举例使用](onenote:#05%20C语言预处理介绍&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={01B164D4-DF74-4C42-A279-6DD0D76F0013}&object-id={D93DADC8-79BE-4405-9113-C27067B25944}&6F&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 5、



## 条件预处理的应用
### 1 、条件预处理的作用
- 1 实现两个版本（调试和发行）之间的切换，借助我们编译器之前的预处理器。

### 2 、实际举例运用
```
#ifdef ABC
	printf("123%s",__FINE__);
	
#else AAA
	printf("222");
	
#endif 
```

### 3 、gcc -DXXX（在命令行定义宏）
[“06 条件预处理的应用”页上的图片](onenote:#06%20条件预处理的应用&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={C360F865-5BA4-458C-ADEB-E90F6940E804}&object-id={33803EB0-0A81-4AC2-9AAF-06FE2046BA6F}&62&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

```c
    #ifdef ABCD
        int a=0b10UL;
        int b=10;
        int c=010;
        int d=0x10;
        printf("a=%d, b=%d, c=%d, d=%d\n", a, b, c, d);

    #else
        printf("ABCD is not defined\n");


    #endif
```
> (base) topeet@ubuntu:~/test/c$ `gcc main.c -o app -DABCD`
(base) topeet@ubuntu:~/test/c$ ./app
a=2, b=10, c=8, d=16






### 4 、


## 宏展开下的#、##使用
### 1 、# 字符串化
[“07 宏展开下的、使用”页上的图片](onenote:#07%20宏展开下的、使用&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={482F50DE-0E6C-4C4A-B84B-268F665D91E6}&object-id={0B54FA0C-77BE-4A78-B0D4-B6BB18B2275E}&30&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 2 、## 连接符号
[“07 宏展开下的、使用”页上的图片](onenote:#07%20宏展开下的、使用&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={482F50DE-0E6C-4C4A-B84B-268F665D91E6}&object-id={0B54FA0C-77BE-4A78-B0D4-B6BB18B2275E}&3D&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

- 2 前缀啊或后缀
- 2 隐藏的方法来实现
- 2 一些函数的父子啊
- 2 函数的调用啊的一些技巧性的赋值

- 1 实际举例：    [“07 宏展开下的、使用”页上的图片](onenote:#07%20宏展开下的、使用&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={482F50DE-0E6C-4C4A-B84B-268F665D91E6}&object-id={0B54FA0C-77BE-4A78-B0D4-B6BB18B2275E}&5F&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 3 、

## ASCII 码表
### 1、 📚 ASCII 码的分类

| 范围       | 含义            | 示例字符                          |
| -------- | ------------- | ----------------------------- |
| 0～31、127 | 控制字符          | `\n`（换行）、`\t`（制表）、`\r`（回车）    |
| 32～126   | 可打印字符         | `'A'`（65）、`'a'`（97）、`'0'`（48） |
| 128～255  | 扩展 ASCII（非标准） | 一些符号、特殊字符（取决于系统）              |

### 2、`char` 类型本质上是一个整数类型，存储的是字符的 ASCII 值(十进制)。
- 1 可以通过强制类型转换在字符和 ASCII 值之间互相转换。
```c
    char ch = 'A';
    int ascii = (int)ch;
    printf("字符 '%c' 的 ASCII 值是 %d\n", ch, ascii); // 输出：字符 A 的 ASCII 值是 65
    return 0;
```
> (base) topeet@ubuntu:~/test/c$ ./app
`字符 'A' 的 ASCII 值是 65`



### 3、ASCII 码表（标准 0～127）

| 十进制    | 十六进制      | 字符    | 名称/说明                            | 用途                |
| ------ | --------- | ----- | -------------------------------- | ----------------- |
| 0      | 0x00      | NUL   | Null（空字符）                        | 用于字符串结束标志         |
| 1      | 0x01      | SOH   | Start of Header（标题开始）            | 控制字符              |
| 2      | 0x02      | STX   | Start of Text（正文开始）              | 控制字符              |
| 3      | 0x03      | ETX   | End of Text（正文结束）                | 控制字符              |
| 4      | 0x04      | EOT   | End of Transmission（传输结束）        | 控制字符              |
| 5      | 0x05      | ENQ   | Enquiry（请求）                      | 控制字符              |
| 6      | 0x06      | ACK   | Acknowledge（确认）                  | 控制字符              |
| 7      | 0x07      | BEL   | Bell（响铃）                         | 蜂鸣提示              |
| 8      | 0x08      | BS    | Backspace（退格）                    | 退格键               |
| 9      | 0x09      | HT    | Horizontal Tab（水平制表符）            | 制表键               |
| 10     | 0x0A      | LF    | Line Feed（换行）                    | 换行符 `\n`          |
| 11     | 0x0B      | VT    | Vertical Tab（垂直制表符）              | 控制字符              |
| 12     | 0x0C      | FF    | Form Feed（换页）                    | 换页符               |
| 13     | 0x0D      | CR    | Carriage Return（回车）              | 回车符 `\r`          |
| 14     | 0x0E      | SO    | Shift Out（移出）                    | 控制字符              |
| 15     | 0x0F      | SI    | Shift In（移入）                     | 控制字符              |
| 16     | 0x10      | DLE   | Data Link Escape（数据链路转义）         | 控制字符              |
| 17     | 0x11      | DC1   | Device Control 1（设备控制1）          | 控制字符              |
| 18     | 0x12      | DC2   | Device Control 2（设备控制2）          | 控制字符              |
| 19     | 0x13      | DC3   | Device Control 3（设备控制3）          | 控制字符              |
| 20     | 0x14      | DC4   | Device Control 4（设备控制4）          | 控制字符              |
| 21     | 0x15      | NAK   | Negative Acknowledge（否定确认）       | 控制字符              |
| 22     | 0x16      | SYN   | Synchronous Idle（同步空闲）           | 控制字符              |
| 23     | 0x17      | ETB   | End of Transmission Block（传输块结束） | 控制字符              |
| 24     | 0x18      | CAN   | Cancel（取消）                       | 控制字符              |
| 25     | 0x19      | EM    | End of Medium（介质结束）              | 控制字符              |
| 26     | 0x1A      | SUB   | Substitute（替换）                   | 控制字符              |
| 27     | 0x1B      | ESC   | Escape（换码符）                      | 特殊控制（如 ANSI 转义序列） |
| 28     | 0x1C      | FS    | File Separator（文件分隔符）            | 控制字符              |
| 29     | 0x1D      | GS    | Group Separator（组分隔符）            | 控制字符              |
| 30     | 0x1E      | RS    | Record Separator（记录分隔符）          | 控制字符              |
| 31     | 0x1F      | US    | Unit Separator（单元分隔符）            | 控制字符              |
| 32     | 0x20      | (空格)  | Space（空格）                        | 可打印字符             |
| 33～126 | 0x21～0x7E | 可打印字符 | -                                | 包括字母、数字、标点符号等     |
| 127    | 0x7F      | DEL   | Delete（删除）                       | 控制字符              |


### 4、可打印 ASCII 字符表（32～126）
| 十进制 | 字符      | 十进制 | 字符  | 十进制     | 字符  | 十进制 | 字符  |
| --- | ------- | --- | --- | ------- | --- | --- | --- |
| 32  | 空格      | 33  | `!` | 34      | `"` | 35  | `#` |
| 36  | `$`     | 37  | `%` | 38      | `&` | 39  | `'` |
| 40  | `(`     | 41  | `)` | 42      | `*` | 43  | `+` |
| 44  | `,`     | 45  | `-` | 46      | `.` | 47  | `/` |
| 48  | `0`     | 49  | `1` | 50      | `2` | 51  | `3` |
| 52  | `4`     | 53  | `5` | 54      | `6` | 55  | `7` |
| 56  | `8`     | 57  | `9` | 58      | `:` | 59  | `;` |
| 60  | `<`     | 61  | `=` | 62      | `>` | 63  | `?` |
| 64  | `@`     | 65  | `A` | 66      | `B` | 67  | `C` |
| 68  | `D`     | 69  | `E` | 70      | `F` | 71  | `G` |
| 72  | `H`     | 73  | `I` | 74      | `J` | 75  | `K` |
| 76  | `L`     | 77  | `M` | 78      | `N` | 79  | `O` |
| 80  | `P`     | 81  | `Q` | 82      | `R` | 83  | `S` |
| 84  | `T`     | 85  | `U` | 86      | `V` | 87  | `W` |
| 88  | `X`     | 89  | `Y` | 90      | `Z` | 91  | `[` |
| 92  | `\`     | 93  | `]` | 94      | `^` | 95  | `_` |
| 96  | `` ` `` | 97  | `a` | 98      | `b` | 99  | `c` |
| 100 | `d`     | 101 | `e` | 102     | `f` | 103 | `g` |
| 104 | `h`     | 105 | `i` | 106     | `j` | 107 | `k` |
| 108 | `l`     | 109 | `m` | 110     | `n` | 111 | `o` |
| 112 | `p`     | 113 | `q` | 114     | `r` | 115 | `s` |
| 116 | `t`     | 117 | `u` | 118     | `v` | 119 | `w` |
| 120 | `x`     | 121 | `y` | 122     | `z` | 123 | `{` |
| 124 |  \|     | 125 | `}` | 126<br> | `~` |     |     |



### 5、





# 二、关键字 - 编译器预先定义了一定意义的字符串（32个）
- 3 在任何环境下都可以使用

## 数据类型（对资源大小进行限制）
### 1 、总揽
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629301.png|Open: Pasted image 20250706150719.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629301.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629301.png)

### 2 、数据类型 - 限制内存(土地)的大小
- 1 c语言用来描述资源的属性。

- 2 数据类型的大小是跟编译器有关的，但是有默认值
### 3 、char（1B=8bit）（软件操作的最小单位）
[“10 数据类型关键字介绍及char类型”页上的图片](onenote:#10%20数据类型关键字介绍及char类型&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={60456CD9-C9F4-46B2-87C6-F6845E9C6A38}&object-id={7F01788B-4E35-4BA0-9AAC-D0758DA14594}&3D&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 4 、int（编译器最优的处理大小）（64位、32位系统中都为4B）
- 2 2B=65535


### 5、short（大多系统上为2B）
[“11 数据类型之int、long、short”页上的图片](onenote:#11%20数据类型之int、long、short&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={92804773-9174-4437-A155-A88C10EC8182}&object-id={CA5F1483-B31F-41A1-B50F-65CD963659DB}&36&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 6、long（大多系统上为8B）
[“11 数据类型之int、long、short”页上的图片](onenote:#11%20数据类型之int、long、short&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={92804773-9174-4437-A155-A88C10EC8182}&object-id={CA5F1483-B31F-41A1-B50F-65CD963659DB}&36&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 7、无符号数 - unsigned（数据）
[“12 数据类型之符号数(unsigned ,signed)、浮点类型(float ,double)、void”页上的图片](onenote:#12%20数据类型之符号数\(unsigned%20,signed\)、浮点类型\(float%20,double\)、void&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={512AEE79-1675-4587-BD34-A4429F249875}&object-id={58A89822-FC44-4F7F-BB3D-8A8AE083EF68}&38&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 8、有符号数 - signed（数字）（默认）
- 3 内存空间的最高字节是符号位
- 2 有符号数的最高位是符号位是无法右移成零
[“12 数据类型之符号数(unsigned ,signed)、浮点类型(float ,double)、void”页上的图片](onenote:#12%20数据类型之符号数\(unsigned%20,signed\)、浮点类型\(float%20,double\)、void&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={512AEE79-1675-4587-BD34-A4429F249875}&object-id={58A89822-FC44-4F7F-BB3D-8A8AE083EF68}&38&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 9、浮点类型 - float（4B）（耗内存）
[“12 数据类型之符号数(unsigned ,signed)、浮点类型(float ,double)、void”页上的图片](onenote:#12%20数据类型之符号数\(unsigned%20,signed\)、浮点类型\(float%20,double\)、void&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={512AEE79-1675-4587-BD34-A4429F249875}&object-id={58A89822-FC44-4F7F-BB3D-8A8AE083EF68}&43&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 10、浮点类型 - double（8B）（耗内存）
[“12 数据类型之符号数(unsigned ,signed)、浮点类型(float ,double)、void”页上的图片](onenote:#12%20数据类型之符号数\(unsigned%20,signed\)、浮点类型\(float%20,double\)、void&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={512AEE79-1675-4587-BD34-A4429F249875}&object-id={58A89822-FC44-4F7F-BB3D-8A8AE083EF68}&43&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)


### 11、void（语义符）（并未申请内存）
[“12 数据类型之符号数(unsigned ,signed)、浮点类型(float ,double)、void”页上的图片](onenote:#12%20数据类型之符号数\(unsigned%20,signed\)、浮点类型\(float%20,double\)、void&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={512AEE79-1675-4587-BD34-A4429F249875}&object-id={58A89822-FC44-4F7F-BB3D-8A8AE083EF68}&4E&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 12、实验（查看数据类型的大小）
```c
    int a=5;
    unsigned int a_unsigned=5U;

    short int a_short=5;
    long int a_long=5L;
    long long int a_long_long=5LL;
    


    char b='A';
    float c=3.14f; //f是​​单精度浮点类型标识符​​
    double d=3.141592653589793;


    printf("%zu\n", sizeof(a));           // 正确：输出 int 的大小
    printf("%zu\n", sizeof(a_unsigned));  // 正确：输出 unsigned int 的大小
    printf("%zu\n", sizeof(a_short));     // 正确：输出 short 的大小
    printf("%zu\n", sizeof(a_long));      // 正确：输出 long 的大小
    printf("%zu\n", sizeof(a_long_long)); // 正确：输出 long long 的大小
    printf("%zu\n", sizeof(b));           // 正确：输出 float 的大小
    printf("%zu\n", sizeof(c));           // 正确：输出 double 的大小
    printf("%zu\n", sizeof(d));           // 正确：输出 long double 的大小
```
> (base) topeet@ubuntu:~/test/c$ ./app
> <span style="background:#d3f8b6">4</span>
> <span style="background:#d3f8b6">4</span>
> <span style="background:#d3f8b6">2</span>
> <span style="background:#d3f8b6">8</span>
> <span style="background:#d3f8b6">8</span>
> <span style="background:#d3f8b6">1</span>
> <span style="background:#d3f8b6">4</span>
> <span style="background:#d3f8b6">8</span>
> 




## 自定义数据类型（基本元素的集合）
### 1 、总揽
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629418.png|Open: Pasted image 20250706150752.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629418.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629418.png)

### 2 、结构体 - struct
[“13 自定义数据类型struct、union”页上的图片](onenote:#13%20自定义数据类型struct、union&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={7B6E5E10-84A3-42FE-9A7B-F5CC9B56592C}&object-id={7C3D0F36-EA3C-4DC8-9C44-A300DD6BE64B}&18&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

```c
    struct a{
        int a;
        char b;
        float c;
        double d;
    };

    struct a s1;
    s1.a=1;

    printf("s1.a=%d\n", s1.a);
    printf("sizeof(s1)=%zu\n", sizeof(s1)); // 输出结构体的大小
```
> (base) topeet@ubuntu:~/test/c$ ./app
s1.a=1
`sizeof(s1)=24`

- 2 每一个变量的起始地址，都是上一个变量的什么未地址啊


### 3 、union - 共用起始地址的一段内存（定义同结构体）
[“13 自定义数据类型struct、union”页上的图片](onenote:#13%20自定义数据类型struct、union&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={7B6E5E10-84A3-42FE-9A7B-F5CC9B56592C}&object-id={7C3D0F36-EA3C-4DC8-9C44-A300DD6BE64B}&7D&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
```c
    union a{
        int a;
        char b;
        float c;
        double d;
    };

    union a s1;
    s1.a=1;

    printf("s1.a=%d\n", s1.a);
    printf("sizeof(s1)=%zu\n", sizeof(s1)); // 输出union的大小
```
> (base) topeet@ubuntu:~/test/c$ ./app
s1.a=1
`sizeof(s1)=8`
- 2 在内核中，一些技巧性的代码啊，能够理解到就可以啊


### 4 、enum（一一列举）（被命名的整型常数(int)的集合）
[“14 自定义数据类型enum”页上的图片](onenote:#14%20自定义数据类型enum&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={FFAB0191-DA96-48D3-816A-4F61E263365F}&object-id={A3E883FE-0E76-436C-877A-15DAA4E8C17C}&29&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
```c
    enum color {
        RED=100,
        GREEN,
        BLUE
    };

    enum color myColor = GREEN;

    printf("myColor=%d\n", myColor); // 输出枚举值对应的整数值
```
> (base) topeet@ubuntu:~/test/c$ ./app
`myColor=101`

> - 枚举值是常量，不能通过赋值语句修改（如 `a = 5` 是错误的）。
> - 枚举常量本质上是整数，但它们的语义更明确（例如 `a` 代表某个特定的含义）。
> - 如果未显式指定值，第一个枚举常量默认为 `0`，后续值依次递增。


- 1 提高代码可读性
```c
// 使用枚举
a1 = a;  // 明确表示赋值为 "a"
if (a1 == b) {
    // 处理逻辑
}

// 不使用枚举
int a1 = 1;  // 魔法数字 1 的含义不明确
if (a1 == 2) {
    // 处理逻辑
}
```


- 1 实际举例： [“14 自定义数据类型enum”页上的图片](onenote:#14%20自定义数据类型enum&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={FFAB0191-DA96-48D3-816A-4F61E263365F}&object-id={A3E883FE-0E76-436C-877A-15DAA4E8C17C}&B9&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
### 5、typedef（重命名）
[“15 自定义数据类型typedef”页上的图片](onenote:#15%20自定义数据类型typedef&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={645B1E1D-3052-4780-ACAB-7F0AF0CCD2AB}&object-id={7F91BB18-C052-4B4C-9BA5-5868E37A33DF}&12&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

`typedef unsigned int u_int;`

### 6、


## 逻辑结构
### 1 、总揽
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629508.png|Open: Pasted image 20250706150757.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629508.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629508.png)

### 2 、if、else（条件分支）

```c
    int a=3;

    if(a>3){
        printf("a is greater than 3\n");
    }else if(a==3){
        printf("a is equal to 3\n");

    }else{
        printf("a is less than 3\n");
    }
```
[“16 逻辑结构关键字”页上的图片](onenote:#16%20逻辑结构关键字&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={2F69F800-4CF8-4A93-A513-99ABFECE3964}&object-id={18D469AA-F2BB-46E4-A181-46464616186C}&29&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 3 、switch、case、default（多分支）
```c
    int a=4;

    switch(a){
        case 1:
            printf("a is 1\n");
            break;
        case 2:
            printf("a is 2\n");
            break;
        case 3:
            printf("a is 3\n");
            break;
        default:
            printf("a is greater than 3\n");
            break;
       
    }

```

[“16 逻辑结构关键字”页上的图片](onenote:#16%20逻辑结构关键字&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={2F69F800-4CF8-4A93-A513-99ABFECE3964}&object-id={18D469AA-F2BB-46E4-A181-46464616186C}&3B&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 4 、do、while（循环）（先执行一次循环体内的代码）
```c
    int a=2;

    do{
        a++;
        printf("a=%d\n",a);
        if(a==3){
            continue; // 当a等于3时，跳出循环
        }
        printf("Continuing the loop\n");
        if(a==8){
            break; // 当a等于8时，退出循环

        }

    }while(a<10);
```
> (base) topeet@ubuntu:~/test/c$ ./app
`a=3`
a=4
Continuing the loop
a=5
Continuing the loop
a=6
Continuing the loop
a=7
Continuing the loop
`a=8
`Continuing the loop`

[“16 逻辑结构关键字”页上的图片](onenote:#16%20逻辑结构关键字&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={2F69F800-4CF8-4A93-A513-99ABFECE3964}&object-id={18D469AA-F2BB-46E4-A181-46464616186C}&59&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 5、for（循环）（通常用于已知循环次数的情况。）
 ```c
    for(int a=1;a<10;a++){
        printf("a=%d\n", a);
    }
```
> (base) topeet@ubuntu:~/test/c$ ./app
a=1
a=2
a=3
a=4
a=5
a=6
a=7
a=8
`a=9`

[“16 逻辑结构关键字”页上的图片](onenote:#16%20逻辑结构关键字&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={2F69F800-4CF8-4A93-A513-99ABFECE3964}&object-id={18D469AA-F2BB-46E4-A181-46464616186C}&59&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 6、continue（跳过当前循环体中剩余的部分并继续下一次循环）
[“16 逻辑结构关键字”页上的图片](onenote:#16%20逻辑结构关键字&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={2F69F800-4CF8-4A93-A513-99ABFECE3964}&object-id={18D469AA-F2BB-46E4-A181-46464616186C}&64&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 7、break（立即退出最近的循环或 switch 语句）
- 2 程序将继续从该结构之后的第一条语句开始执行

[“16 逻辑结构关键字”页上的图片](onenote:#16%20逻辑结构关键字&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={2F69F800-4CF8-4A93-A513-99ABFECE3964}&object-id={18D469AA-F2BB-46E4-A181-46464616186C}&6F&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
### 8、goto（使程序跳转到标记了相应标签的位置,并终止循环）
[“16 逻辑结构关键字”页上的图片](onenote:#16%20逻辑结构关键字&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={2F69F800-4CF8-4A93-A513-99ABFECE3964}&object-id={18D469AA-F2BB-46E4-A181-46464616186C}&77&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

```c
    int a=2;

    do{
        a++;
        printf("a=%d\n",a);
        if(a==3){
            continue; // 当a等于3时，跳出循环
        }
        printf("Continuing the loop\n");
        if(a==5){
            goto label; // 当a等于5时，跳转到标签
        }
        if(a==8){
            break; // 当a等于8时，退出循环

        }

    }while(a<10);

label:
    printf("Exited the loop at a=%d\n", a);
```
> (base) topeet@ubuntu:~/test/c$ ./app
a=3
a=4
Continuing the loop
a=5
Continuing the loop
`Exited the loop at a=5`

- 2 一般都在同一个函数内使用,不进行跨函数使用

## 类型修饰符 - 对内存资源存放位置的限定
### 1 、总揽
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629745.png|Open: Pasted image 20250706150804.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629745.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171629745.png)

### 2 、register（尽量限制变量定义在寄存器上）
[“17 类型修饰符(一)_register ,auto”页上的图片](onenote:#17%20类型修饰符\(一\)_register%20,auto&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={9B92E7A0-88A3-4335-8026-DFE85AA86FBE}&object-id={7F558D21-0876-4F56-835D-D062E0A42655}&6C&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

- 2 CPU直接访问寄存器是比访问内存要快的效率更高的
- 2 适用于频繁使用的局部变量，如循环计数器
- 2 给我们人和人之间交流的时候，带来一种思想，原来a访问比较频繁。

- 3 错误使用会在编译时报错
```c
    register int a=10;

    printf("a=%d",&a);
```
> (base) topeet@ubuntu:~/test/c$ gcc main.c -o app
main.c: In function ‘main’:
main.c:16:5: error: `address of register variable ‘a’ requested`
   16 |     printf("a=%d",&a);
      |     ^~~~~~

### 3 、auto（默认）
- 3 普通内存可读可写空间中分配的一段区域（栈空间）
[“17 类型修饰符(一)_register ,auto”页上的图片](onenote:#17%20类型修饰符\(一\)_register%20,auto&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={9B92E7A0-88A3-4335-8026-DFE85AA86FBE}&object-id={7F558D21-0876-4F56-835D-D062E0A42655}&48&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 4 、static（静态）
[“18 类型修饰符(二)_static_const”页上的图片](onenote:#18%20类型修饰符\(二\)_static_const&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={CF81B6CA-5F59-4891-AF7E-5AEE8AFCFCEB}&object-id={7D6A1ACE-E9E8-4001-94F8-E5573BCA796D}&16&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

#### 全局变量 （限制了该变量的作用域仅限于声明它的源文件内）       
- 1 其他文件即使通过 `extern` 声明也无法访问它。
```c
// test.c 
static int a = 10;   // 只能在 test.c 内部访问 

//test.h
extern int a;

// other.c 
extern int a; // 错误！无法访问 test.c 中的 static 变量
```


#### 局部变量（函数结束，静态局部变量的值仍然保留）         
- 1 还是局部变量，在函数外部还是无法调用。
```c
void test_function() {
    static int static_a=1;
    printf("static_a=%d\n", static_a);
    static_a++;
}

    test_function(); // 调用测试函数
    test_function(); // 再次调用测试函数，观察static变量的变化
```
> (base) topeet@ubuntu:~/test/c$ ./app
`static_a=1`
`static_a=2`

#### 函数（限制了函数只能在定义它的文件内部被调用）                  
- 1 其他文件<span style="background:#d3f8b6">即使包含头文件也无法访问它</span>。
```c
// test.c 
static void test(){
    printf("tesr.c\n");
}   // 只能在 test.c 内部访问 

//test.h
static void test();

// other.c 
test(); // 错误！无法访问 test.c 中的 static函数
```

#### 跨文件共享变量的正确用法
```c
// test.c
int a = 10;  // 定义全局变量

// test.h
extern int a;  // 声明全局变量

// other.c
#include "test.h"
extern int a；
printf("%d\n", a);  // 可以访问 a
```

### 5、const（常量）（只读）
[“18 类型修饰符(二)_static_const”页上的图片](onenote:#18%20类型修饰符\(二\)_static_const&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={CF81B6CA-5F59-4891-AF7E-5AEE8AFCFCEB}&object-id={7D6A1ACE-E9E8-4001-94F8-E5573BCA796D}&FC&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

#### const与指针定义
[“18 类型修饰符(二)_static_const”页上的图片](onenote:#18%20类型修饰符\(二\)_static_const&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={CF81B6CA-5F59-4891-AF7E-5AEE8AFCFCEB}&object-id={7D6A1ACE-E9E8-4001-94F8-E5573BCA796D}&FE&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

- 1 指针本身不可修改     常量指针，初始化后不能更改其指向的地址
```c
    char * const p="Hello, World!"; // 指向字符串常量的指针

    p=(char *)0x1293; // 错误：不能修改指向字符串常量的指针
```
> (base) topeet@ubuntu:~/test/c$ gcc main.c -o app
main.c: In function ‘main’:
main.c:36:6: error: assignment of read-only variable ‘p’
   36 |     p=(char * )0x1293; // 错误：`不能修改常量指针`
      |      ^
- 2 如果指向的数据不是常量，则可以通过* q修改数据


- 1 指针指向的内容不可修改
```c
    const char *p="hello"; // 指向字符串常量的指针

    *p="he"; // 错误：不能修改常量的内容
```
> (base) topeet@ubuntu:~/test/c$ gcc main.c -o app
main.c: In function ‘main’:
main.c:37:7: error: assignment of read-only location ‘* p’
   37 |    ` * p="he"; // 错误：不能修改常量的内容`
      |       ^

- 1 // 两者都不可修改
```c
const char * const p="hello"; // 指向字符串常量的常量指针
```

#### const与函数（保证传入的数据不会被函数内部修改）（返回只读变量）
[“18 类型修饰符(二)_static_const”页上的图片](onenote:#18%20类型修饰符\(二\)_static_const&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={CF81B6CA-5F59-4891-AF7E-5AEE8AFCFCEB}&object-id={19FD547C-EB91-49E9-88E1-27D54ABE475A}&A&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

```c
void test_function(const int a) {

    printf("static_a=%d\n", a);
    a++;

}

    const int a=11;
    test_function(a);
```
> (base) topeet@ubuntu:~/test/c$ gcc main.c -o app
main.c: In function ‘test_function’:
main.c:9:6: `error: increment of read-only parameter ‘a’`
    9 |     a++;
      |      ^~


### 6、volatile（不优化编译）
[“19 类型修饰符(三)_volatile”页上的图片](onenote:#19%20类型修饰符\(三\)_volatile&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={8FD51D8E-B429-4698-9DCD-E2E24C5D4312}&object-id={A2081200-C60E-497C-BE50-930497E40174}&16&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
```c
    volatile int a=11;
```

#### 使用场景（硬件寄存器访问）（可能在程序逻辑之外被修改的变量上）
[“19 类型修饰符(三)_volatile”页上的图片](onenote:#19%20类型修饰符\(三\)_volatile&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={8FD51D8E-B429-4698-9DCD-E2E24C5D4312}&object-id={A2081200-C60E-497C-BE50-930497E40174}&44&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)




### 7、


## 杂项
### 1 、总揽
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630075.png|Open: Pasted image 20250706150811.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630075.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630075.png)

### 2 、sizeof
- 3 查看内存容量的工具。              `sizeof a;`

> `sizeof` 的返回值类型是 `size_t`
> 根据 C99 标准，`size_t` 的格式说明符是 `%zu`
> - `z` 表示 `size_t` 类型。
> - `u` 表示无符号整数。

```c
    printf("%zu\n", sizeof(a));           // 正确：输出 int 的大小
    printf("%zu\n", sizeof(a_unsigned));  // 正确：输出 unsigned int 的大小
    printf("%zu\n", sizeof(a_short));     // 正确：输出 short 的大小
    printf("%zu\n", sizeof(a_long));      // 正确：输出 long 的大小
    printf("%zu\n", sizeof(a_long_long)); // 正确：输出 long long 的大小
    printf("%zu\n", sizeof(b));           // 正确：输出 float 的大小
    printf("%zu\n", sizeof(c));           // 正确：输出 double 的大小
    printf("%zu\n", sizeof(d));           // 正确：输出 long double 的大小
```



### 3 、return
- 2 返回的概念。




### 4 、

# 三、运算符
- 3 熟悉常用运算符的典型操作，总结什么时候使用什么运算符

## 算术操作运算
### 1 、总揽
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/8733181d6e9f2caae9613cc6d1859073_MD5.jpeg|Open: file-20250818205048528.png]]
![[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/8733181d6e9f2caae9613cc6d1859073_MD5.jpeg]]


### 2 、+、-



### 3 、* 、/、%
[“20 常用运算符(一) + - * / %”页上的图片](onenote:#20%20常用运算符\(一\)%20+%20%20-%20%20*%20%20\%20%20%25&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={0AFFCDDC-879B-4D5B-B234-35EB55F33D3B}&object-id={53AEB6C5-BD68-43B8-9DFA-2264C8CD83E2}&32&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

- 2 * 耗资源
- 2 % 取一个范围的数  n%m = res【0 - m-1】

- 1 取个位（%）
```c
    int a=139;

    int b=a%10;

    printf("b=%d\n", b);
```
> (base) topeet@ubuntu:~/test/c$ ./app
`b=9`

- 1 去掉个位（`/`）
```c
    int a=139;

    int b=a/10;

    printf("b=%d\n", b);
```
> (base) topeet@ubuntu:~/test/c$ ./app
`b=13`


### 4 、




## 逻辑运算（结果为布尔数）
### 1 、总揽
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630413.png|Open: Pasted image 20250706165107.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630413.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630413.png)

### 2 、|| 、&&
[“21 常用运算符(二)_逻辑运算符”页上的图片](onenote:#21%20常用运算符\(二\)_逻辑运算符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={535722AC-DD20-429C-ADE3-9859A1335733}&object-id={020BCF94-FF9D-4D9B-94BC-46CFE6606EA9}&5E&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
- 1 ||    有1为1                             && 有0为零
```c
    if(a>4||a>2){
        printf("a is greater than 4 or 2\n");

    }

    if(a>4&&a>2){
        printf("a is greater than 4 and 2\n");

    }
```
> (base) topeet@ubuntu:~/test/c$ ./app
`a is greater than 4 or 2`


- 3 执行有先后顺序
[“21 常用运算符(二)_逻辑运算符”页上的图片](onenote:#21%20常用运算符\(二\)_逻辑运算符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={535722AC-DD20-429C-ADE3-9859A1335733}&object-id={020BCF94-FF9D-4D9B-94BC-46CFE6606EA9}&1C&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 3 、>、>= 、<   、 <=



### 4 、！（结果取反）
```c
    int a=3;

    if(!(a>4||a>2)){
        printf("a is greater than 4 or 2\n");

    }
```
> (base) topeet@ubuntu:~/test/c$ ./app
(base) topeet@ubuntu:~/test/c$ 


### 5、？ ： （三目运算符）
[“21 常用运算符(二)_逻辑运算符”页上的图片](onenote:#21%20常用运算符\(二\)_逻辑运算符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={535722AC-DD20-429C-ADE3-9859A1335733}&object-id={020BCF94-FF9D-4D9B-94BC-46CFE6606EA9}&80&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
```c
    int age=10;

    char *status=(age>=18)?"Adult":"Minor";
    printf("Status: %s\n", status); // 输出：Status: Minor
```
> (base) topeet@ubuntu:~/test/c$ ./app
Status: `Minor`

### 6、


## 位运算
### 1 、总揽
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630580.png|Open: Pasted image 20250706165113.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630580.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630580.png)

### 2 、<<（左移）（高位丢弃，低位补0）
[“22位运算符(一)移位运算符”页上的图片](onenote:#22位运算符\(一\)移位运算符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={6B467C5A-C3CA-485A-BFC8-3310D073C2F8}&object-id={FF970EBB-3516-4468-9143-3EAD9756D5D5}&16&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
```c
    int a=0b11111111111111111111111111111111;

    a<<=31; // 左移2位，相当于乘以4
    print_binary(a); 
    printf("a=%d\n", a); 
```
> (base) topeet@ubuntu:~/test/c$ ./app
>` 10000000000000000000000000000000`
>` a=-2147483648`
> 
> **第一位是符号位，不可移。否则结果为11111111111111111111111111111111**

- 1 等价于将x乘以2的n次方(但更高效)
```c
    int a=1;
    a<<=2; // 左移2位，相当于乘以4

    printf("a=%d\n", a); // 输出：a=4
```
> (base) topeet@ubuntu:~/test/c$ ./app
`a=4`

### 3 、>>（右移）（底位丢弃）
[“22位运算符(一)移位运算符”页上的图片](onenote:#22位运算符\(一\)移位运算符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={6B467C5A-C3CA-485A-BFC8-3310D073C2F8}&object-id={FF970EBB-3516-4468-9143-3EAD9756D5D5}&52&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
- 1 高位正数补零，负数补1（有符号整数而言）
```c
    int a=0b11111111111111111111111111110000;

    a>>=31; 
    print_binary(a); 
    printf("a=%d\n", a); 
```
> (base) topeet@ubuntu:~/test/c$ ./app
`11111111111111111111111111111111`
a=-1


- 2 涉及到乘除法，移位操作能操作得了的，我们尽量用移位操作。
### 4 、&（清零器）
[“23 位运算符(二)与或运算符”页上的图片](onenote:#23%20位运算符\(二\)与或运算符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={87F284D4-D45D-4769-AC69-4A2AC399E1A1}&object-id={8EBF8F17-FE2A-4DA1-A330-09DFAEA3DB41}&16&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
```c
    int a=0b11111111111111111111111111111111;

    // a>>=30; 
    a&=0b1111;


    print_binary(a); 
    printf("a=%d\n", a); 
```
> (base) topeet@ubuntu:~/test/c$ ./app
`00000000000000000000000000001111`
a=15


### 5、|（置1器）
[“23 位运算符(二)与或运算符”页上的图片](onenote:#23%20位运算符\(二\)与或运算符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={87F284D4-D45D-4769-AC69-4A2AC399E1A1}&object-id={8EBF8F17-FE2A-4DA1-A330-09DFAEA3DB41}&7A&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

```c
    int a=0b00000000000000000000000000000110; 

    // a>>=30; 
    a|=0b1111;


    print_binary(a); 
    printf("a=%d\n", a); 
```
> (base) topeet@ubuntu:~/test/c$ ./app
`00000000000000000000000000001111`
a=15


### 6、`^`（相同为零，不同为一）（异或）
[“24 位运算符(三)取反,异或运算符”页上的图片](onenote:#24%20位运算符\(三\)取反,异或运算符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={C3BBD4CF-0D69-4769-A0B6-01C1CB1F7268}&object-id={48942FF3-6B32-4685-998D-FE2BF3E05E41}&20&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
```c
    char a=0b00001111; // 二进制表示的字符

    a^=0b11111111; // 按位异或操作，将低4位和高4位互换

    print_binary_char(a); 
    printf("a=%c\n", a); 
```
> (base) topeet@ubuntu:~/test/c$ ./app
`11110000`
`a=�`


### 7、~（取反）
[“24 位运算符(三)取反,异或运算符”页上的图片](onenote:#24%20位运算符\(三\)取反,异或运算符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={C3BBD4CF-0D69-4769-A0B6-01C1CB1F7268}&object-id={48942FF3-6B32-4685-998D-FE2BF3E05E41}&3A&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
```c
    char a=0b00001111; // 二进制表示的字符

    a=~a; // 按位异或操作，将低4位和高4位互换

    print_binary_char(a); 
    printf("a=%c\n", a); 
```
> (base) topeet@ubuntu:~/test/c$ ./app
`11110000`
`a=�`


### 8、



## 内存访问符号
### 1 、总揽
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630675.png|Open: Pasted image 20250706165119.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630675.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630675.png)

### 2 、()  （限制符）（限制优先级）（函数访问））
[“25 常用运算符(三)_内存访问符”页上的图片](onenote:#25%20常用运算符\(三\)_内存访问符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={EDE206E3-456F-4FD0-BACD-4CF59CAFF370}&object-id={C7CCB882-6A2D-420F-A370-ADCDC0D0EAC3}&2B&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)



### 3 、`[ ]`（内存访问的ID符号）（`有*解引用功能`）
[“25 常用运算符(三)_内存访问符”页上的图片](onenote:#25%20常用运算符\(三\)_内存访问符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={EDE206E3-456F-4FD0-BACD-4CF59CAFF370}&object-id={C7CCB882-6A2D-420F-A370-ADCDC0D0EAC3}&36&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

```c
    char *a="hello"; 

    printf("a=%c\n", a[3]); // 输出：a=l
```
> (base) topeet@ubuntu:~/test/c$ ./app
>` a=l`

### 4 、`{ } `(函数体的限制符)


### 5、`->`（访问该结构体或类的成员）（地址访问符）（`有*解引用功能`）
[“25 常用运算符(三)_内存访问符”页上的图片](onenote:#25%20常用运算符\(三\)_内存访问符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={EDE206E3-456F-4FD0-BACD-4CF59CAFF370}&object-id={C7CCB882-6A2D-420F-A370-ADCDC0D0EAC3}&55&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

```c
    typedef struct mingle {
        int a;
        char b;
        float c;
    }ming,*ming_ptr; // 定义一个结构体 ming 和一个指向 ming 的指针 ming_ptr
    //ming_ptr 是指向 ming 类型的指针类型别名（等价于 struct mingle*）。

    ming_ptr yuan=NULL; // 定义一个指向 ming 结构体的指针
    yuan=(ming_ptr)malloc(sizeof(ming_ptr));

    yuan->a = 10; // 访问结构体成员
    printf("yuan->a=%d\n", yuan->a); // 输出：yuan->a=10
```
> (base) topeet@ubuntu:~/test/c$ ./app
yuan->a=10

- 2 ming_ptr 是指向 ming 类型的指针类型别名（等价于 struct mingle*）。

### 6、` . `（结构体成员访问符）（访问对象的属性或方法。）
[“25 常用运算符(三)_内存访问符”页上的图片](onenote:#25%20常用运算符\(三\)_内存访问符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={EDE206E3-456F-4FD0-BACD-4CF59CAFF370}&object-id={C7CCB882-6A2D-420F-A370-ADCDC0D0EAC3}&5A&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
```c
    typedef struct mingle {
        int a;
        char b;
        float c;
    }ming,*ming_ptr; // 定义一个结构体 ming 和一个指向 ming 的指针 ming_ptr

    ming yuan = {10, 'A', 3.14f}; // 定义并初始化一个 ming 结构体变量

    printf("yuan.a=%d\n", yuan.a); // 输出：yuan.a=10
```
> (base) topeet@ubuntu:~/test/c$ ./app
`yuan.a=10`



### 7、& （取地址运算符）
[“25 常用运算符(三)_内存访问符”页上的图片](onenote:#25%20常用运算符\(三\)_内存访问符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={EDE206E3-456F-4FD0-BACD-4CF59CAFF370}&object-id={C7CCB882-6A2D-420F-A370-ADCDC0D0EAC3}&86&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 8、* （解引用运算符）（访问指针所指向的变量的值）
[“25 常用运算符(三)_内存访问符”页上的图片](onenote:#25%20常用运算符\(三\)_内存访问符&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={EDE206E3-456F-4FD0-BACD-4CF59CAFF370}&object-id={C7CCB882-6A2D-420F-A370-ADCDC0D0EAC3}&8E&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
```c
    char *a="hello"; 

    printf("a=%c\n", *a); // 输出：a=l
```
> (base) topeet@ubuntu:~/test/c$ ./app
`a=h`




## 赋值运算
### 1 、总揽
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630800.png|Open: Pasted image 20250706165123.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630800.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171630800.png)

### 2 、=


### 3 、+=、-=、&=、|=




# 四、C语言内存空间的使用（指针）
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631278.png|Open: Pasted image 20250706214738.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631278.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631278.png)

## 指针概述
### 1 、
- 1 地址的一个代名词、    内存资源的门牌号

### 2 、指针大小（32位系统4B）（64位系统8B）
```c
    char *a="hello"; 

    printf("a=%zu\n", sizeof a); // 输出：a=l
```
> (base) topeet@ubuntu:~/test/c$ ./app
`a=8`

### 3 、指针也就是地址
[“01 指针概述1”页上的图片](onenote:#01%20指针概述1&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={C5A04AA9-D2EE-419B-9EE9-7152CA0C4A73}&object-id={4DC01927-8B1A-4A31-9BF0-61C21F5A0BD2}&11&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

- 2  指针变量存放指针这个概念的盒子

### 4 、指针变量类型（内存的读取方法）
```c
    char *a="hello"; 

    int *b=(int*)a; // 将字符串指针转换为整数指针

    print_binary(b[0]); // 输出字符串前4个字符的二进制表示
    printf("a=%d\n", b[0]); 

```
> (base) topeet@ubuntu:~/test/c$ ./app
01101100011011000110010101101000
`a=1819043176`

- 2 `"hello"` 实际占6字节，但 `b[0]` 仅读取前4字节，最后的 `\0` 会被忽略。

#### 十进制差异：
> - 用户输出为 **1,819,043,176**，对应的十六进制为 `0x6C6C65A8`，与预期不符。
> - **可能原因**：
> 	1. **内存对齐错误**：强制类型转换可能导致未对齐访问（如地址 `0x1000` 未 4 字节对齐），读取了额外数据。
> 	2. **编译器优化或平台差异**：某些编译器可能插入填充字节或优化字符串存储。
> 	3. **`print_binary` 实现错误**：函数可能错误地处理了符号位或字节顺序。
> 


### 5、pc机是小端字节（高在高，低在低）
- 1 变量分配从内存地址的高到低位分配的
- 2 读取时从下往上读取。

[“02 指针概述2_举例1”页上的图片](onenote:#02%20指针概述2_举例1&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={A6BA3102-E56B-402B-B390-170260F87FD6}&object-id={6E5A0A4E-8089-4DB6-87AF-A2324E10BBB3}&3A&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
- 1 0x12345678在内存的分布
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/fde1f5dc0661e5ff45b306d47b79902b_MD5.jpeg|Open: file-20250818232211403.png]]
![[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/fde1f5dc0661e5ff45b306d47b79902b_MD5.jpeg]]


- 1 从地址低位开始读，一个字节一个字节的读。      
- 2 故读到的为：0x78563412

#### 以“hell”的分布读取举例
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/b500ec4d1bc7597b58adcd602f7db7cb_MD5.jpeg|Open: file-20250818233920497.png]]
![[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/b500ec4d1bc7597b58adcd602f7db7cb_MD5.jpeg]]
- 2 <span style="background:#affad1">变量分配从内存地址的高到低位分配的</span>
- 1 从地址低位开始读，一个字节一个字节的读。     
- 2 故读到的为：108 108 101 104
> 组合后的32位整数为：0x6C6C6568
> 十进制结果：**1,819,043,112**。





### 6、指针初始化
- 1 在C语言中，指针变量的值必须是<span style="background:#affad1">合法的内存地址</span>（例如变量的地址、动态分配的内存地址等）
```c
char *p = &a；
```

#### 将字符串赋值给字符指针
```c
char *p = "hello"; // p 存储的是字符串 "hello" 的首地址
```

> - 编译器将`"hello"`存储在常量区，内容为`{'h', 'e', 'l', 'l', 'o', '\0'}`。
> - 双引号返回字符串首字符`'h'`的地址，赋值给指针`p`，此时`p`指向常量区的`'h'`。



## 指针+修饰符
### 1 、指针与 const
[“04 针修饰符const介绍”页上的图片](onenote:#04%20针修饰符const介绍&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={AF3E83EC-2AC8-47AD-BFB0-11A8F8D05BF7}&object-id={A1220846-2C20-4482-8E5D-E877BDE29E19}&12&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

- 1 实例分析
[“05 指针修饰符const举例”页上的图片](onenote:#05%20指针修饰符const举例&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={321D7BBD-EBA6-446B-8C85-E1B5455C371E}&object-id={0134AA87-6B86-4344-A372-72F99E1701B1}&16&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

- 2 写指针赋值的时候，一定要想办法去了解到，我们指针指向的这段内存真正的属性是什么
### 2 、指针与volatile（防止优化指向内存地址）
[“06 指针修饰符volatile、typedef”页上的图片](onenote:#06%20指针修饰符volatile、typedef&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={1F0530AA-51A3-44F8-99C7-1FC3F7705F1E}&object-id={D015A40E-D05B-4EA5-90A6-C4777D1800F2}&16&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

```c
    volatile char *a="hello"; 
```


### 3 、指针与typedef（重命名）
[“06 指针修饰符volatile、typedef”页上的图片](onenote:#06%20指针修饰符volatile、typedef&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={1F0530AA-51A3-44F8-99C7-1FC3F7705F1E}&object-id={D015A40E-D05B-4EA5-90A6-C4777D1800F2}&27&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

```c
    typedef char * string_ptr; // 定义一个指向字符的指针类型别名 string_ptr

    string_ptr a="hello"; 
```

- 1 更复杂的一些声明，给他别名化
- [“06 指针修饰符volatile、typedef”页上的图片](onenote:#06%20指针修饰符volatile、typedef&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={1F0530AA-51A3-44F8-99C7-1FC3F7705F1E}&object-id={D015A40E-D05B-4EA5-90A6-C4777D1800F2}&69&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)




## 指针+运算符

### 1、指针与++、--、+、-
[“07 指针运算符加减标签操作”页上的图片](onenote:#07%20指针运算符加减标签操作&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={0AC375D6-8789-447C-95A4-98E0FAA3E05D}&object-id={62B6C23C-158C-428E-896A-DC1DC339DC36}&12&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
```c
    typedef char * string_ptr; // 定义一个指向字符的指针类型别名 string_ptr

    string_ptr a="hello"; 

    a++;
    a+=2;

    printf("a=%c\n", a[0]); 
```
> (base) topeet@ubuntu:~/test/c$ ./app
`a=l`

- 1 p++ 、p--      更新地址  会把P的值也改变
- 1 指针的加法、减法运算，实际上加的是一个单位（指针变量）
### 2、`变量名[n]`的使用（会取出内容）
[“07 指针运算符加减标签操作”页上的图片](onenote:#07%20指针运算符加减标签操作&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={0AC375D6-8789-447C-95A4-98E0FAA3E05D}&object-id={62B6C23C-158C-428E-896A-DC1DC339DC36}&3F&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 3、指针运算符加减举例
[“08 指针运算符加减举例1”页上的图片](onenote:#08%20指针运算符加减举例1&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={E661947D-DB45-4823-BE4B-A945477A30C2}&object-id={AE9E74FE-14B4-440E-B296-87D0267412A6}&16&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

- 1 变量分配从内存地址的高到低位分配的
- 2 读取时从下往上读取。
```c
    int a=0x12345678;
    int b=0x99991199;

    int *p1=&b;
    char *p2=(char*)&b; // 将整数指针转换为字符指针

    printf("p1[1]=%x\n", p1[1]); // 输出整数a
    printf("p2[1]=%x\n", p2[1]); // 输出字符指针的第二个元素（即 b 的高位）
```
> (base) topeet@ubuntu:~/test/c$ ./app
`p1[1]=12345678`
`p2[1]=11`
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/4003a611d878dd2cfdda4878118c8ac1_MD5.jpeg|Open: file-20250819140250412.png]]
![[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/4003a611d878dd2cfdda4878118c8ac1_MD5.jpeg]]




### 4、指针越界访问举例（对const变量操作）
```c
    const int a=0x12345678;
    int b=0x99991199;

    int *p1=&b;

    p1[1]=0x100;

    printf("a=%x\n", a); // 输出整数a
```
> (base) topeet@ubuntu:~/test/c$ ./app
`a=100`


### 5、指针逻辑运算符（>=、 <= 、== 、 !=）（比较）
- 1 指针必须是同类型的比较才有意义
```c
    int a=0x12345678;
    int b=0x99991199;

    int *p1=&a;
    int *p2=&b;

    if(p1>p2) {
        printf("p1 points to a larger address than p2\n");
    } else if(p1<p2) {      
        printf("p1 points to a smaller address than p2\n");
    } else {
        printf("p1 and p2 point to the same address\n");
    }

    if(p1!=p2) {
        printf("p1 and p2 point to different addresses\n");

    }

    if(p1==NULL) {
        printf("p1 is NULL\n");
    } else {
        printf("p1 is not NULL\n");

    }

    printf("a: %p\n", &a);
    printf("b: %p\n", &b);

```
> (base) topeet@ubuntu:~/test/c$ ./app
`p1 points to a smaller address than p2`
p1 and p2 point to different addresses
p1 is not NULL
`a: 0x7ffff422b4a0`
`b: 0x7ffff422b4a4`

> - **栈整体向下增长**，但**栈帧内部的局部变量地址顺序由编译器决定**。
> - 用户观察到 `a` 的地址小于 `b`，说明编译器按声明顺序依次分配地址（`a` 在低地址，`b` 在高地址）。
> - **地址顺序不可依赖**，因为不同编译器或优化选项可能导致不同结果。

| 概念       | 行为说明                              |
| -------- | --------------------------------- |
| 栈增长方向    | 从高地址向低地址扩展（向下增长）。                 |
| 栈帧内部变量布局 | 由编译器决定，可能按声明顺序分配地址（地址递增）。         |
| 地址顺序依赖性  | 不同编译器或优化选项可能导致不同结果，C语言标准未规定顺序。    |
| 实际观察结果   | `a` 的地址低于 `b`，说明编译器按声明顺序分配栈帧内部地址。 |

### 6、


## 多级指针
### 1 、二级指针（存放地址的地址空间）（NULL结束）
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/a2517cd8f14d3da3ec567a7421692265_MD5.jpeg|Open: file-20250819142823058.png]]
![[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/a2517cd8f14d3da3ec567a7421692265_MD5.jpeg]]

- 1 通过多级指针把不相关的几个内存组，合成一种线性关系



#### 双重指针的作用
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631510.png|Open: Pasted image 20250708170542.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631510.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631510.png)



### 2 、多级指针举例分析
```c
int main(int argc,char **argv){

    int i;

    for(i=0;i<argc;i++){
        //传入的其实就是地址值（也就是指针值），printf会对地址进行操作，然后将地址上的值取出来的。所以最终输出的结果是值，而非地址
        printf("argv[%d] is %s\n",i,argv[i]); // 输出命令行参数

    }

}
```
> (base) topeet@ubuntu:~/test/c$ `./app 193 340 49u4 `
argv[0] is ./app
argv[1] is 193
argv[2] is 340
argv[3] is 49u4

- 1 图解分析
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/60679598a7c8cc25ecb414c9c8e85595_MD5.jpeg|Open: file-20250819144045100.png]]
![[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/60679598a7c8cc25ecb414c9c8e85595_MD5.jpeg]]

- 2 实际开发中来说，我们我们大部分能用到二维指针就可以了
- 2 读代码的时候，其实就是在读什么一个人，他在设计这个问题的解决方案
### 3 、




# 五、数组（特殊的指针）（用`[]`解引用）
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631667.png|Open: Pasted image 20250706222458.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631667.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631667.png)

## 数组的定义（内存分配的一种形式）
### 1 、`数据类型 数组名[m]  `
[“13 数组的定义”页上的图片](onenote:#13%20数组的定义&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={1DA6B4B6-5316-46A9-90C0-908500764415}&object-id={DA4FE507-33D4-4179-BD3D-287E1D8B3344}&1C&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
```c
    char buf[100]={"hello world"};
    printf("buf=%s\n", buf); // 输出字符串
```
> (base) topeet@ubuntu:~/test/c$ ./app 
`buf=hello world`


> m的作用域是在申请的时候
>  **数组名是首地址**，是常量
>  **数组可以越界**,编译器管不了
### 2 、注意
```c
char arr[] = "hello"; // 数组arr存储字符串的副本
```

> 数组`arr`在栈中分配内存，内容是字符串常量的拷贝（`{'h', 'e', 'l', 'l', 'o', '\0'}`）。
> - **不可直接赋值**：数组名`arr`是地址常量，不能直接赋值新字符串（如`arr = "new"`会报错）。


## 数组空间的初始化
### 1 、空间的赋值（拷贝）
- 1 按照标签逐一处理
- 
[“14 数组空间的初始化1”页上的图片](onenote:#14%20数组空间的初始化1&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={2C6CEEDE-1139-49A7-8BC9-9276583AE492}&object-id={650104A5-A375-4377-ABF7-ACEFEECF83FC}&3D&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
### 2 、char+数组
```c
char buf1[10] = {"abc"}; // 写法1：使用初始化列表（大括号） 
char buf2[10] = "abc"; // 写法2：直接使用字符串字面量 ，只可以在定义时用。
```
>  两种写法最终生成的机器码完全相同。
> `buf[3]` 会自动存储 `\0`，数组大小必须至少为 `字符串长度 + 1`。

```c
 char buf[10]= {'a','b','c'};        非字符串
```
> 如果希望 `buf` 成为一个完整的字符串，必须手动确保末尾有 `\0`


> 剩余的未初始化元素会自动初始化为 `\0`（空字符），因为C语言规定未显式初始化的数组元素会被默认初始化为0（即`\0`）。
> **字符串数组未尾需有/0。**

### 3 、strcpy（字符串整个拷贝函数）
```c
char *strcpy (char *dest, const char *src)
```

```c
    char buf[100]={"hello world"};

    strcpy(buf,"yang ming yuan");

    printf("buf=%s\n", buf); // 输出字符串
```
> (base) topeet@ubuntu:~/test/c$ ./app 
`buf=yang ming yuan`

- 2 将 src 字符串(包括未尾的`\0`完整地复制到 dest 中

### 4 、strncpy（字符串逐个拷贝函数）
```c
char *strncpy (char *dest,
		      const char *src, size_t n)
```

```c
    char buf[100]={"hello world djsfjlfjfljlfdskspfk;fk;dddd"};

    strncpy(buf,"yang ming yuan",sizeof(buf)-1); // 将字符串复制到字符数组中


    printf("buf=%s\n", buf); // 输出字符串
```
> (base) topeet@ubuntu:~/test/c$ ./app 
`buf=yang ming yuan`

> 如果**n 比src长**，则src中的`\0`也会拷贝过去。
### 5、字符空间（给人看，定义字符串）（\0作为结束标志）
- 1 char buf[10]；
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/0ebe81e803a9fef33666b6de4660cb37_MD5.jpeg|Open: file-20250819161725117.png]]
![[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/0ebe81e803a9fef33666b6de4660cb37_MD5.jpeg]]

### 6、非字符空间（数据采集）
- 1 unsigned char buff[10];
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/c870d00bff61a6ac51672c639c82030a_MD5.jpeg|Open: file-20250819161732139.png]]
![[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/c870d00bff61a6ac51672c639c82030a_MD5.jpeg]]
```c
    unsigned char buf[100]={0x11,0x23};

    printf("buf=%x\n", buf[0]); // 输出字符串
```
> (base) topeet@ubuntu:~/test/c$ ./app 
`buf=11`


### 7、memcpy函数（在内存中复制一块指定大小的数据）（任意类型）

```c
    int buf[10];
    int sensor_buf[100]={00,00,00,23,45,78};

    memcpy(buf,sensor_buf,10*sizeof(int));

    printf("buf[5]=%d\n",buf[5]);
    printf("buf[6]=%d\n",buf[6]);
```
> (base) topeet@ubuntu:~/test/c$ ./app 
`buf[5]=78`
`buf[6]=0`



### 8、注意
> c语言的目的
> 其实就是对资源进行操作
> 而资源就是我们的内存回收我们分配的东西
> 主要把它分成两种
> 一个是字符类的啊
> 一个是非字符类的。

> 你找这个对象过后
> 我们**第一个反应**是什么
> 这个对象到底是什么
> 是**有符号/无符号**
> 然后我们再来看其他的啊
## 数组与指针
### 1 、指针数组（相当于二维指针）（`*a[100]`）
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631781.png|Open: Pasted image 20250707221149.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631781.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631781.png)

- 1 指针数组的话是有好多指针然后指向不同的块

```c
    int a=0x12345678;
    int b=0x99991199;

    int *buf[100];

    buf[0]=&b; // 将整数 b 的地址转换为字符指针

    printf("buf[0]=%x\n", *buf[0]); // 输出整数 b 的值
    printf("buf[1]=%x\n", *buf[1]); // 
```
> (base) topeet@ubuntu:~/test/c$ ./app 
`buf[0]=99991199`
`段错误 (核心已转储)`





### 2 、二维数组与二维指针（没有任何关系）

- 1 二维指针只是在描述什么数，把描述地址的线性关系的一种。实际上是地址的一个存储器。


### 4 、数组与指针的辨别方式
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171632052.png|Open: Pasted image 20250707221644.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171632052.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171632052.png)





### 5、数组指针（`(*a)[100]`）（二维数组）
- 1 一个指针读取多个块，这个相当于是一个指针，然后可以一次性读取100个块
- 2 指针本身可以当数组用。
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171632168.png|Open: Pasted image 20250707222100.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171632168.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171632168.png)

```c
    int b[5][6]={{1,2,3,4,5,6},{7,8,9,1,2,3}};

    int (*l)[6]; // 定义一个指向指针数组的指针 l，包含 5 个指向整数的指针
    l = b; // 将指针数组的地址赋给指针 l

    printf("l[0][0]=%d\n", (*l)[0]); // 输出指针数组 p 的第一个元素的值，即整数 a 的值
    printf("l[0][5]=%d\n", l[0][5]); 
    printf("l[1][5]=%d\n", l[1][5]); 
```
> (base) topeet@ubuntu:~/test/c$ ./app 
`l[0][0]=1`
`l[0][5]=6`
`l[1][5]=3`


### 6、


## 多维数组

### 1、二维数组
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631911.png|Open: Pasted image 20250707221436.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631911.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171631911.png)

- 1 一行一块读内存的       先行再列
```c
    int b[3][4]={
        {1, 2, 3,4},
        {4, 5, 6,7},
        {7, 8, 9,10}
    };

    printf("b[2][2]=%d\n", b[2][2]); // 输出二维数组 b 的第 3 行第 3 列的值
```
> (base) topeet@ubuntu:~/test/c$ ./app 
`b[2][2]=9`


### 3、指针与二维数组
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171632696.png|Open: Pasted image 20250708205625.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171632696.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171632696.png)
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171632981.png|Open: Pasted image 20250708205702.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171632981.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171632981.png)
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171633129.png|Open: Pasted image 20250708205709.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171633129.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171633129.png)



### 4、定义三维数组
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171633636.png|Open: Pasted image 20250708205741.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171633636.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171633636.png)
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171633758.png|Open: Pasted image 20250708205808.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171633758.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171633758.png)
### 5、访问三维数组元素
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171633962.png|Open: Pasted image 20250708205824.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171633962.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171633962.png)
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634098.png|Open: Pasted image 20250708205836.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634098.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634098.png)

### 6、


# 六、结构体
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634235.png|Open: Pasted image 20250707222305.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634235.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634235.png)

## 字节对齐
### 1 、提高性能，牺牲空间
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634328.png|Open: Pasted image 20250707222336.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634328.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634328.png)


- 1 四个字节对齐
- 1 这个结构体的sizeof，最终的结果一定是四的倍数

```c
    struct b{
        int a;
        char c;

    };

    struct b ming;

    printf("%zu\n",sizeof ming);
```
> (base) topeet@ubuntu:~/test/c$ ./app 
`8`


### 2 、结论
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634481.png|Open: Pasted image 20250707222458.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634481.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634481.png)

### 3 、




## 位域
### 1 、


### 2 、


### 3 、

### 4 、
### 5、


### 6、


### 7、


### 8、






# 七、内存分布图
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634611.png|Open: Pasted image 20250707222531.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634611.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634611.png)

## 内存分布思想概述
### 1 、内存的属性
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634749.png|Open: Pasted image 20250707222640.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634749.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634749.png)

### 2 、对main函数名分析（地址）
- 1 函数名其实就是这一块代码的什么代名词，也就是称之为有点像数组的什么标签
- 2 标签其实实质上就是什么，就是个地址啊

```c
    printf("%p\n",main);
```
> (base) topeet@ubuntu:~/test/c$ ./app 
`0x5598ad12021e`



### 3 、内存分布思想（代码定义在内存中的位置）
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634829.png|Open: Pasted image 20250707223021.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634829.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171634829.png)

- 1 代码段好像都很低啊，都在   低     这个位置上
- 1 局部变量也就是我们所说的定义的，int a啊，所以它比较   高
- 1 全局变量跟我们代码段的地址靠的非常非常的近，  低   端地址上
```c
int a;

    
    int b;

    printf("%p\n",main);
    printf("%p\n",&b);
    printf("%p\n",&a);
```
> (base) topeet@ubuntu:~/test/c$ ./app 
`0x55793eadd23e     代码段      低
`0x7fffd7fd18a4     局部变量     高
`0x55793eae0014     全局变量     低




### 4 、总结（内存分布图）
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171635399.png|Open: Pasted image 20250707223422.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171635399.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171635399.png)



### 5、



## 栈空间（局部变量）
### 1 、栈空间定义

- 1 认为系统就会在内存中，专门开辟一段区间。去保存大括号里写的变量的所有局部变量
- 2 当我们的什么这个函数执行完过后，这里面的空间，可以简单的认为它就销毁了。
### 2 、static静态化局部变量（存放在**全局的数据空间**）
```c
int a;
	
	static int b;

    printf("%p\n",main);
    printf("%p\n",&b);
    printf("%p\n",&a);
```
> (base) topeet@ubuntu:~/test/c$ ./app 
0x5632b2bea21e
`0x5632b2bed014
`0x5632b2bed018
- 1 和全局变量存放在同一空间。


- 1 只改变生命周期，不改变作用域，仍然是只在该函数内有效
- 2 仍可在另一个函数中定义该变量
### 3 、nm命令（查看可执行文件、目标文件或库文件中的符号表）
```c
void test(){

    static int b;

}

int main(int argc,char **argv){

    static int b;
}
```
> (base) topeet@ubuntu:~/test/c$ nm app
000000000000401c B a
0000000000004014 `b b.3013
0000000000004018 `b b.3018
0000000000004010 B __ bss_start

- 2 通过添加后缀方式区分

### 4 、





## 堆空间（malloc）
### 1 、堆空间定义
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171636033.png|Open: Pasted image 20250707230052.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171636033.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171636033.png)

- 1 生存在某一个时间段内，就该用到堆空间。
### 2 、malloc()分配好内存
[“23 内存分布之堆空间”页上的图片](onenote:#23%20内存分布之堆空间&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={E4E356C9-523F-4D50-86F5-553EB495D691}&object-id={E4EBD079-5A75-40C7-AA17-8F5F15F93936}&12&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

### 3 、free（）释放内存
[“23 内存分布之堆空间”页上的图片](onenote:#23%20内存分布之堆空间&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={E4E356C9-523F-4D50-86F5-553EB495D691}&object-id={E4EBD079-5A75-40C7-AA17-8F5F15F93936}&56&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)

```c
    typedef struct ming{
        int a;
        char b;


    }ming0,*ming1;

    ming1 a;
    a=(ming1)malloc(sizeof(ming1));

    a->a=11;

    printf("%p\n",a);


    free(a);
```
> (base) topeet@ubuntu:~/test/c$ ./app 
`0x55c621d6e2a0`




## 静态空间(代码段  只读数据段  全局变量)
### 1 、定义
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171636441.png|Open: Pasted image 20250707223546.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171636441.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171636441.png)

- 1 你只能读这段空间内容，这个属性操作系统已经完全给你保护好了

### 2 、汇编文件中的组成进化为可执行文件的过程。
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171636868.png|Open: Pasted image 20250707224035.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171636868.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171636868.png)

- 1 所谓链接，我们把它打包成静态数据，打包成一个最终的可执行程序。


### 5 、size命令（查看可执行文件的内存分布）（静态空间分布段）
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171637503.png|Open: Pasted image 20250707224742.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171637503.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171637503.png)

> (base) topeet@ubuntu:~/test/c$ size app
   **text**	                                 **data**	                             **bss**	    dec	    hex	filename
   1639	                                 600	                              8	   2247	    8c7	app
> **代码段、只读数据段**           **初始化的全局数据**             **未初始化的全局数据**

- 1 这三个段就是我们所说的什么所说的静态数据
- 2 可以认为这几个段，在我们程序运行之前啊，可能内存中已经有了。随着整个程序的消失才会消失


### 5、



# 八、C语言函数的使用
## 函数概述
### 1 、总揽
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171637644.png|Open: Pasted image 20250708135953.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171637644.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171637644.png)

### 2 、函数概述
- 1 一堆代码的集合，用一个标签（函数名）去描述它
- 1 复用化

- 1 其实读代码，为了提高效率，往往是先看声明，然后我们再说你觉得哪个函数有必要读。
### 3 、函数三要素
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171637776.png|Open: Pasted image 20250708140608.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171637776.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171637776.png)

### 4 、用指针保存函数
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171638025.png|Open: Pasted image 20250708140631.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171638025.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171638025.png)

#### 函数名就是地址，知道地址我们就可以调用它
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171638254.png|Open: Pasted image 20250708141107.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171638254.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171638254.png)

```c
void test(int a){

    static int b;
    printf("test\n");
}

void test1(int a){

    static int b;
    printf("test1\n");
}

    void (*p[7])(int);

    p[0]=test;
    p[1]=test1;

    p[0](11);
    p[1](12);

```
> (base) topeet@ubuntu:~/test/c$ ./app 
`test`
`test1`

### 5、定义函数，调用函数
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171638724.png|Open: Pasted image 20250708140732.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171638724.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171638724.png)

### 6、实例分析 - 指针当函数用
```c
    int (*myprintf)(const char *,...);

    myprintf=printf;
    myprintf("myprintf\n");
```
> (base) topeet@ubuntu:~/test/c$ ./app 
myprintf


### 7、实例分析 - 指针数组与函数指针的结合使用
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171638853.png|Open: Pasted image 20250708141340.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171638853.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171638853.png)


```c
void test(int a){

    static int b;
    printf("test\n");
}

void test1(int a){

    static int b;
    printf("test1\n");
}

    void (*p[7])(int);

    p[0]=test;
    p[1]=test1;

    p[0](11);
    p[1](12);

```
> (base) topeet@ubuntu:~/test/c$ ./app 
`test`
`test1`





### 8、




## 输入参数
### 1 、总揽
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171639444.png|Open: Pasted image 20250708140118.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171639444.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171639444.png)

### 2 、输入参数分析
- 1 承上启下的功能

- 1 形参：只是大小作用，没有定义这个变量。
- 2 他是不会管你语法，直接怎么来，就怎么接收而已
- 2 [“第4章 函数实参形参拷贝举例”页上的图片](onenote:#第4章%20函数实参形参拷贝举例&section-id={2C7898E3-CF4E-4DD6-BB38-515BCA6A7286}&page-id={B899C15D-0DF9-44D0-B2F2-9BDF44236936}&object-id={EFE18F33-3025-4485-A3F4-2614FF081D5B}&35&base-path=https://d.docs.live.net/52d4b76bb0ffcf51/Documents/\(RK3568\)Linux驱动开发/嵌入式C语言/嵌入式C语言.one)
### 3 、实参传递给形参的方式 - 拷贝
- 1 按位逐一逐一的赋值

### 4 、值传递（上层调用者保护自己空间不被修改的能力）
```c
void test(int a){

    static int b;
    printf("test\n");
}

    test(11);
```

### 5、地址传递（调用者下层子函数修改自己空间值的方式）
```c
void test(int *a){

    *a=100;

    printf("test\n");
}


    int a=11;

    test(&a);

    printf("%d\n",a);
```
> (base) topeet@ubuntu:~/test/c$ ./app 
test
`100`


### 6、地址传递 - 连续空间的传递
#### 结构体变量（更省空间）
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171639792.png|Open: Pasted image 20250708145702.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171639792.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171639792.png)

```c
//全局变量
typedef struct ming{
    int a;
    char b;


}ming0,*ming1;

void test1(ming1 abc){

    abc->a=100;
    abc->b='b';

    printf("%d\n",abc->a);
    printf("%c\n",abc->b);
    printf("test1\n");
}

    ming1 abc;
    abc=(ming1)malloc(sizeof(ming0));
    test1(abc);

    printf("%d\n",abc->a);
    printf("%c\n",abc->b);
```
> (base) topeet@ubuntu:~/test/c$ ./app 
100
b
test1
100
b



#### 数组名（标签）
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171640340.png|Open: Pasted image 20250708145828.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171640340.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171640340.png)
```c
void test2(int a[10]){
    a[0]=100; // 修改数组的第一个元素

    printf("a[0]=%d\n", a[0]); // 输出修改后的值
    printf("test2\n");


}

    int a[10]={0,1,2,3,4,5,6,7,8,9};

    test2(a); // 调用函数，传递数组
```
> (base) topeet@ubuntu:~/test/c$ ./app 
`a[0]=100`
test2



### 7、连续空间只读性（const）
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171641033.png|Open: Pasted image 20250708150246.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171641033.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171641033.png)

```c
void test2(const int a[10]){

    a[0]=100; // 错误：不能修改 const 数组的内容
    printf("a[0]=%d\n", a[0]); 
    printf("test2\n");


}


    int a[10]={1,2,3,4,5,6,7,8,9,10}; // 定义一个数组 a

    test2(a); // 调用函数，传递数组
```
> (base) topeet@ubuntu:~/test/c$ gcc main.c -o app
> main.c: In function ‘test2’:
> main.c:75:9: `error: assignment of read-only location ‘*a’`
>    75 |     a[0]=100; // 错误：不能修改 const 数组的内容
>       |         ^
> 



### 8、对空间操作的二要素（空间首地址、结束标志）
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171641205.png|Open: Pasted image 20250708150630.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171641205.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171641205.png)



### 9、字符空间传递（char * ）（结束标志是：0x00）
- 1 常见处理举例
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171641585.png|Open: Pasted image 20250708150910.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171641585.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171641585.png)
```c
void test3(char *p){

    int i=0;


    if(p==NULL){
        printf("p is NULL\n");
    }

    while(p[i]){

        printf("p[%d]=%c\n", i, p[i]); // 输出字符串 p 的每个字符
        i++;

    }

}
```
> (base) topeet@ubuntu:~/test/c$ ./app 
p[0]=h
p[1]=e
p[2]=l
p[3]=l
p[4]=o


### 10、非字符空间操作（结束标志是数量长度）
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171641661.png|Open: Pasted image 20250708151424.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171641661.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171641661.png)
- 1 void * :数据空间的标识符

```c
void test3(void *p,int len){

    int i=0;
    unsigned char *p1 = (unsigned char *)p; // 将 void 指针转换为 const char 指针


    for(i=0;i<len;i++){

        printf("p[%d]=%x\n", i, p1[i]); // 输出字符串 p 的每个字符

    }

}

    unsigned char *p; // 定义一个字符串指针 p
    p=(unsigned char *)malloc(5*sizeof(unsigned char));
    p[0]=0x11;
    p[1]=0x22;
    p[2]=0x33;
    p[3]=0x44;
    p[4]=0x55;


    test3(p,5);
```
> (base) topeet@ubuntu:~/test/c$ ./app 
p[0]=11
p[1]=22
p[2]=33
p[3]=44
p[4]=55



> - 1 void 指针作为函数形参时，可以接受任何类型指针传入
> - 2 最终的操作一定要**把void * 转换成具体类型**

#### 非字符空间中的一种标准声明
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171642240.png|Open: Pasted image 20250708151526.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171642240.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171642240.png)


### 11、函数地址传递总结
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171642554.png|Open: Pasted image 20250708152913.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171642554.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171642554.png)


### 12、输入参数是二维指针
- 1 上层函数，希望这个子函数帮我们更新一下什么地址空间
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171643062.png|Open: Pasted image 20250708165820.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171643062.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171643062.png)



### 13、








## 返回值 - 返回基本数据类型（提供启下功能的一种表现形式）
### 1 、总揽 
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171643524.png|Open: Pasted image 20250708140214.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171643524.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171643524.png)

### 2 、基本语法
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171625198.png|Open: Pasted image 20250708165458.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171625198.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171625198.png)


### 3 、不能返回数组



## 返回值 - 返回连续空间类型（指针类型）
### 1 、总揽
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171625475.png|Open: Pasted image 20250708140254.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171625475.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171625475.png)

### 2 、注意 - 指针指向堆区或者静态存储区
- 1 返回什么空间类型，就定义一个一模一样的空间类型接收

```c
unsigned char *test5(){

    unsigned char *p; // 定义一个字符串指针 p
    p=(unsigned char *)malloc(5*sizeof(unsigned char));
    p[0]=0x11;
    p[1]=0x22;
    p[2]=0x33;
    p[3]=0x44;
    p[4]=0x55;

    return p; // 返回字符串指针

}

    int i;
    unsigned char *p; // 定义一个字符串指针 p  

    p=test5(); // 调用函数，获取字符串指针
    for(i=0;i<5;i++){

        printf("p[%d]=%x\n", i, p[i]); // 输出字符串 p 的每个字符


    }
```
> (base) topeet@ubuntu:~/test/c$ ./app 
p[0]=11
p[1]=22
p[2]=33
p[3]=44
p[4]=55

### 3 、函数返回类型内部实现概述
- 1 这就是有的人写什么功能的函数能直接先把具体框架定下来
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171625782.png|Open: Pasted image 20250708170313.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171625782.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171625782.png)
### 4 、





# 九、面试题讲解
[想成为嵌入式程序员应知道的0x10个基本问题（面试必备）_想成为嵌入式工程师0x10-CSDN博客](https://blog.csdn.net/weixin_41034400/article/details/113844418)

[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171626116.png|Open: Pasted image 20250708170619.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171626116.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171626116.png)

## 常见面试题_宏定义
### 1 、宏使用注意
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171627078.png|Open: Pasted image 20250708170804.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171627078.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171627078.png)

- 1 红名和红体，红体和defi之间，至少一个什么空格或者是table键

### 2 、在宏定义时用乘法
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171627407.png|Open: Pasted image 20250708171058.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171627407.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171627407.png)
- 1 用下划线，把每一个那个单词之间做一个区分符

- 1 在宏定义时用乘法，编译的过程中就把它计算好了。运行时候cpu就不用计算了。
- 2 小括号圆括号啊做一个保护措施

- 1 加这个UL，移植到别的平台上再在运行，也不会因大小问题而出错。
### 3 、



## 常见面试题_数据申明

[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171627725.png|Open: Pasted image 20250708171701.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171627725.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171627725.png)

- 1 右边的优优先级要高一点

- 1 以a为中心节点右边啊右边是方括号，读内存的方式是块。
- 1 如果右边啊是小括号，读内存的方式是按照函数三要素的方式一块一块读。

## 常见面试题_static

[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171627922.png|Open: Pasted image 20250708171953.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171627922.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171627922.png)




## 常见面试题_其他
### 1 、关键字const有什么含意?
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628114.png|Open: Pasted image 20250708172018.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628114.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628114.png)

### 2 、关键字volatile有什么含意?
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628217.png|Open: Pasted image 20250708172027.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628217.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628217.png)

### 3 、位操作
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628398.png|Open: Pasted image 20250708172112.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628398.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628398.png)

### 4 、访问固定内存位置
[[嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628569.png|Open: Pasted image 20250708172137.png]]
![RK3568（linux学习）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628569.png](嵌入式知识学习（通用扩展）（未）/linux基础知识学习/assets/嵌入式C语言/file-20250810171628569.png)

- 1 觉得看起来方便就可以
### 5、


## 想成为嵌入式程序员应知道的0x10个基本问题（面试必备）
[想成为嵌入式程序员应知道的0x10个基本问题（面试必备）想成为嵌入式工程师0x10-CSDN博客](https://blog.csdn.net/weixin_41034400/article/details/113844418)
### 1 、


### 2 、


### 3 、

### 4 、

### 5、


### 6、


### 7、


### 8、

