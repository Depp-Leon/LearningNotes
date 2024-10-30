## 1. Qt多线程概述

- Qt 默认只有一个主线程，该线程既要处理窗口移动，又要处理各种运算。
- **当要处理的运算比较复杂时，窗口就无法移动**，所以在处理程序时在很多情况下都需要使用多线程来完成。

示例：移动窗口和复杂循环，当主窗口进行复杂循环时，主线程被占用，窗口其他功能比如(移动)使用不了

```c++
#include "widget.h"
#include "ui_widget.h"
#include <QDebug>

Widget::Widget(QWidget *parent) : QWidget(parent), ui(new Ui::Widget)
{
  ui->setupUi(this);
  
  connect(ui->startBtn, &QPushButton::clicked, [=](){
  	// 模拟复杂运算
    for (int i = 1; i <= 10000000; i++)
    {
      ui->lcdNumber->display(QString::number(i));
    }
  });
}
```



## 2. QThread用法-线程类实现逻辑

导入头文件：`#include<QThread>`

方式： 单独定义线程类，线程类完成全部的业务逻辑，主线程只负责启动子线程和退出子线程

### 2.1 使用步骤

1)  创建一个类（`MyThread`），并继承 `QThread`

2)  `QThread` 中有一个 `protected`  级别的  `virtual void run()`， 必须在该函数中实现线程业务逻辑

3)  启动线程必须使用 `start()` 槽函数

4)  在 `MyThread` 中可以定义线程执行完成信号(`isDone`)， 在 `run`函数结束前发射`isDone`信号，来处理线程执行完毕的逻辑

### 2.2 实现

1. 创建一个类，并继承 `QThread`

```c++
class MyThread : public QThread{
    Q_OBJECT
public:
    explicit MyThread(QObject *parent = nullptr);

protected:
    // 线程处理函数，不能直接调用，需要使用start来启动线程
    void run();

signals:
    // 自定义信号，当线程执行完毕时可以发射该信号，告诉系统线程已执行完毕
    void isDone();

public slots:
};
```

2. 在 `run` 函数中实现业务逻辑，继承了`QThread`类，需要实现它的虚拟函数，`run`即要实现的功能，通过主窗口创建该线程对象，调用`start()`方法就会自动启动该线程的`run()`方法

<img src="source/images/Qt%E5%A4%9A%E7%BA%BF%E7%A8%8B/image-20240728170610637.png" alt="image-20240728170610637" style="zoom:67%;" />

```c++
void MyThread::run(){
  // 模拟复杂业务逻辑
  for (int i = 0; i < 10000000; i++){
    qDebug() << i;
  }
  
  // 线程结束时发送 isDone 信号
  emit this->isDone();
}
```

3. 在主窗口中实例化 `MyThread` 类，并调用 `start`方法，来启动线程

<img src="source/images/Qt%E5%A4%9A%E7%BA%BF%E7%A8%8B/image-20240728170814250.png" alt="image-20240728170814250" style="zoom:80%;" />

4. 连接 `myThread` 对象 和 `isFinished` 信号

```c++
Widget::Widget(QWidget *parent) : QWidget(parent), ui(new Ui::Widget)
{
  ui->setupUi(this);

  myThread = new MyThread(this);
  connect(ui->startBtn, &QPushButton::clicked, [=](){
    // 通过 start 来启动线程，不能直接调用 run
    myThread->start();
  });

  // 当子线程结束时发送 isDone 信号，连接信号和槽 可以处理子线程结束之后的事情
  connect(myThread, &MyThread::isDone, [=](){
    qDebug() << "线程结束";
    myThread->quit();
  });
}
```

## 3. QThread用法-子类不实现逻辑

方式：主窗口与线程类配合完成业务逻辑。 

- 子类提供线程能力，但不继承`QThread`
- 主窗口利用子类提供的线程能力进行业务处理。 同时还需要开启和关闭子线程



### 3.1 使用步骤

1. 创建子类（`MyThread`），继承于 `QObject`。

   > 在该类中设置一个能够每秒钟发送一次信号的函数， 功能就是提供线程能力

2. 在主窗口中引入`MyThread`，并将其对象放在线程（`QTread`）中进行调用 （ ==注意：此步并没有启动线程==）

3. 主窗口中接收信号，在匹配的槽函数中进行业务处理

4. 点击按钮时启动线程 ( 使用该方法启动线程时，必须使用信号和槽的方式 )

5. 关闭定时器：  ① 结束线程   ② 结束 while 循环



### 3.2 实现

1. 创建子类（`MyThread`），继承于 `QObject`  (在该类中设置一个能够每秒钟发送一次信号的函数)

```c++
class MyThread : public QObject
{
    Q_OBJECT
public:
    explicit MyThread(QObject *parent = nullptr);

    // 线程处理函数
    void dealThread();

signals:
    // 提供给主窗口的信号
    void mySignal();

public slots:
    // 线程处理函数
    void dealThread();
};
```

```c++
void MyThread::dealThread()
{
    while (1)
    {
        QThread::sleep(1);
        emit this->mySignal();
        qDebug() << "子线程号:" << QThread::currentThread();
    }
}
```

2. 在主窗口中引入`MyThread`，并将其对象放在线程中进行调用

3. 主窗口中接收信号，在匹配的槽函数中进行业务处理

```c++
Widget::Widget(QWidget *parent) : QWidget(parent), ui(new Ui::Widget)
{
  ui->setupUi(this);

  // 实例化自定义的线程，但是不能指定父对象
  myThread = new MyThread;

  // 创建子线程
  QThread qThread = new QThread(this);

  // 将自定义的线程加入到子线程中, 就是将自定义线程移动到系统提供的线程对象中
  // 如果实例化 MyThread 时指定了父对象就不能移动了
  myThread->moveToThread(qThread);

  // 处理线程发射的信号
  connect(myThread, &MyThread::mySignal, [=](){
    static int i = 0;
    i++;
    ui->lcdNumber->display(QString::number(i));
  });
}
```

<img src="source/images/Qt%E5%A4%9A%E7%BA%BF%E7%A8%8B/image-20240728172804243.png" alt="image-20240728172804243" style="zoom:80%;" />

4. 点击按钮时启动线程，该方式启动线程时需要调用  `start()`  方法，同时发送自定义信号 `startSubThread`


```c++
//检测主窗口定义的开始子线程信号
connect(this,&Widget::startSubThread,myThread,&MyThread::dealThread);
//点击按钮信号，实现开启子线程和发射其对于的信号
connect(ui->pushButton,&QPushButton::clicked,[=](){
         qThread->start();
         emit this->startSubThread();
    });
```

5. 关闭定时器：  ① 结束线程   ② 结束 while 循环 ；在 `MyThread` 中设置 `isStop` 属性，用来控制线程和循环的开启或者关闭


```c++
class MyThread : public QObject{
    Q_OBJECT
public:
    explicit MyThread(QObject *parent = nullptr);
    // 线程处理函数
    void dealThread();
    // 修改isStop状态
    void setIsStop(bool flag);
signals:
    // 提供给主窗口的信号
    void mySignal();
public slots:
private:
  	// 是否关闭线程的标志   true 为关闭， false 为不关闭
    bool isStop = false;
};
```

```c++
void MyThread::setIsStop(bool flag){
    this->isStop = flag;
}
```

```c++
//主窗口停止按钮
void Widget::on_stopBtn_clicked()
{
    myThread->setIsStop(true);
    qThread->quit();
    qThread->wait();
}
```

### 3.3 总结

<img src="source/images/Qt%E5%A4%9A%E7%BA%BF%E7%A8%8B/image-20240728174605042.png" alt="image-20240728174605042" style="zoom:80%;" />

### 3.4 案例-轮播图

绘图基础回顾：

1)   绘制图片要在 `void  paintEvent(QPaintEvent *event)`  事件中

2)   使用到的类  `QPainter 、 QPixmap`

3)   窗口中的图片需要进行重绘时，需要使用 `QWidget` 的 `update` 方法





#### 1. 通过点击信号切换图片

##### 1.1 核心思路

核心思路： 将图片资源地址保存在一个 `QVector` 中，通过索引号来访问； 切换到下一张图片就是 索引号自增1 ，再调用 `update()` 方法重绘图片

① 使用数组保存图片资源地址

② 使用索引来控制显示哪张图片

③ 在 `paintEvent` 事件中绘制图片

> 也可以不用事件(不用画家)，直接将图片设置到label中即可

④ 点击下一张按钮时对索引号进行自增1，再重绘图片



##### 1.2 使用画家事件实现

```c++
class Widget : public QWidget{
  Q_OBJECT

public:
  explicit Widget(QWidget *parent = nullptr);
  ~Widget();
  // 绘图事件
  void paintEvent(QPaintEvent *);

private:
  Ui::Widget *ui;
	// imgPaths 用来保存图片资源路径
  QVector<QString> imgPaths;
  // index 用来控制显示 imPaths 中的那张图片
  int index = 0;
};
```

```c++
Widget::Widget(QWidget *parent) : QWidget(parent), ui(new Ui::Widget){
  ui->setupUi(this);

  this->resize(600, 600);

  // 将数组保存在一个 QVector 中
  imgPaths = {
    ":/images/1.jpg", ":/images/2.jpg", ":/images/3.jpg", ":/images/4.jpg"
  };

  // 点击下一张按钮时对索引号自增1, 注意进行边界检测
  connect(ui->nextBtn, &QPushButton::clicked, [=](){
    this->index = ++this->index == 4 ? 0 : this->index;
    // 重绘图片
    this->update();
  });
}

void Widget::paintEvent(QPaintEvent *){
  // 根据索引号绘制图片
  QPainter painter(this);
  QPixmap pix;
  pix.load(this->imgPaths[this->index]);
  painter.drawPixmap(QRect(0, 0, 200, 200), pix);
}
```

##### 1.3  使用label填充实现

```c++
Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);

    this->imageList = {
        ":/images/071.jpg",":/images/100.jpg",":/images/111.jpg",":/images/305.jpg"
    };
    //使用label填充图片
	ui->label->setPixmap(QPixmap(this->imageList[0]));
    ui->label->setScaledContents(true);

    //根据索引号来实现切换图片
    connect(ui->pushButton,&QPushButton::clicked,[this](){
        this->index ++;
        if(this->index == this->imageList.size()){
            this->index = 0;
        }
        ui->label->setPixmap(QPixmap(this->imageList[this->index]));
        ui->label->setScaledContents(true);
    });
}
```



#### 2. 使用线程替换next按钮

##### 2.1 核心思路

核心思路：创建 `MyThread` 类，每三秒向主窗口发送一个信号，主窗口每次接收到该信号时自动对索引号自增1，接着在重绘图片

1)  创建 `MyThread` 类，每3秒向窗口类

2)  主窗口监听到`MyThread`发送的信号后，对索引号进行自增1，接着在重绘图片

3)  程序启动时，开启子线程



##### 2.2 使用QThread实现

1. 创建 `MyThread` 类，每3秒向窗口类发送信号

```c++
class MyThread : public QObject{
    Q_OBJECT
public:
  explicit MyThread(QObject *parent = nullptr);

  // 定义线程函数
  void dealThread();

signals:
  // 自定义信号，每三秒向主窗口发送一次信号
  void mySignal();

};
```

```c++
void MyThread::dealThread(){
  // 每 3 秒向窗口发送一次信号
  while (1){
    QThread::sleep(3);
    emit this->mySignal();
  }
}
```

2. 主窗口监听到`MyThread`发送的信号后，对索引号进行自增1，接着在重绘图片

```c++
class Widget : public QWidget
{
    Q_OBJECT

public:
    ...

signals:
    // 启动子线程的信号
    void startSubThread();

private:
    ...

    MyThread *myThread;
    QThread *qThread;
};
```

```c++
// 实例化自定义类 和 线程类，并将自定义类添加到线程中
myThread = new MyThread;
qThread = new QThread(this);
myThread->moveToThread(qThread);
//监听MyThread中的mySignal信号，每次接到信号时就就切换图片
connect(myThread,&MyThread::mySignal,[this](){
        this->index++;
        if(this->index == this->imageList.size()){
            this->index = 0;
        }
        ui->label->setPixmap(QPixmap(this->imageList[this->index]));
        ui->label->setScaledContents(true);
    });
```

3. 程序启动时，开启子线程

```c++
// 启动子线程就直接调用 dealThread 槽函数
connect(this, &Widget::startSubThread, myThread, &MyThread::dealThread);

// 启动子线程
qThread->start();
emit this->startSubThread();
```

