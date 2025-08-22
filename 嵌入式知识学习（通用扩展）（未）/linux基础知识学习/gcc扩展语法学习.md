---
title: "{{title}}"
aliases: 
tags: 
description: 
source:
---

# 备注(声明)：





# 一、GCC C语言扩展

## 变量类型扩展
### 1 、128位的整数
```c
__int128 a;
unsigned __int128 b;
```

> **范围​**​：有符号（-2^127 ~ 2^127-1），无符号（0 ~ 2^128-1）
> ​**​问题​**​：标准库函数（如`printf`/`scanf`）不支持直接操作
> 2^128=3.4028236692093846346337460743177e+38
> ​**​解决方案​**​：或分高低64位输出：


#### 举例：
```c
    unsigned __int128 val;
    val=(unsigned __int128)0xFFFFFFFFFFFFFFFFULL<< 64| (unsigned __int128)0xFFFFFFFFFFFFFFFFULL;

    printf("High: %lu\nLow: %lu\n", (uint64_t)(val >> 64), (uint64_t)val);
```
> (base) topeet@ubuntu:~/test/gcc$ ./app
`High: 18446744073709551615`
`Low: 18446744073709551615`



### 2 、线程私有变量  （Thread-local storage (TLS)）
```c
// 每个线程会有自己独立的变量a
__thread static int a=9;（GCC扩展）

// C++11及之后
thread_local int var;（标准关键字）
```

> - 每个线程独立存储变量副本，生命周期与线程绑定。
> - 避免共享数据的竞争，​**​无需锁​**​（如线程局部日志、上下文信息）


### 3 、零长度数组
```c
char data[0];（GNU扩展，非C标准）
```
- 2 **核心用途​**​：作为结构体末尾的​**​灵活数组成员​**​（Flexible Array Member）

#### 举例：
```c
    typedef struct line{
        int length;
        char content[0];
    }line;

    line *l = (line *)malloc(sizeof(line) + 100);

    l->content[99]='a';

    printf("%c\n",l->content[99]);
```
> (base) topeet@ubuntu:~/test/gcc$ ./app
>` a`


### 4 、可变参数宏（Variadic Macros）
| 宏定义形式                        | 空参数支持 | 标准性       |
| ---------------------------- | ----- | --------- |
| `debug1(fmt, ...)`           | ❌     | C99       |
| `debug2(fmt, args...)`       | ❌     | GNU扩展     |
| `debug3(fmt, ##__VA_ARGS__)` | ✅     | GNU扩展（推荐） |
> **​##运算符的作用​**:当`__VA_ARGS__`为空时，删除前面的逗号，避免语法错误：

#### 实际应用​
- 1 ​​日志系统​：
```c
#define LOG(fmt, ...) printf("[%s]"fmt, __func__, ##__VA_ARGS__)

LOG("error:%d\n",-1);
```
> (base) topeet@ubuntu:~/test/gcc$ ./app
`[main]error:-1`

- 1 调试增强​​：自动注入文件名和行号：
```c
#define DEBUG(fmt, ...) printf("%s:%d " fmt, __FILE__, __LINE__, ##__VA_ARGS__)

DEBUG("error:%d\n",-1);
```
> (base) topeet@ubuntu:~/test/gcc$ ./app
`main.c:19 error:-1`


### 5、 结构体与数组的指定初始化
```c
    int a[6] = {[2]=1,[3 ... 5]=2,[0]=3};

    for(int i=0;i<6;i++){
        printf("%d ",a[i]);
    }
    printf("\n");

    typedef struct {
        int a;
        int b;
        int c;
    } point;

    point p={.a=11,.b=22,.c=33};

    printf("p.a=%d p.b=%d p.c=%d\n",p.a,p.b,p.c);
```
> (base) topeet@ubuntu:~/test/gcc$ ./app
`3 0 1 2 2 2 `
`p.a=11 p.b=22 p.c=33`




### 6、范围case
- 1 **GNU扩展​**​：`case 'A' ... 'Z':`或 `case 1 ... 5:`
```c
    int a;
    a=6;
    switch(a){

        case 0 ... 9:
            printf("0-9\n");
            break;

        case 11 ... 19:
            printf("11-19\n");
            break;

        default:
            printf("other\n");
            break;

        break;

    }
```
> (base) topeet@ubuntu:~/test/gcc$ ./app
`0-9`



### 7、


### 8、



## 声明函数属性（ `__attribute__((xxx))`）
### 1 、函数执行时机控制​（`constructor`/ `destructor`​）
> **`constructor`/ `destructor`​**
> ​
> - ​**​功能​**​：`constructor`指定函数在 `main()`前自动执行，`destructor`在 `main()`结束或 `exit()`后执行
> - ​**​优先级​**​：
>     - `constructor(priority)`：数值越小优先级越高（如 `__attribute__((constructor(101)))`）
>     - `destructor(priority)`：数值越小优先级越低
>     
> - ​**​注意​**​：C++ 静态对象的构造/析构顺序与 `constructor`/`destructor`的顺序无关，由编译器决定
> - ​**​应用场景​**​：全局资源初始化（如日志系统）、库加载时的自动配置
```c
void __f() { 
    printf("ming\n");
    exit(0);

}

void __f() __attribute((constructor(101)));
```
> (base) topeet@ubuntu:~/test/gcc$ ./app
`ming`


- 2 注意：构造函数优先级从0到100是保留位，不能使用。

### 2 、符号别名与可见性​（alias、visibility）
#### ​​`alias("target")`
- 1 将当前函数声明为另一个函数的别名（等价调用）
```c
//原始函数
void __f() { printf("ming\n");}

//给 __f 起一个别名 __e
void __e() __attribute((weak,alias("__f"),visibility("hidden")));

//实际调用的是 __f
    __e();
```
> (base) topeet@ubuntu:~/test/gcc$ ./app
`ming`



#### `visibility("visibility_type")`

| 类型          | 可见性规则            | 用途          |
| ----------- | ---------------- | ----------- |
| `default`   | 外部模块可见（默认）       | 动态库的公开接口    |
| `hidden`    | **仅当前模块可见**      | 内部函数，避免符号冲突 |
| `internal`  | 模块内可见且不可被覆盖      | 关键内部函数      |
| `protected` | 模块内可见，但外部可引用不可覆盖 | 受限公开接口      |

```c
void __e() __attribute((weak,alias("__f"),visibility("hidden")));
```
- 2 **​应用​**​：动态链接库（.so）的符号导出控制



### 3 、代码优化与行为控制​（​​`noreturn`、​​`deprecated`、`aligned` ）
#### `noreturn`
- 1 声明函数不会返回（如退出函数），避免编译器生成无效返回代码
```c
void __f() { 
    printf("ming\n");
    exit(0);

}

void __f() __attribute((noreturn));
```
- 2 确实需要 `__f` 不返回，需在函数中调用 `exit()`、`abort()` 或进入死循环才行。


#### `deprecated`
- 1 标记函数已废弃，调用时触发编译警告
```c
void __f() { 
    printf("ming\n");
    exit(0);

}

void __f() __attribute((deprecate("改用 new__f()")));
```
> (base) topeet@ubuntu:~/test/gcc$ gcc main.c -o app
main.c:16:1:` warning:` ‘deprecate’ attribute directive ignored [-Wattributes]
   16 | void __ f() __ attribute((deprecate("改用 new__f()")));
      | ^~~~
- 2  代码迁移时兼容旧接口。


#### `aligned` 
- 1 指定函数第一条指令的内存对齐（字节数），`alignment`必须是 2 的幂
```c
void __f() __attribute((aligned(64)));
```
- 2 优化 CPU 指令缓存行访问（如高频调用函数）

- 1 ​**​对齐限制​**​：`aligned`的对齐值受硬件平台最大对齐限制（如 ARM 可能仅支持 16 字节对齐）

### 4 、内存布局控制​（`section`）
- 1 将函数从默认 `.text`段移至自定义段        ​​`section ("section-name")`
```c
void __f() __attribute((section(".uboot")));
```
- 2 嵌入式开发中将关键函数放入特定内存区域（如 Flash 启动代码）
- 2 避免函数被链接器优化移除。

### 5、扩展属性​（​​`format`）
#### `format (printf, arg_idx, first_to_check)`
- 1 对变参函数进行格式字符串检查（类似 `printf`）
```c
void log(const char *fmt,...) __attribute((format(printf,1,2)));
```
- 2 防止格式字符串与参数不匹配导致的运行时错误。

|参数|含义|
|---|---|
|`printf`|表示该函数的格式规则应遵循 `printf` 的格式规则|
|`1`|表示第 **1 个参数**（从 1 开始计数）是格式字符串（即 `fmt`）|
|`2`|表示从第 **2 个参数** 开始是可变参数，需要与格式字符串匹配|



### 6、典型应用场景总结​
| ​**​属性​**​      | ​**​典型用途​**​                |
| --------------- | --------------------------- |
| `constructor`   | 初始化全局资源（数据库连接、硬件寄存器配置）      |
| `alias`+ `weak` | 库的默认实现（允许用户覆盖）              |
| `visibility`    | 动态库的接口隐藏/公开控制               |
| `section`       | 嵌入式系统的引导代码、中断向量表定位          |
| `noreturn`      | 终止进程的函数（`exit()`、`abort()`） |
| `format`        | 自定义日志函数的安全性校验               |




### 7、


### 8、



## 变量属性（定义同时使用才行）
### 1 、`section("section-name")`
- 1 将变量放入自定义内存段（而非默认.data/.bss）
```c
char stack[10000] __attribute((section("STACK"))); // 栈空间分配  
int init_data __attribute__((section("INITDATA")));   // 初始化数据段
```

>  ​**​嵌入式系统​**​：精确控制硬件寄存器映射（如UART设备）
>  ​**​引导程序​**​：分离初始化代码与运行时数据
>  ​**​陷阱​**​：
>     - 未初始化变量可能被放入`COMMON`段，导致链接冲突 → 用`-fno-common`编译选项或`__attribute__((nocommon))`


### 2 、`aligned(alignment)`
- 1 - 指定变量/结构体的内存对齐（alignment=2^n）
```c
    typedef struct {
        int a;
        int b;
        int c;
    } point;
    point p __attribute((aligned(16)))={.a=11,.b=22,.c=33};// 16字节对齐  
    printf("p.a=%d p.b=%d p.c=%d\n",p.a,p.b,p.c);


    int a[10] __attribute((aligned(64)))={1,2,3,4,5,6,7,8,9,10};// 缓存行对齐
    printf("a=%d\n",a[1]);
```
> (base) topeet@ubuntu:~/test/gcc$ ./app
p.a=11 p.b=22 p.c=33
a=2

- 1 **性能优化​**​：
> - 避免CPU跨缓存行访问（如SIMD指令要求128位对齐）
> - 减少DMA传输的边界检查开销


### 3 、







### 4 、




### 5、



## 内敛汇编（asm）
### 1 、`asm` 关键字允许在C代码中内嵌汇编指令


### 2 、基础格式​
```c
asm("int $3"); // 触发调试断点（DebugBreak）
```

### 3 、扩展格式（带操作数）​
```c
    int src=1;
    int dts;

    asm(
        "mov %1,%0\n\t" //mov指令复制值
        "add $1,%0\n\t"  //add指令加1
        : "=r"(dts)  // 输出操作数
        : "r"(src)   // 输入操作数
        : "cc"       // 修改的寄存器
        // : "memory"   // 可选，表示内存可能被修改
    );

    printf("%d\n",dts);
```
> (base) topeet@ubuntu:~/test/gcc$ ./app
`2`

- 1 代码步骤分析
> 输入操作数 `"r"(src)` 表示编译器会将 `src` 的值（即 `1`）加载到某个通用寄存器中（记为寄存器`A(%1)`）。
> `mov %1, %0` 将寄存器A（输入操作数 `%1`）的值复制到另一个寄存器（输出操作数 `%0`，记为寄存器B）。此时寄存器B的值为 `1`。
> `add $1, %0` 对寄存器B（即 `%0`）的值加1。此时寄存器B的值变为 `2`。
> 输出操作数 `"=r"(dts)` 将寄存器B的最终值（`2`）写入变量 `dts`。



### 4 、应用场景​
> - ​**​硬件交互​**​：直接读写端口（如`outb`指令）
> - ​**​性能关键代码​**​：手动优化循环（如CRC32计算）
> - ​**​原子操作实现​**​：通过`lock cmpxchg`指令构建自旋锁


### 5、



## 原子操作内置函数（`__atomic`系列）
### 1. 内存序（Memory Order）模型​​
| **内存序​**​          | ​**​作用​**​                      | ​**​适用场景​**​      |
| ------------------ | ------------------------------- | ----------------- |
| `__ATOMIC_RELAXED` | 无同步约束，仅保证原子性                    | 独立计数器（如统计）        |
| `__ATOMIC_ACQUIRE` | 禁止后续操作重排到该操作前（**读屏障**）          | **锁获取**、共享数据加载后操作 |
| `__ATOMIC_RELEASE` | 禁止前面操作重排到该操作后（写屏障）              | **锁释放**、共享数据修改前操作 |
| `__ATOMIC_ACQ_REL` | 读执行获取屏障，写执行释放屏障（RMW操作）          | CAS操作             |
| `__ATOMIC_SEQ_CST` | 全局顺序一致性（默认），等效`ACQUIRE+RELEASE` | 强一致性需求（如标志位同步）    |
| `__ATOMIC_CONSUME` | 因实现缺陷，实际被提升为`ACQUIRE`           | 不推荐使用             |

> 💡 内存序与Linux内核屏障对应关系
> - `__atomic_thread_fence(__ATOMIC_SEQ_CST)`≡ `smp_mb()`
> - `__atomic_load_n(ptr, ACQUIRE)`≡ `smp_load_acquire()`




### ​​2. 常用原子操作函数​​
#### ​原子加载
```c
type __atomic_load_n(type *ptr, int memorder);      // 返回值
void __atomic_load(type *ptr, type *ret, int memorder); // 结果存ret
```

#### 原子存储
```c
void __atomic_store_n(type *ptr, type val, int memorder);
void __atomic_store(type *ptr, type *val, int memorder);
```

#### ​原子交换
```c
type __atomic_exchange_n(type *ptr, type val, int memorder); // 返回旧值
void __atomic_exchange(type *ptr, type *val, type *ret, int memorder);
```


#### 原子算术运算
```c
​返回新值​​：__atomic_add_fetch, __atomic_sub_fetch, __atomic_and_fetch（支持`&|^~`位操作）
    
返回旧值​：__atomic_fetch_add, __atomic_fetch_sub, __atomic_fetch_or
```


#### ​原子比较交换（CAS）
```c
bool __atomic_compare_exchange_n(type *ptr, type *expected, type desired, 
                                 bool weak, int success_mem, int failure_mem);
```

> - `weak=true`：允许虚假失败（性能更高）
> - `failure_mem`不能强于 `success_mem`

#### ​标志位操作
```c
bool __atomic_test_and_set(void *ptr, int memorder);  // 设置字节为非零值
void __atomic_clear(bool *ptr, int memorder);        // 原子清零
```

#### 内存屏障
```c
void __atomic_thread_fence(int memorder);    // 线程间同步
void __atomic_signal_fence(int memorder);    // 线程内与信号处理函数同步
```


#### 溢出检查函数（`__builtin_xxx_overflow`）
> - ​**​实时检测​**​：在算术运算时检测溢出，返回布尔值指示是否溢出
> - ​**​类型覆盖​**​：支持有符号/无符号整数（`int`/`long`/`long long`）及通用类型

```c
// 加法溢出检测
bool __builtin_add_overflow(type1 a, type2 b, type3 *res); 
bool __builtin_sadd_overflow(int a, int b, int *res);  // 有符号特化

// 减法溢出检测
bool __builtin_sub_overflow(type1 a, type2 b, type3 *res);
bool __builtin_ssub_overflow(int a, int b, int *res);

// 乘法溢出检测
bool __builtin_mul_overflow(type1 a, type2 b, type3 *res);
bool __builtin_smul_overflow(int a, int b, int *res);
```

- 1 编译时检测扩展​
```c
bool __builtin_add_overflow_p(a, b, c);  // 不存储结果，仅返回是否溢出

​​应用场景​​：宏定义中安全执行常量运算
#define SAFE_ADD(a, b) (__builtin_add_overflow_p(a, b, (typeof(a+b))0) ? 0 : (a+b))
```




### ​​3. 自旋锁实现示例​​（`__atomic`）
```c
#include <stdatomic.h> // 引入原子操作所需的头文件
#include <stdio.h>     // 引入标准输入输出库
#include <stdlib.h>    // 引入动态内存分配函数的头文件
#include <threads.h>   // 如果需要创建线程，需引入此头文件
#include <unistd.h>    // 引入sleep函数的头文件

// 定义自旋锁结构体，包含一个原子布尔变量作为锁的状态
typedef struct {
    atomic_bool lock;  // true 表示锁已被占用，false 表示锁可用
} spinlock_t;

/**
 * 尝试获取锁，如果锁不可用，则不断循环直到成功获取锁为止。
 * @param s 指向spinlock_t结构体的指针
 */
void spin_lock(spinlock_t *s) {
    // 使用__atomic_exchange_n原子操作尝试设置锁为true，并等待直到成功
    while (__atomic_exchange_n(&s->lock, true, __ATOMIC_ACQUIRE));
}

/**
 * 释放锁，将锁状态设置为false，表示锁现在是可用的。
 * @param s 指向spinlock_t结构体的指针
 */
void spin_unlock(spinlock_t *s) {
    // 使用__atomic_store_n原子操作将锁状态设置为false
    __atomic_store_n(&s->lock, false, __ATOMIC_RELEASE);
}

int main(){
    // 动态分配spinlock_t结构体所需内存
    spinlock_t *s = malloc(sizeof(spinlock_t)); 
    if (s == NULL) { // 确保内存分配成功
        fprintf(stderr, "内存分配失败\n");
        return EXIT_FAILURE;
    }
    
    // 初始化锁，设置为未锁定状态
    s->lock = false; 

    // 获取锁
    spin_lock(s);

    // 打印消息
    printf("hello world\n");

    // 模拟长时间操作，比如处理数据或等待I/O完成
    sleep(10); 

    // 释放锁
    spin_unlock(s);

    // 释放分配给锁的内存
    free(s);

    return 0;
}

```
- 1 无法在​**​多进程环境​**​中正常工作的根本原因在于：​**​锁状态变量 `s->lock`未在进程间共享​**​。
> - `malloc`分配的内存属于​**​进程私有堆​**​，不同进程的 `s->lock`是独立副本，而非共享变量
> - 进程A加锁修改的是自己的 `lock`副本，进程B无法感知此变化。


### 4 、安全计数器实例（溢出检测）​
```c
#include <stdio.h>
#include <limits.h>

int safe_add(int a, int b) {
    int res;
    if (__builtin_add_overflow(a, b, &res)) {
        fprintf(stderr, "Addition overflow!\n");
        return 0; // 或自定义处理
    }
    return res;
}

int main() {
    printf("%d\n", safe_add(INT_MAX, 1));  // 触发溢出检测
    return 0;
}

```



### 5、




## 其他
### 1 、持有当前函数名的字符串
> - `__func__`: C99标准的一部分
> - ` __FUNCTION__` ：是\_\_func\_\_的另一个名称，用于与旧版本的GCC向后兼容。
> - `__PRETTY_FUNCTION__` ：\_\_func\_\_的另一个名字，在C++中包含了函数签名

```c
    printf("%s\n",__FUNCTION__);
    printf("%s\n",__PRETTY_FUNCTION__);
```
> (base) topeet@ubuntu:~/test/gcc$ ./app
`main`
`main`


### 2 、`__builtin_return_address`（获取调用栈中指定层级的返回地址）
- 1 **作用​**​：返回当前函数或其调用者的​**​返回地址​**​（即函数执行完毕后应跳转的指令地址），常用于调试、性能分析或安全监控场景。
```c
void * __builtin_return_address (unsigned int level)
```
> - ​**​`level = 0`​**​：获取​**​当前函数​**​的返回地址（即调用当前函数的下一条指令地址）
> - ​**​`level = 1`​**​：获取​**​直接调用者​**​（上一级函数）的返回地址
> - ​**​`level = n`​**​：向上追溯第 `n`级调用者的返回地址（例如 `level=2`获取上上级）


```c
    void* caller_addr = __builtin_return_address(0); // 获取函数地址
    printf("Called by: %p\n", caller_addr);
```
> (base) topeet@ubuntu:~/test/gcc$ ./app
`Called by: 0x7fe7a8b3d083`






### 3 、

### 4 、





# 二、预处理过程及其指令

## 预处理过程
### 1 、字符集转换​
> - 源文件被**转换为编译器内部字符集（通常为UTF-8）**
> - 非标准字符可能导致未定义行为（如某些编译器对非ASCII字符的警告）

### 2 、初始文本处理​
#### 行结束符标准化​​
- 1 统一处理`LF`/`CRLF`/`CR`为`\n`，GCC默认无警告但C标准视为未定义行为

#### 三字符序列替换​​（需`-trigraphs`编译选项）

```c
??( → [     ??) → ]     ??< → {      ??> → }     ??= → #
??/ → \     ??' → ^     ??! → |      ??- → ~
```
- 2 默认禁用（`-Wtrigraphs`警告）

#### 反斜杠续行​​
- 1 以`\`结尾的行与下一行合并，后不能有空格

- 2 典型应用：跨行宏定义
```
#define LONG_MACRO \
   do { \
       func1(); \
       func2(); \
   } while(0)
```
#### 注释替换
- 1 所有注释（`//`和`/* */`）被替换为单个空格

### 3、Token化与预处理语言​​
- 1 - **Token分类​**​：标识符、数字、字符串、标点、其他

- 1 **宏扩展优先级​**​：
```c
#define foo() bar
foo()baz  → bar baz   // 优先识别foo()而非foo
a+++++b   → a++ + ++b // 词法分析优先分割最长token
```

> C的关键字对预处理器没有意义；它们是普通的标识符。例如，可以定义名称为关键字的宏。定义了可以被视为预处理关键字的唯一标识符。

### 4 、预处理语言
> token化后，tokens流可能会被直接传输给 compiler’s parser。然后，**如果包含预处理语言，则会先进行转换**。这是大多数人认为的预处理器的工作。

- 1 预处理语言由要执行的指令和要扩展的宏组成。：
> - 包含头文件。这些是可以替换到程序中的声明文件。
> - 宏展开。您可以定义宏，宏是C代码任意片段的缩写。预处理器将在整个程序中用宏的定义替换宏。某些宏是自动为您定义的。
> - 条件编译。您可以根据各种条件包括或排除程序的各个部分。
> - 线路控制。如果使用程序将源文件组合或重新排列为中间文件，然后进行编译，则可以使用行控制来通知编译器每个源行的原始来源。
> - 诊断。您可以在编译时检测问题并发出错误或警告。




### 5、


### 6、






## 头文件包含机制
### 1 、包含指令​
```c
#include <file.h>  // 系统目录搜索（-I添加路径）
#include "file.h"  // 先当前目录，再系统目录（-iquote控制）
```

- 1  编译时指定的目录（通过 `-iquote` 控制）。
```
gcc -iquote /project/include main.c # 添加自定义当前目录路径
```
> (base) topeet@ubuntu:~/test/gcc$ `gcc main.c test.c -o app `
main.c:9:10: fatal error: test.h: `没有那个文件或目录`
    9 | #include "test.h"
      |          ^~~~~~~~
compilation terminated.
(base) topeet@ubuntu:~/test/gcc$ `gcc main.c test.c -o app -iquote ./inc`
(base) topeet@ubuntu:~/test/gcc$ ./app
4




### 2、gcc如何实现调用函数
- 1 头文件（`.h`）只包含函数的声明（declaration），而函数的具体实现（definition）通常位于源文件（`.c`）或库文件（`.a`、`.so`）中

> 标准库函数（如 `printf`、`malloc`）的实现通常在 **GNU C Library（glibc）** 中。例如：
> - `printf` 的实现位于 glibc 的源码中（如 `stdio-common/vfprintf.c`）。
> - 编译时，链接器会**自动链接 glibc 库**。


> **自己编写的函数需要自己链接。**
(base) topeet@ubuntu:~/test/gcc$ `gcc main.c test.c -o app`
(base) topeet@ubuntu:~/test/gcc$ ./app
4



### 3 、



### 4 、



## 宏定义与操作
### 1 、基础宏类型​
| **类型​**​ | ​**​语法​**​              | ​**​示例​**​                    | ​**​注意事项​**​       |
| -------- | ----------------------- | ----------------------------- | ------------------ |
| 无参宏      | `#define IDENT text`    | `#define PI 3.14`             | 末尾无分号              |
| 带参宏      | `#define MACRO(p) text` | `#define SQUARE(x) ((x)*(x))` | 参数全括号避免优先级错误       |
| 可变参数宏    | `#define MACRO(...)`    | `#define LOG(fmt, ...)`       | 用`__VA_ARGS__`引用参数 |

### 2 、高级操作符​
- 1 **字符串化（#）​**​：将参数转为字符串字面值
```d
#define STR(s) #s      // STR(hello) → "hello"
```


- 1 **符号连接（##）​**​：拼接标识符
```d
#define CONCAT(a,b) a##b // CONCAT(var,1) → var1
```
- 2 `##` 的拼接结果 `hellowww` 必须是一个已存在的标识符（如变量、函数名等），否则会报错
```c
#define STR(a,b) a##b

    char *hello="hello";


    char *c;
    c=STR(hell,o);
    printf("%s\n",c);
```
> (base) topeet@ubuntu:~/test/gcc$ ./app
`hello`



- 1 **多级展开​**​：
```d
#define xstr(s) str(s) // 强制二次展开
#define str(s) #s
str(foo) → "foo"       // 直接展开
xstr(foo) → "4"        // foo先展开为4，再字符串化
```
- 2 xstr(foo) → str(s) → str(42) → #42 → "42"
> `xstr(s)` 的宏体是 `str(s)`，且 `s` 没有被 `#` 或 `##` 操作符处理。因此：
> 1. 在替换 `xstr(foo)` 时，参数 `s`（即 `foo`）**会被预先展开**为 `42`。
> 2. 替换为 `str(42)`，此时 `str(42)` 被展开为 `#42` → `"42"`。


### 3 、预定义宏​
| **宏名​**​      | ​**​类型​**​ | ​**​含义​**​           |
| ------------- | ---------- | -------------------- |
| `__FILE__`    | 字符串        | **当前源文件名**           |
| `__LINE__`    | 整型         | **当前行号**（`#line`可修改） |
| `__DATE__`    | 字符串        | 编译日期（"MMM DD YYYY"）  |
| `__TIME__`    | 字符串        | 编译时间（"HH:MM:SS"）     |
| `__func__`    | 字符串        | **当前函数名**（C99标准）     |
| `__cplusplus` | 长整型        | C++版本标识（如`202002L`）  |
|               |            |                      |


### 4 、



##  条件编译与诊断
### 1 、条件编译指令​
```c
#ifdef DEBUG         // 检查DEBUG是否定义
    debug_code();
#elif defined(TEST)  // 多分支条件
    test_code();
#else
    release_code();
#endif
```

### 2 、编译诊断​
- 1 **错误阻断​**​：
```d
#if !defined(C_VERSION)
    #error "C_VERSION must be defined"
#endif
```

- 1 **警告提示​**​：
```d
#if __STDC_VERSION__ < 201112L
    #warning "Recommend C11 or later"
#endif
```

```c
#if __STDC_VERSION__ < 501112L
    #warning "Recommend C11 or later"
#endif
```
> (base) topeet@ubuntu:~/test/gcc$ `gcc main.c -o app`
main.c:57:6: warning:` #warning "Recommend C11 or later" [-Wcpp]`
   57 |     #warning "Recommend C11 or later"
      |      ^~~~~~~



### 3 、



##  特殊指令与最佳实践
### 1 、行控制与Pragma​
```d
#line 42 "new_filename.c"  // 修改行号和文件名
_Pragma("pack(1)")         // 等价于 #pragma pack(1)
```
- 2 `#gragma pack(<opt:pow(2, x>)` 指定后续结构体内存对齐的字节数


- 2 - ​**​`#pragma once`​**​：非标准但广泛支持的头文件守卫替代方案
```c
    #line 100 "tect.c"
    _Pragma("pack(1)")

    printf("%s\n",__FILE__);
    printf("%d\n",__LINE__);

```
> (base) topeet@ubuntu:~/test/gcc$ ./app
`tect.c`
`103`


### 2 、宏定义陷阱规避
|​**​问题​**​|​**​错误示例​**​|​**​解决方案​**​|
|---|---|---|
|多次求值|`SQUARE(i++)`→ `((i++)*(i++))`|改用内联函数或避免副作用参数|
|优先级错误|`#define MUL(a,b) a*b`→ `MUL(2+3,4)=14`|全括号包裹：`((a)*(b))`|
|类型不安全|`MAX("a", 5)`比较指针与整数|函数模板（C++）或类型检查宏|

### 3 、头文件设计规范​
> 1. **内容限制​**​：仅包含声明（**函数原型、`extern`变量、宏**），避免定义
> 
> 2.​**​自包含原则​**​：`myheader.h`应独立编译，显式包含其依赖项
> 	
> 3.**​守卫必加​**​：防止重复包含
```d
// 传统方式
#ifndef UNIQUE_NAME_H
#define UNIQUE_NAME_H
/* ...头文件内容... */
#endif
```

### 4 、预处理的核心价值
> 1. **可移植性​**​：通过条件编译适配不同平台/环境
>     
> 2. ​**​可维护性​**​：宏和头文件实现代码复用与模块化
>     
> 3. ​**​调试灵活性​**​：条件编译控制调试日志输出
>     
> 4. ​**​性能优化​**​：内联宏替代小函数减少调用开销> 




### 5、




## 
### 1 、


### 2 、


### 3 、

### 4 、

### 5、


### 6、


### 7、


### 8、




# 三、

## 
### 1 、


### 2 、


### 3 、

### 4 、

### 5、


### 6、


### 7、


### 8、



## 
### 1 、


### 2 、


### 3 、

### 4 、
### 5、


### 6、


### 7、


### 8、



## 
### 1 、


### 2 、


### 3 、

### 4 、

### 5、


### 6、


### 7、


### 8、


# 四、

## 
### 1 、


### 2 、


### 3 、

### 4 、

### 5、


### 6、


### 7、


### 8、



## 
### 1 、


### 2 、


### 3 、

### 4 、
### 5、


### 6、


### 7、


### 8、



## 
### 1 、


### 2 、


### 3 、

### 4 、

### 5、


### 6、


### 7、


### 8、


# 五、

## 
### 1 、


### 2 、


### 3 、

### 4 、

### 5、


### 6、


### 7、


### 8、



## 
### 1 、


### 2 、


### 3 、

### 4 、
### 5、


### 6、


### 7、


### 8、



## 
### 1 、


### 2 、


### 3 、

### 4 、

### 5、


### 6、


### 7、


### 8、


