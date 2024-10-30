# 1. TCP

同C++网络编程一样，需要先建立连接，即需要有服务端和客户端(两个窗口)

![image-20240728180822617](source/images/Qt%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B/image-20240728180822617.png)

## 1. 步骤

1. 创建server端口继承于`QWidget`

2. 通过Qt设计师界面类新建一个client界面也继承于`QWidget`

3. 服务端创建Tcp服务，开放监听ip和端口号

   > 同时启动客户端窗口，便于检测测试

4. 服务端通过`tcpsetver`提供的信号`newConnection`，进行后续的写、读、操作(通过`tcpsocket`)

5. 客户端通过`tcpsocket`的`connectToHost`来通过ip和端口号建立与客户端的连接



## 2. 实现

### 2.1 客户端

```c++
#include <QWidget>
#include <QTcpSocket>

namespace Ui {
class client;
}

class client : public QWidget
{
    Q_OBJECT

public:
    explicit client(QWidget *parent = nullptr);
    ~client();

private:
    Ui::client *ui;
    //客户端的TcpSocket
    QTcpSocket *client_socket;
};
```

```c++
client::client(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::client)
{
    ui->setupUi(this);

    this->setWindowTitle("客户端");

    this->client_socket = new QTcpSocket(this);
    // 点击"连接到服务器"时，打开socket，连接到服务端
    connect(ui->connBtn,&QPushButton::clicked,[this](){
        QString ip = ui->ipline->text();
        quint16 port = ui->portline->text().toInt();
        this->client_socket->connectToHost(ip,port);
    });


    // 成功连接到服务器之后，在屏幕上显示连接成功信息
    // 一旦成功连接到服务器之后，就会触发connected信号
    connect(this->client_socket,&QTcpSocket::connected,[this](){
        ui->screen->setText("成功连接到服务器");
    });

    // 点击“发送”时，获取用户输入信息并写入socket
    connect(ui->sendBtn,&QPushButton::clicked,[this](){
        QString msg = ui->msgline->text();
        this->client_socket->write(msg.toUtf8());
        ui->msgline->clear();
    });

    // 当readRead信号被触发时，读取socket中的数据(服务端发送的数据)
    connect(this->client_socket,&QTcpSocket::readyRead,[this](){
       QByteArray buf = this->client_socket->readAll();
       ui->screen->append(buf);
    });

    // 点击"断开连接"时，断开与服务器的连接
    connect(ui->disBtn,&QPushButton::clicked,[this](){
       this->client_socket->disconnectFromHost();
        ui->screen->append("您已断开连接");
    });
}
```



### 2.2 服务端

```c++
#include <QWidget>
#include <QTcpServer>
#include <QTcpSocket>


namespace Ui {
class server;
}

class server : public QWidget
{
    Q_OBJECT

public:
    explicit server(QWidget *parent = nullptr);


    ~server();

private:
    Ui::server *ui;
    QTcpServer *tserver;
    QTcpSocket *server_socket;
};
```

```c++
server::server(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::server)
{
    ui->setupUi(this);

    this->setWindowTitle("服务端");
    this->move(260,150);

    // 启动客户端窗口
    client *c = new client;
    c->show();
    c->move(1060,150);

    // 创建服务器
    this->tserver = new QTcpServer(this);
    //参数1：要监听的客户机地址，如果是any，表示监听所有主机地址，也可以给特定的主机地址进行监听
    //参数2：通过指定的端口号进行访问服务器，如果是0，表示由服务器自动分配。如果非0，则表示指定端口号
    this->tserver->listen(QHostAddress::Any,8888);

    // 当有客户端连接成功之后，将客户端的信息显示到屏幕上
    // 一旦有客户端连接进来就会触发newConnection信号
    connect(this->tserver,&QTcpServer::newConnection,[this](){
       // 获取正在连接的客户端的socket对象信息
        this->server_socket = this->tserver->nextPendingConnection();
        // 通过这个socket对象获取连接进来的客户端的ip地址
        QString ip = this->server_socket->peerAddress().toString();
        quint16 port = this->server_socket->peerPort();
        // 将客户端的信息显示到屏幕上
        ui->screen->setText(QString("%1:%2 -- 成功连接到服务器").arg(ip).arg(port));

        // 一旦有数据从客户端传送过滤，就会触发QTcpSocket::readyRead信号
        // 信号触发之后，就可以从socket中读取数据了
        connect(this->server_socket,&QTcpSocket::readyRead,[this](){
            // readAll() 一次性读取socket中所有的数据
            QByteArray buf = this->server_socket->readAll();
            ui->screen->append(buf);
        });

        // 点击"发送"按钮，得到用户输入的数据，写入socket
        connect(ui->sendBtn,&QPushButton::clicked,[this](){
           QString msg = ui->msgTxt->toPlainText();
           this->server_socket->write(msg.toUtf8());
           ui->msgTxt->clear();
        });

        // 点击"断开连接"时，断开与服务器的连接
        connect(ui->disBtn,&QPushButton::clicked,[this](){
           this->server_socket->disconnectFromHost();
            ui->screen->append("您已断开连接");
        });

    });
}

```







# 2. UDP

Udp不需要建立连接，所以只需要指定ip和端口号就可以使两个客户端(端口)之间进行连接

案例：第一台(8888端口)向第二台(9999端口)发送信息

## 2.1 第一台

```c++
#include <QUdpSocket>

c1::c1(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::c1)
{
    ui->setupUi(this);
    this->setWindowTitle("8888端口");
        
    //主窗口上面开启c2窗口方便展示调试
    c2 *c_2 = new c2;
    c_2->show();


    /////////////////////
    QUdpSocket *c1_socket = new QUdpSocket(this);
    c1_socket->bind(8888);

    // 点击"发送"按钮,获取对方ip，对方端口号，发送给对方的信息
    // 根据ip和端口号，将信息发送出去
    connect(ui->sendBtn,&QPushButton::clicked,[=](){
        QString ip = ui->ipline->text();
        int port = ui->portline->text().toInt();
        QString msg = ui->msgtxt->text();

        c1_socket->writeDatagram(msg.toUtf8(),QHostAddress(ip),port);
    });
}
```



## 2.2 第二台

```c++
#include <QUdpSocket>

c2::c2(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::c2)
{
    ui->setupUi(this);

    this->setWindowTitle("9999端口");

    QUdpSocket *c2_socket = new QUdpSocket(this);
    c2_socket->bind(9999);

    // 一旦有数据发送过来，就会触发reayRead信号
    connect(c2_socket,&QUdpSocket::readyRead,[=](){
        char buf[1024] = {0};
        QHostAddress ip;
        quint16 port;
        c2_socket->readDatagram(buf,1024,&ip,&port);

        ui->screen->append(QString("%1:%2:%3").arg(ip.toString()).arg(port).arg(buf));
        qDebug()<< ip << port;
    });
}
```

> 如果想要切换第二台向第一台发送数据，即再分别补充各自的connect连接readyRead和clicked信号槽功能即可

