---
title: "Qt学习笔记（四）多线程_qt 多线程网络编程-CSDN博客"
source: "https://blog.csdn.net/sincerelover/article/details/143690935?spm=1001.2014.3001.5502"
author:
published:
created: 2025-04-21
description: "文章浏览阅读743次，点赞21次，收藏25次。在Qt中，多线程的处理一般是通过QThread类来实现。QThread代表一个在应用程序中可以独立控制的线程，也可以和进程中的其他线程共享数据。之前我们在Linux应用篇中提到过多线程的概念，使用它的好处是可以同时操作好几个目标，而不是因为上一个目标未结束使得需要的操作陷入阻塞状态。_qt 多线程网络编程"
tags:
  - "clippings"
---
## 系列文章目录

[Qt学习](https://so.csdn.net/so/search?q=Qt%E5%AD%A6%E4%B9%A0&spm=1001.2101.3001.7020) 笔记（一）Qt的基础知识及环境编译（泰山派）  
Qt学习笔记（二）Qt 信号与槽  
Qt学习笔记（三） [网络编程](https://so.csdn.net/so/search?q=%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B&spm=1001.2101.3001.7020)  
Qt学习笔记（四）多线程

---

---

## 前言

  在Qt中，多线程的处理一般是通过QThread类来实现。QThread代表一个在应用程序中可以独立控制的线程，也可以和进程中的其他线程共享数据。之前我们在 Linux 应用篇中提到过多线程的概念，使用它的好处是可以同时操作好几个目标，而不是因为上一个目标未结束使得需要的操作陷入阻塞状态。

---

## 一、QThead

### 1.1 QThead的引入

  QThread 提供了线程启动、停止以及与其他对象通信的能力，我们可以利用主线程用于处理 GUI 操作，而长时间的耗时任务（如文件 I/O、网络请求、大数据处理等）可以放到其他线程中去执行，从而避免界面卡顿现象。QThread 是 Qt 提供的一个线程管理类，封装了原生的线程接口，使得线程的创建、启动、终止和通信更加直观和方便。  
  QThread 线程类是实现多线程的核心类。Qt有两种多线程的方法，其中一种是继承QThread的run() 函数 ，另外一种是把一个继承于QObject的类转移到一个Thread里。两种方法区别不大，用起来都比较方便，但继承QObject的方法更加灵活，笔者这里偏向前者。  
**继承QThread**  
  继承QThread是创建线程的一个普通方法。其中创建的线程只有run()方法在线程里的。其他类内定义的方法都在主线程内。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f0c3c917062a486bab40421546f13a73.png#pic_center)  
  通过上面的图我们可以看到，主线程内有很多方法在主线程内，但是子线程，只有 run()方法是在子线程里的。run()方法是继承于QThread类的方法，用户需要重写这个方法，一般是把耗时的操作写在这个run()方法里面。  
**继承QObject的线程**  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/dc08b3a8ffe34d6c86ba4ff0b6e5398e.png#pic_center)  
  与上一种方法不同，我们先写一个类继承 QObject，通过QObject::moveToThread()方法将它移到一个QThread线程里执行。那么可以通过主线程发送信号去调用QThread线程的方法如上图的fun4()，fun5()等等。这些方法都是在QThread线程里执行的。

### 1.2 相关API

| 函数名 | 描述 |
| --- | --- |
| run() | 线程入口函数 |
| start() | 通过调用run()函数开始执行线程，操作系统根据优先级参数调度线程 |
| currentThread() | 返回一个指向 管理当前执行线程 的QThread的指针 |
| isRunning() | 若线程正在运行返回true，否则返回false |
| sleep()/msleep()/usleep() | 实现线程休眠，单位为秒/毫秒/微秒 |
| wait() | 阻塞线程 |
| quit() | 请求线程退出事件循环，常用于安全关闭线程。 |
| finished() | 当线程结束时会发出该信号，可以通过该信号来实现线程的清理工作 |

**注：使用wait()函数后，线程会阻塞至直到满足以下任何一个条件：**

- 与此QThread对象关联的线程已经完成执行（即当它从run()返回时），若线程已经完成，这个函数将返回true；若线程尚未启动，也返回true；
- 已经过了几毫秒，若时间是ULONG\_MAX（默认值），那么等待永远也不会超时（线程必须从run()返回），若等待超时，此函数返回false与POSIX pthread\_join() 函数类似terminate()| 终止线程的执行。线程可以立即终止，也可以不立即终止，取决于操作系统的调度策略。在terminate()之后使用QThread::wait()来确保终止

### 1.3 QThread的工作流程

  就像前面提到的，笔者偏向继承QThread的写法，这里以这种为主。QThread 的核心流程是创建一个线程对象，将任务移动到该线程，然后启动线程以执行任务。QThread 内部维护了一个事件循环，确保线程可以响应事件和信号槽的触发。线程启动时自动调用 run() 方法，线程结束时会发送 finished 信号。

1. 定义一个类继承 QThread 并重写 run() 函数
2. 线程处理函数里面写入需要执行的复杂数据处理，需要注意以下几点：  
	 - 确定 run() 函数需要执行的具体数据处理任务，例如文件读取、数据分析、图像处理、网络请求等。  
	 - 处理流程设计：在 run() 中合理设计任务处理流程，比如是否需要循环、数据的预处理和后处理、错误处理等。  
	 - 线程间通信：在任务处理过程中，可能需要将进度、结果或错误状态传递回主线程。可以使用信号槽机制实现这些交互。  
	 - 资源释放：确保在任务完成时清理分配的资源，以防止内存泄漏或资源占用。
3. 使用对象调用 start() 函数来启动线程
4. 定义一个信号通知主线程执行完成
5. 线程关闭与资源清理

## 二、继承QThread的代码实现

  代码部分也是利用前段时间写的 [毕设拯救计划（二）基于QT的智能家居（Onenet云）](https://blog.csdn.net/sincerelover/article/details/143515162) 中的代码演示，它主要是在 Qt 中使用 QThread 可以让 DHT11 的数据读取在后台线程中执行，从而避免阻塞主界面线程。如果大家不会dht11的话可以看一下笔者之前写的驱动 [Linux驱动开发笔记（五） 基于设备树与GPIO子系统（含单总线）的操作实验](https://blog.csdn.net/sincerelover/article/details/139328317?sharetype=blogdetail&sharerId=139328317&sharerefer=PC&sharesource=sincerelover&spm=1011.2480.3001.8118) 。  
  首先，新建一个类 DHT11ReaderThread，继承 QThread 并实现数据读取逻辑。

```cpp
// dht11readerthread.h
#ifndef DHT11READERTHREAD_H
#define DHT11READERTHREAD_H

#include <QThread>
#include <QString>

class DHT11ReaderThread : public QThread
{
    Q_OBJECT

public:
    DHT11ReaderThread(QObject *parent = nullptr);
    ~DHT11ReaderThread();

protected:
    // QThread 的主执行函数，前面提到了我们采用的方式主要是在run函数上
    void run() override; 

signals:
    // 用于发送新数据的信号
    void newData(QString temperature, QString humidity); 

private:
    // 用于控制线程的运行状态
    bool keepRunning; 
};

#endif // DHT11READERTHREAD_H
1234567891011121314151617181920212223242526272829
```

  在 dht11readerthread.cpp的run函数中实现 DHT11 的数据读取逻辑，并在读取到数据后通过信号发送给主线程。

```cpp
// dht11readerthread.cpp
#include "dht11readerthread.h"
#include "dht11.h"
#include <QDebug>

DHT11ReaderThread::DHT11ReaderThread(QObject *parent)
    : QThread(parent), keepRunning(true)
{
    dht11_init(); // 初始化 DHT11
}

DHT11ReaderThread::~DHT11ReaderThread()
{
    keepRunning = false;
    dht11_close(); // 关闭 DHT11
}

void DHT11ReaderThread::run()
{
    while (keepRunning) {
        char temperature;
        char humidity;

        // 读取 DHT11 数据
        if (dht11_read(&humidity, &temperature) == 0) {
            // 将读取到的数据格式化为字符串
            QString tempStr = QString("%1°C").arg((int)temperature);
            QString humiStr = QString("%1%").arg((int)humidity);

            // 发出信号，传递新数据
            emit newData(tempStr, humiStr);
        } else {
            qDebug() << "Failed to read data from DHT11.";
        }

        // 设置线程休眠 5 秒，控制读取频率
        msleep(5000);
    }
}
123456789101112131415161718192021222324252627282930313233343536373839
```

  在 MainWindow 的头文件中声明 updateDisplay 槽函数，用于接收来自线程的温湿度数据并更新 UI。

```cpp
// mainwindow.h

#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>

QT_BEGIN_NAMESPACE
namespace Ui { class MainWindow; }
QT_END_NAMESPACE

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

private slots:
    void updateDisplay(const QString &temperature, const QString &humidity); // 更新显示的槽函数

private:
    Ui::MainWindow *ui;
};

#endif // MAINWINDOW_H
123456789101112131415161718192021222324252627
```

  在 MainWindow 中创建 DHT11ReaderThread 的实例，并连接信号以更新 QTextBrowser。

```cpp
// mainwindow.cpp

#include "mainwindow.h"
#include "ui_mainwindow.h"
#include "dht11readerthread.h"

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 创建 DHT11 读取线程实例
    DHT11ReaderThread *readerThread = new DHT11ReaderThread(this);

    // 连接线程的信号到更新 QTextBrowser 的槽
    connect(readerThread, &DHT11ReaderThread::newData, this, &MainWindow::updateDisplay);

    // 启动线程
    readerThread->start();
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 更新 QTextBrowser 显示的槽函数
void MainWindow::updateDisplay(const QString &temperature, const QString &humidity)
{
    QString displayText = QString("Temperature: %1\nHumidity: %2").arg(temperature).arg(humidity);
    ui->textBrowser->setText(displayText);
}
123456789101112131415161718192021222324252627282930313233
```

**注：这里强调一下，不要在main.cpp和mainwindow.cpp中重复实例化对象。**

---

**免责声明：本文参考了网上公开的部分资料，仅供学习参考使用，若有侵权或勘误请联系笔者**

内容来源：csdn.net

作者昵称：云雨歇

原文链接：https://blog.csdn.net/sincerelover/article/details/143690935

作者主页：https://blog.csdn.net/sincerelover

实付 元

[使用余额支付](https://blog.csdn.net/sincerelover/article/details/)

点击重新获取

扫码支付

钱包余额 0

抵扣说明：

1.余额是钱包充值的虚拟货币，按照1:1的比例进行支付金额的抵扣。  
2.余额无法直接购买下载，可以购买VIP、付费专栏及课程。

[余额充值](https://i.csdn.net/#/wallet/balance/recharge)

举报

 [![](https://csdnimg.cn/release/blogv2/dist/pc/img/toolbar/Group-dark.png) 点击体验  
DeepSeekR1满血版](https://ai.csdn.net/?utm_source=cknow_pc_blogdetail&spm=1001.2101.3001.10583) 隐藏侧栏

![程序员都在用的中文IT技术交流社区](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_app.png)

程序员都在用的中文IT技术交流社区

程序员都在用的中文IT技术交流社区

![专业的中文 IT 技术社区，与千万技术人共成长](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_wechat.png)

专业的中文 IT 技术社区，与千万技术人共成长

专业的中文 IT 技术社区，与千万技术人共成长

![关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_video.png)

关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！

关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！

客服 返回顶部

![](https://i-blog.csdnimg.cn/direct/f0c3c917062a486bab40421546f13a73.png#pic_center)