客户端界面用Qt,为了方便拿Qt已经封装好的TCP写（直接就可以调用QTcpServer QTcpsocket。。）,只是用了网络框架。

打比方： 客户端-》用户     服务器-》软件商

使用socket进行网络通信，使用tcp协议



**<font style="color:rgb(36, 41, 47);">返回引用</font>**<font style="color:rgb(36, 41, 47);">的主要目的是为了避免不必要的拷贝操作，并且提供对对象的直接访问。具体来说：</font>

+ **<font style="color:rgb(36, 41, 47);">减少拷贝开销</font>**<font style="color:rgb(36, 41, 47);">：返回对象的引用避免了拷贝对象的开销，尤其是当对象比较大或者拷贝代价较高时，返回引用可以显著提高效率。</font>
+ **<font style="color:rgb(36, 41, 47);">避免拷贝构造函数</font>**<font style="color:rgb(36, 41, 47);">：对于复杂对象或大量数据的类，拷贝构造函数可能非常耗时，通过返回引用可以避免这种开销。</font>



### 客户端与服务器的搭建
#### 配置文件
配置文件中存放客户端与服务端用于连接的 ip和端口号（客户端要连接哪一个服务器要把地址给他  当服务器地址改变时我们不希望改变代码所以放在配置文件中）

1. 新建资源文件：项目右键添加文件，选择Qt -> Reaource File
2. 添加前缀 和 文件：前缀是对资源文件的分类，添加文件，新建一个.config文件(127.0.0.1 5000)
3. 读取配置文件
    1. 使用Qfile类操作文件（路径：“ : + 前缀 + 文件名”）
    2. 创建一个QFile对实例，以只读模式打开配置文件，readAll读取文件返回一个QByteArray对象
    3. QByteArray对象转成QString
    4. QString按\r\n进行拆分出每行的字符串
    5. QStringList按下标取出每行数据
    6. 关闭文件

![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1720268538515-e2be187a-6f8d-4e57-85fc-1bc17a5f0646.png)

```cpp
void Client::loadConfig()
{
    //新建一个QFile对象
    QFile file(":/client.config");//固定冒号开头 /是前缀  创建file对象，利用file读取数据
    //只读模式打开文件
    if(file.open(QIODevice::ReadOnly)){
        QByteArray baData = file.readAll();
        QString strData = QString(baData);
        QStringList strList = strData.split("\r\n");//读到的内容按照\r\n进行拆分返回一个列表
        m_strIp = strList.at(0);//ip
        m_usProt = strList.at(1).toUShort();//端口号 字符串转成16位短整型
        qDebug()<<"打开用户配置文件 ip:"<<m_strIp<<"port:"<<m_usProt;
        file.close();
    }else{
    qDebug()<<"打开配置失败";
    }
}
```

【注】

+ **<font style="color:rgb(36, 41, 47);">客户端</font>**<font style="color:rgb(36, 41, 47);">：加载配置文件中的IP和端口号，通过</font><font style="color:rgb(71, 101, 130);">QTcpSocket::connectToHost</font><font style="color:rgb(36, 41, 47);">方法尝试连接到服务器，并通过信号槽机制处理连接成功和数据接收事件。</font>
+ **<font style="color:rgb(36, 41, 47);">服务器</font>**<font style="color:rgb(36, 41, 47);">：加载配置文件中的IP和端口号，通过</font><font style="color:rgb(71, 101, 130);">MyTcpServer::listen</font><font style="color:rgb(36, 41, 47);">方法开始监听连接请求，等待客户端连接。</font>

#### 客户端
tcp进程通信，使用network模块

1. 新建项目：建立一个Qt Widget Application,基类选择QWidget
2. 添加network模块：pro项目文件中添加

```cpp
QT       += core gui network
```

3. 添加配置文件：ip和端口号设置为成员变量，封装加载配置的函数
4. 使用socket连接：
    1. 定义成员变量m_tcpSocket
    2. 使用socket连接到服务器：socket调用connectToHost函数，参数为ip和端口号(ip需要转为QHostAddress)
    3. 连接成功进行提示：连接成功socket发出connected信号，定义一个槽函数并关联(connect要在调用connectToHost之前)

```cpp
// client.h
class Client : public QWidget
{
    Q_OBJECT

public:
    Client(QWidget *parent = nullptr);
    ~Client();
    //加载配置文件
    void loadConfig();

public slots:
    void showConnect();

private:
    Ui::Client *ui;
    //ip 和 端口号
    QString m_strIp;//设置成成员变量在函数外也能访问
    quint16 m_usProt;

    //用于连接服务器的socket
    QTcpSocket m_tcpSocket;

};
// client.h
#include "client.h"
#include "ui_client.h"
#include <QFile>
#include <QDebug>
#include <QHostAddress>

Client::Client(QWidget *parent)
    : QWidget(parent)
    , ui(new Ui::Client)
{
    ui->setupUi(this);
    loadConfig();//加载配置文件

    //关联成功的信号和处理信号的槽函数
    connect(&m_tcpSocket,&QTcpSocket::connected,this,&Client::showConnect);
    //连接到服务器
    m_tcpSocket.connectToHost(QHostAddress(m_strIp),m_usProt);

}
void Client::showConnect()
{
    qDebug()<<"连接服务器成功";
}

```

#### 单例模式
保证一个类**只有一个实例**，控制某些共享资源的访问权限。创建了个对象后，之后再创建一个新对象， 此时会获得之前已创建的对象，而不是一个新对象。

为该实例提供**一个全局访问节点**。单例模式允许在程序的任何地方访问特定对象。但是它可以保护该实例不被其他代码覆盖。

资源共享: 当程序中需要共享某个资源（如数据库连接、文件系统）的实例时，可以使用单例模式确保所有代码使用同一个实例，避免资源的重复创建和消耗。

1. 私有化构造函数，删除拷贝构造和拷贝赋值运算符，防止通过这些途径创建实例
2. 定义获取单例模式对象的静态函数getInstance
3. 该函数返回静态局部对象的引用，静态局部变量是线程安全的
4. 创建实例的地方通过getInstance创建

```cpp
private:
    //为了实现单例模式，私有化构造函数，删除拷贝构造函数和拷贝赋值运算符
    Client(QWidget *parent = nullptr);
    Client(const Client& instance) = delete;
    Client& operator = (const Client&) = delete;

Client &Client::getInstance()
{
    //建立单例模式
    static Client instance;//静态局部变量在C++11后是线程安全的
    return instance;
}

```

#### 服务器
【补】客户端连接通过QTcpSocket类 服务器监听QTcpServer。重写QTcpSocket mytcpsocket继承QTcpSocket，同理

1. 新建Server项目：包含network模块，**加载配置文件**（与客户端一样）
2. **监听**客户端连接：QTcpServer类调用listen函数
    1. 新建MyTcpServer类，继承QTcpServer，实现单例模式
    2. Server类实现MyTcpServer对象调用listen函数（监听函数）
3. 处理客户连接:有客户连接时，QTcpServer会自动调用incomingConnection函数，打印连接成功，将客户端的socked存下来
    1. MyTcpServer类重写incomingConnection函数，打印连接成功
    2. 新建MyTcpServer继承QTcpServer
    3. MyTcpServer类定义成员变量tcpSocketList,存放客户端socket
    4. incomingConnection函数新建MyTcpServer对象，通过incomingConnection函数参数handle初始化，放入列表中

```cpp
// mytcpserver.h
class MyTcpServer : public QTcpServer
{
    Q_OBJECT
public:
    static MyTcpServer&getInstance();
    void incomingConnection(qintptr handle);//有客户端连接会调用该函数，返回客户端信息


private:
    MyTcpServer();
    QList<MyTcpSocket*>m_tcpSocketList;
};

// mytcpserver.cpp
void MyTcpServer::incomingConnection(qintptr handle)
{
    qDebug()<<"新客户端连接";
    //将连接的客户端socket存入socket列表
    MyTcpSocket *pTcpSocked = new MyTcpSocket;
    pTcpSocked->setSocketDescriptor(handle);
    m_tcpSocketList.append(pTcpSocked);
}
```

### 通信协议
协议：长度（数据的长度 整个协议的长度） 数据（64存短的 柔性数组） 类型（协议类型）（结构体存）

#### 柔性数组
结构体中至少有两个元素，最后一个元素是数量未知的数组，结构体的大小不算最后一个元素，申请时间时需按需申请

柔性数组（Flexible Array）是一种C语言中的特性，也称作结构体中的可变数组成员或者零长数组。它的定义形式类似于结构体的最后一个成员是一个数组，但其大小在运行时确定，而不是在编译时确定。这种技术通常用于动态分配内存，并且结合了结构体的优势，使得数据结构更加灵活。

+ 内存分配： 使用 malloc 或者 calloc 来动态分配柔性数组的内存，注意要分配足够的空间来容纳结构体和柔性数组成员。
+ 访问柔性数组： 柔性数组的访问方式与普通数组类似，可以使用索引来访问元素。
+ 释放内存： 使用 free 函数释放动态分配的内存，确保不会发生内存泄漏。

```cpp
#include <stdio.h>
#include <stdlib.h>

typedef struct PDU
{
    int a;
    int b;
    int c;
    int d[];//柔性数组 可变数组,结构体的最后一个元素是数量未知的数组
}PDU;

int main()
{
    printf("Hello World!\n");
    PDU * pdu =malloc(sizeof(PDU)+20*sizeof(char));
    pdu->a = 88;
    memcpy(pdu->d,"hello world",12);//复制内存
    printf("pdu->a = %d, pdu->d = %s\n",pdu->a,(char*)pdu->d);

    free(pdu);
    pdu = NULL;
    return 0;
}
```

#### 协议设计
协议结构体包括：协议数据总长度 实际消息长度 消息类型 实际消息（参数+消息）创建协议数据结构体和协议数据初始化函数



1. 新建C++头文件和对应的cpp文件protocol
2. 定义结构体，定义PDU初始化函数，定义消息类型的枚举值
3. PDU初始化函数参数为消息实际长度，根据长度申请空间，初始化协议数据

```cpp
// protocol.h
#ifndef PROTOCOL_H
#define PROTOCOL_H

typedef unsigned int unit;

enum ENUM_MSG_TYPE{
    ENUM_MSG_TYPE_MIN = 0,
    ENUM_MSG_TYPE_REGIST_REQUEST,
    ENUM_MSG_TYPE_REGIST_RESPEND,
    ENUM_MSG_TYPE_MAX = 0x00fffff,
};

struct PDU{//协议数据单元结构体
    unit uiPDULen;  //总的协议长度
    unit uiMsgLen;  //实际消息的长度
    unit uiMsgtype; //消息类型
    char caData[64];//参数
    char caMsg[];   //实际消息
};

//构建协议数据单元
PDU *mkPDU(unit uiMsgLen);//参数

#endif // PROTOCOL_H

// protocol.cpp
#include "protocol.h"
#include <stdlib.h>
#include <string.h>

PDU *mkPDU(unit uiMsgLen)
{
    unit uiPDULen = sizeof(PDU) + uiMsgLen;//要申请的总长度
    PDU *pdu =(PDU*)malloc(uiPDULen);
    if(pdu == NULL)
    {
        exit(1);//结束进程
    }
    //申请完长度最好初始化一下
    memset(pdu,0,uiPDULen);
    pdu->uiPDULen=uiPDULen;
    pdu->uiMsgLen=uiMsgLen;
    return pdu;
}
```

#### 客户端发送
客户端有一个用于发送消息的ui界面

1. 在client.ui界面添加lineEdit、pushButton，按钮转到槽函数
2. 获取lineEdit中的文字ui->lineEdit->text()
3. 根据消息长度构建pdu,pdu初始化消息类型和实际消息
4. 使用socket发送PDU，m_tcpSocket.write()
5. 释放pdu

```cpp
// client.cpp
void Client::on_pushButton_clicked()
{
    QString strMsg =ui->input_LE->text();
    qDebug()<<"on_pushButton_clicked strMsg="<<strMsg;
    if(strMsg.isEmpty()){
        QMessageBox::warning(this,"发送消息","发送消息不能为空");
        return;
    }
    PDU*pdu=mkPDU(strMsg.size());//构建pdu
    memcpy(pdu->caMsg,strMsg.toStdString().c_str(),strMsg.size());
    pdu->uiMsgtype = ENUM_MSG_TYPE_REGIST_REQUEST;
    m_tcpSocket.write((char*)pdu,pdu->uiPDULen);//发送pdu
    free(pdu);
    pdu = NULL;
}
```

#### 服务器接收
服务器接收数据：MyTcpSocket连接readyRead信号，槽函数里调用read接收数据

1. MyTcpSocket创建一个槽函数recvMsg
2. 读取协议总长度uiPDULen
3. 读取协议除了协议总长度以外的内容（第一个参数pdu的指针要偏移一个unit长度，第二个参数总长度减去一个unit长度）
4. 打印结果
5. 释放pdu
6. 在MyTcpSocket的构造函数里连接信号和槽函数

```cpp
// mytcpsocket.cpp
#include "mytcpsocket.h"
#include "protocol.h"

MyTcpSocket::MyTcpSocket()
{
    connect(this,&QTcpSocket::readyRead,this,&MyTcpSocket::recvMsg);
}

void MyTcpSocket::recvMsg()
{
    qDebug()<<"recvMsg 接收消息长度："<<this->bytesAvailable();
    //读协议长度
    unit uiPDULen = 0;
    this->read((char*)&uiPDULen,sizeof(unit));//总的协议长度
    unit uiMsgLen =uiPDULen - sizeof(unit);
    PDU *pdu = mkPDU(uiMsgLen);
    this->read((char*)pdu + sizeof(unit), uiPDULen - sizeof(unit));//读取协议总长度以外的数据
    qDebug()<<"recvMsg 消息类型"<<pdu->uiMsgtype
            <<" 消息内容："<<pdu->caMsg
            <<" 参数："<<pdu->caData;
    free(pdu);
    pdu = NULL;
}
```

### 注册登录功能
#### 数据库表的设计
![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1720339528295-0822d156-6bb0-471e-9beb-d1a55d1d8bba.png)

【注】tinyint(1)--只有一位的小整形 通常用来表示bool类型

#### 数据库操作
+ 连接数据库
    - server项目.pro添加sql模块
    - 创建继承QObject的操作数据库类OperateDb，实现单例模式
    - OperateDb定义QSqlDatabase的数据库对象m_db
    - 构造函数中通过QsqlDatabase::addDatabase("QMYSQL")构造m_db，析构函数中关闭m_db
    - 定义连接数据库函数:配置并连接
    - 添加驱动文件：将libmysql.dll放入项目对应的bulid/debug目录下

#### 登录界面
![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1720401268292-94aa2345-9e6d-45ba-a698-8301cbd6d87a.png)

#### 添加接口的流程
1. 客户端发送请求(槽函数)
    1. 获取需要发送的数据
    2. 协议中添加对应请求和响应的消息类型
    3. 创建并初始化 pdu:填入消息类型和实际消息
    4. 发送给服务器 pdu
2. 服务器接收、处理并响应请求
    1. 服务器的接收消息函数根据消息类型进行处理
    2. 如果有数据库操作，数据库的操作类添加对应执行 sql语句的函数
    3. 取出消息进行处理
    4. 发送给客户端处理结果
3. 客户端接收响应(并展示结果):客户端的接收消息函数根据消息类型进行处理

#### 注册功能
1. 客户端发送请求(槽函数)
    1. 从输入框中获取用户名和密码
    2. 协议中添加注册功能的请求和响应的消息类型
    3. 创建并初始化 pdu:填入消息类型，用户名和密码放入 caData
    4. 发送给服务器 pdu
2. 服务器接收、处理并响应请求
    1. 服务器的接收消息函数处理注册请求的消息类型
    2. 数据库的操作类添加注册功能的函数，参数为用户名和密码，构建 sql:查询用户是否已存在、插入用户表一条记录
    3. caData 中取出用户名和密码进行处理，调用数据库函数
    4. 发送给客户端处理结果

3.客户端接收响应(并展示结果):客户端的接收消息函数根据消息类型进行处理

客户端发送请求:

```cpp
// protocol.h
enum ENUM_MSG_TYPE{
    ENUM_MSG_TYPE_MIN = 0,
    //注册
    ENUM_MSG_TYPE_REGIST_REQUEST,
    ENUM_MSG_TYPE_REGIST_RESPEND,
    ENUM_MSG_TYPE_MAX = 0x00fffff,
};

// client.cpp
void Client::on_regist_PB_clicked()
{
    qDebug()<<"on_regist_PB_clicked start";
    QString strName = ui->username_LB->text();
    QString strPwd = ui->password_LB->text();
    qDebug()<<"strName:"<<strName
            <<"strPwd:"<<strPwd;
    if(strName.isEmpty()||strPwd.isNull()||strName.size()>32||strPwd.size()>32){
        QMessageBox::critical(this,"注册","用户名或密码非法");
        return;
    }
    PDU*pdu = mkPDU(0);//没使用柔性数组
    pdu->uiMsgtype = ENUM_MSG_TYPE_REGIST_REQUEST;
    memcpy(pdu->caData,strName.toStdString().c_str(),32);
    memcpy(pdu->caData+32,strPwd.toStdString().c_str(),32);//注意指针偏移
    qDebug()<<"uiMsgtype:"<<pdu->uiMsgtype
           <<"caData:"<<pdu->caData;
    m_tcpSocket.write((char*)pdu,pdu->uiPDULen);//发送pdu
    free(pdu);
    pdu = NULL;
}

void Client::recvMsg()
{
    qDebug()<<"recvMsg 接收消息长度："<<m_tcpSocket.bytesAvailable();
    //读协议长度
    unit uiPDULen = 0;
    m_tcpSocket.read((char*)&uiPDULen,sizeof(unit));//总的协议长度

    //读取协议长度以外的数据
    unit uiMsgLen =uiPDULen - sizeof(unit);
    PDU *pdu = mkPDU(uiMsgLen);
    m_tcpSocket.read((char*)pdu + sizeof(unit), uiPDULen - sizeof(unit));//读取协议总长度以外的数据
    qDebug()<<"recvMsg 消息类型"<<pdu->uiMsgtype
            <<" 消息内容："<<pdu->caMsg
            <<" 参数1："<<pdu->caData
            <<" 参数2："<<pdu->caData+32;
    //根据消息类型进行处理
    switch(pdu->uiMsgtype){
    case ENUM_MSG_TYPE_REGIST_RESPEND:{
        bool ret;
        memcpy(&ret,pdu->caData,sizeof(bool));
        if(ret){
            QMessageBox::information(this,"注册","注册成功");
        }else{
            QMessageBox::information(this,"注册","注册失败，用户名或密码非法");
        }
        break;
    }
    default:
        qDebug()<<"未处理的消息类型"<<pdu->uiMsgtype;
        break;
    }

    free(pdu);
    pdu = NULL;
}

// mytcpsocked.cpp
void MyTcpSocket::recvMsg()
{
    qDebug()<<"recvMsg 接收消息长度："<<this->bytesAvailable();
    //读协议长度
    unit uiPDULen = 0;
    this->read((char*)&uiPDULen,sizeof(unit));//总的协议长度

    //读取协议长度以外的数据
    unit uiMsgLen =uiPDULen - sizeof(unit);
    PDU *pdu = mkPDU(uiMsgLen);
    this->read((char*)pdu + sizeof(unit), uiPDULen - sizeof(unit));//读取协议总长度以外的数据
    qDebug()<<"recvMsg 消息类型"<<pdu->uiMsgtype
            <<" 消息内容："<<pdu->caMsg
            <<" 参数1："<<pdu->caData
            <<" 参数2："<<pdu->caData+32;
    //根据消息类型进行处理
    switch(pdu->uiMsgtype){
    case ENUM_MSG_TYPE_REGIST_REQUEST:{
        //处理注册
        qDebug()<<"ENUM_MSG_TYPE_REGIST_REQUEST";
        //读取pdu中的用户名和密码
        char caName[32]={'\0'};//初始化字符串数组
        char caPwd[32]={'\0'};
        memcpy(caName,pdu->caData,32);
        memcpy(caPwd,pdu->caData+32,32);
         qDebug()<<"用户名："<<caName<<" 密码："<<caPwd;
        //数据库处理注册
        bool ret = OperateDb::getInstance().handleRegist(caName,caPwd);
        qDebug()<<"数据库处理结果 ret:"<<ret;
        //构建pdu并发送
        PDU*respdu = mkPDU(0);
        respdu->uiMsgtype =ENUM_MSG_TYPE_REGIST_RESPEND;
        memcpy(respdu->caData,&ret,sizeof(bool));
        write((char*)respdu,respdu->uiPDULen);
        free(respdu);
        respdu=NULL;
        break;
    }
    default:
        qDebug()<<"未处理的消息类型"<<pdu->uiMsgtype;
        break;
    }


    free(pdu);
    pdu = NULL;
}
```

```cpp
// operatedb.cpp
bool OperateDb::handleRegist(char *name, char *pwd)
{
    qDebug()<<"handleRegist start";
    if(name == NULL ||pwd == NULL){
        return false;
    }
    //检查添加的用户是否已经存在
    QString sql = QString("select * from user_info where name ='%1'").arg(name);
    qDebug()<<"检查添加的用户是否存在 sql:"<<sql;
    QSqlQuery q;
    if(!q.exec(sql)||q.next()){
        return false;
    }
    //添加用户
    sql = QString("insert into user_info(name,pwd) values('%1','%2')").arg(name).arg(pwd);
    qDebug()<<"添加用户:"<<sql;
    return q.exec(sql);//exec--执行
}
```

#### 登录功能
1. 客户端发送请求:
    1. 协议中添加登录的请求和响应类型
    2. 客户端的登录按钮槽函数，构建 pdu 并发送
2. 服务器接收并处理请求，响应给客户端
    1. 数据库类新增处理登录函数，将online 字段置为 1
    2. 接收消息函数处理登录，定义一个属性存登录名
    3. 构建 pdu，发送给客户端是否登录成功
3. 客户端接收并展示响应结果:根据响应类型和响应结果显示内容

```cpp
// operatedb.cpp
bool OperateDb::handLeLogin(char *name, char *pwd)
{
    qDebug()<<"handleRegist start name"<<name
            <<"pwd"<<pwd;
    if(name == NULL ||pwd == NULL){
        return false;
    }
    //检查登录用户名和密码是否匹配
    QString sql = QString("select * from user_info where name='%1' and pwd='%2'").arg(name).arg(pwd);
    qDebug()<<"检查登录的用户名和密码是否正确 sql:"<<sql;
    QSqlQuery q;
    if(!q.exec(sql) || !q.next())
    {
        return false;
    }
    //将online置为1
    sql = QString("update user_info set online=1 where name='%1' and pwd='%2'").arg(name).arg(pwd);
    qDebug()<<"online置为1 sql:"<<sql;
    return q.exec(sql);
}

// mytcpsocket.cpp
 case ENUM_MSG_TYPE_LOGIN_REQUEST:{
    //处理登录
    qDebug()<<"ENUM_MSG_TYPE_LOGIN_REQUEST star";
    //读取pdu中的用户名和密码
    char caName[32]={'\0'};//初始化字符串数组
    char caPwd[32]={'\0'};
    memcpy(caName,pdu->caData,32);
    memcpy(caPwd,pdu->caData+32,32);
     qDebug()<<"用户名："<<caName<<" 密码："<<caPwd;
    //数据库处理登录
    bool ret = OperateDb::getInstance().handleRegist(caName,caPwd);
    m_strLoginName = caName;
    qDebug()<<"数据库处理结果 ret:"<<ret;
    //构建pdu并发送
    PDU*respdu = mkPDU(0);
    respdu->uiMsgtype =ENUM_MSG_TYPE_LOGIN_REQUEST;
    memcpy(respdu->caData,&ret,sizeof(bool));
    write((char*)respdu,respdu->uiPDULen);
    free(respdu);
    respdu=NULL;
    break;
    }
    default:
        qDebug()<<"未处理的消息类型"<<pdu->uiMsgtype;
        break;
    }

// mytcpsocket.h
public:
    MyTcpSocket();
    QString m_strLoginName;

// client.cpp
 case ENUM_MSG_TYPE_LOGIN_REQUEST:{
        bool ret;
        memcpy(&ret,pdu->caData,sizeof(bool));
        if(ret){
            QMessageBox::information(this,"登录","登录成功");
        }else{
            QMessageBox::information(this,"登录","登录失败，用户名或密码非法");
        }
    break;
    }
```

#### 处理客户端关闭
+ 当前用户状态设置为0：socket断开连接会发出disconnected信号
    - 数据库的操作类添加下线的函数，指定用户名的online字段设置为0
    - 在socket类增加一个槽函数处理下线clientOffline，连接disconnected信号
    - 处理下线的槽函数调用数据库下线的函数
+ 下线的socket从socket列表中移除
    - 在socket类中定义一个下线信号，通过clientOffine发出信号
    - 在tcpServer定义槽函数，在incomingConnection函数中连接槽
    - 槽函数中删除下线用户的socket

当前用户登录状态置为0：

```cpp
// operatedb.cpp
void OperateDb::handleOffline(const char *name)
{
    qDebug()<<"handleOffline start";
    if(name == NULL){
        qDebug()<<"handleOffline name is NULL";
    }
    QString sql = QString("update user_info set online=0 where name='%1'").arg(name);
    qDebug()<<"online置为1 sql:"<<sql;
    QSqlQuery q;
    q.exec(sql);
}

// mytcpsocket.cpp
void MyTcpSocket::clientOffline()
{
    OperateDb::getInstance().handleOffline(m_strLoginName.toStdString().c_str());
    emit offline(this); //发出信号
}

MyTcpSocket::MyTcpSocket()
{
    connect(this,&QTcpSocket::readyRead,this,&MyTcpSocket::recvMsg);
    connect(this,&QTcpSocket::disconnected,this,&MyTcpSocket::clientOffline);
}
```

下线的socket从socket列表中移除:

```cpp
// mytcpsocket.h
signals:
    void offline(MyTcpSocket *mysocked);

// mytcpsocket.cpp
void MyTcpSocket::clientOffline()
{
    OperateDb::getInstance().handleOffline(m_strLoginName.toStdString().c_str());
    emit offline(this); //发出信号
}

// mytcpserver.h
public slots:
    //移除下线客户端的socket的槽函数
    void deleteSocket(MyTcpSocket*mysocket);

// mytcpserver.cpp
void MyTcpServer::incomingConnection(qintptr handle)
{
    qDebug()<<"新客户端连接";
    //将连接的客户端socket存入socket列表
    MyTcpSocket *pTcpSocket = new MyTcpSocket;
    pTcpSocket->setSocketDescriptor(handle);//新建的socket对应的就是接收到的客户端socked
    m_tcpSocketList.append(pTcpSocket);
//    //打印接收到的客户端
//    foreach (MyTcpSocket *pTcpSocket,m_tcpSocketList){
//        qDebug() <<pTcpSocket;
//    }

    //连接退出登录信号和槽--因为服务器连接之后才知道是哪一个socket
    connect(pTcpSocket,&MyTcpSocket::offline,this,&MyTcpServer::deleteSocket);
}
void MyTcpServer::deleteSocket(MyTcpSocket *mysocket)
{
    m_tcpSocketList.removeOne(mysocket);//从socket列表移除
    mysocket->deleteLater();//延迟移除  自己消灭自己
    mysocket = NULL;

    //测试是否成功移除
    qDebug()<<m_tcpSocketList.size();
    for(int i=0;i<m_tcpSocketList.size();i++)
    {
        qDebug()<<m_tcpSocketList.at(i)->m_strLoginName;
    }
//    //增强for循环
//    foreach(MyTcpSocket * pSocket,m_tcpSocketList)//将每一个m_tcpSocketList中的元素
//    {
//        qDebug()<<pSocket->m_strLoginName;
//    }

}
```

### Ul界面
#### 界面设计
1. 创建三个设计师界面类 Index、Friend、File
2. Index 首页里添加一个 QStackedWidget控件控制切换界面，将 Friend 和 File 添加进去
3. 创建两个按钮用于切换界面:ui->stackedWidget->setCurrentindex(0);，修改样式表:QPushButton{border:none;background-color:rgb(255,255,255);padding:20px;}
4. 两个按钮的垂直策略 Expanding

```cpp
// index.cpp
void Index::on_friend_PB_clicked()
{
    ui->stackedWidget->setCurrentIndex(1);
    ui->friend_PB->setStyleSheet("QPushButton{border:none;background-color:rgb(255,255,255);padding:20px;}");
    ui->file_PB->setStyleSheet("QPushButton{border:none;background-color:rgba(255,255,255,0);padding:20px;}");
}

void Index::on_file_PB_clicked()
{
    ui->stackedWidget->setCurrentIndex(0);
    ui->friend_PB->setStyleSheet("QPushButton{border:none;background-color:rgba(255,255,255,0);padding:20px;}");
    ui->file_PB->setStyleSheet("QPushButton{border:none;background-color:rgb(255,255,255);padding:20px;}");
}
```

#### 界面跳转
1. index首页设置为单例模式
2. 登录成功后的创建Index实例并展示界面，隐藏登录界面

```cpp
//创建index的单例模式
Index &Index::getInstance()
{
    static Index instance;
    return instance;
}
void Client::recvMsg(){
        ...
//登陆成功然后才会弹出界面
Index::getInstance().show();
//登陆成功隐藏登录页面
this->hide();
}
```

### 用户功能
#### 查找用户
Friend 好友界面点击查找用户按钮，转到槽函数，增加一个输入框的弹窗，获取到用户输入的用户名，用户名发送给服务器，服务器查找用户并返回结果，客户端显示结果



+ 客户端槽函数创建输入框的弹窗，从弹窗获取用户输入用户名并发送
+ 服务器查找用户并返回结果
    - 数据库操作类增加一个函数用于查找用户，返回三个结果:-1不存在、0离线、1在线
    - 消息处理函数中调用数据库函数，将结果返回给客户端
+ 客户端接收并显示结果
+ ![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1720510548296-b487859b-5f41-4d87-892b-37af3fa573ee.png)

```cpp
// lgprotocol.h
//查找用户
ENUM_MSG_TYPE_FIND_USER_REQUEST,
ENUM_MSG_TYPE_FIND_USER_RESPEND,

// friend.cpp
//客户端发送请求
void Friend::on_findUser_PB_clicked()
{
    QString strName = QInputDialog::getText(this,"搜索","用户名：");
    qDebug()<<"on_findUser_PB_clicked strName:"<<strName;
    if(strName.isNull())
    {
        return;
    }
    PDU*pdu = mkPDU(0);//没使用柔性数组
    memcpy(pdu->caData,strName.toStdString().c_str(),strName.size());
    pdu->uiMsgtype = ENUM_MSG_TYPE_FIND_USER_REQUEST;
    qDebug()<<"uiMsgtype:"<<pdu->uiMsgtype
           <<"caData:"<<pdu->caData;
    //定义公有方法获取私有成员
    Client::getInstance().getTcpSocket().write((char*)pdu,pdu->uiPDULen);
    free(pdu);
    pdu = NULL;
}

//服务器新建处理请求的函数
//服务器接收并处理请求，响应给客户端
// operatedb.cpp
int OperateDb::handleFindUser(const char *name)
{
    qDebug()<<"handleFindUser start";
    if(name == NULL){
        qDebug()<<"handleFindUser name is NULL";
    }
    QString sql = QString("select online from user_info where name='%1'").arg(name);
    qDebug()<<"查找用户的online sql:"<<sql;
    QSqlQuery q;
    q.exec(sql);
    if(q.next())
    {
        q.value(0).toInt();
    }
    return -1;
}

// mytcpsocket.cpp
case ENUM_MSG_TYPE_FIND_USER_REQUEST:{
    //处理用户查找
    qDebug()<<"ENUM_MSG_TYPE_FIND_USER_REQUEST start";
    //读取pdu中的要查找的用户名
    char caName[32]={'\0'};//初始化字符串数组
    memcpy(caName,pdu->caData,32);
     qDebug()<<"要查找的用户名："<<caName;
    //数据库处理
    int ret = OperateDb::getInstance().handleFindUser(caName);
    qDebug()<<"数据库处理结果 ret:"<<ret;
    //构建pdu并发送
    PDU*respdu = mkPDU(0);
    respdu->uiMsgtype =ENUM_MSG_TYPE_FIND_USER_RESPEND;
    memcpy(respdu->caData,caName,32);
    memcpy(respdu->caData+32,&ret,sizeof(bool));
    write((char*)respdu,respdu->uiPDULen);
    free(respdu);
    respdu=NULL;
    break;
}

//客户端接收并展示响应结果
// client.cpp
case ENUM_MSG_TYPE_FIND_USER_RESPEND:{
    qDebug()<<"ENUM_MSG_TYPE_LOGIN_FIND_USER_RESPEND start";
    char caName[32]={'\0'};
    memcpy(&caName,pdu->caData,32);
    int ret;
    memcpy(&ret,pdu->caData+32,sizeof(int));
    if(ret == -1){
        QMessageBox::information(this,"搜索",QString("%1 不存在").arg(caName));

    }else if(ret == 0){
        QMessageBox::information(this,"搜索",QString("%1 不在线").arg(caName));
    }else{
        QMessageBox::information(this,"搜索",QString("%1 在线").arg(caName));
    }
break;
}


```

#### 在线用户
+ ui 界面:创建 Qt 设计师界面类 OnlineUser，一个显示在线用户的列表框
+ Friend 类中定义私有属性OnlineUser*类型及用于获取的公有方法，在 Friend 构造函数new 该属性

```cpp
//添加界面 friend.h
public:
    OnlineUser * getOnlineUser();
private:
    OnlineUser * m_pOnlineUser;

// friend.cpp
Friend::Friend(QWidget *parent) :QWidget(parent),ui(new Ui::Friend)
{
    m_pOnlineUser = new OnlineUser;
}
OnlineUser *Friend::getOnlineUser()
{
    return m_pOnlineUser;
}
```

+ 客户端发送请求
    - 协议中新增在线用户的消息类型
    - 在线用户按钮的槽函数中构建 pdu 并发送请求
    - 显示在线用户界面
+ 服务器接收、处理并响应请求
    - 数据库操作类增加获取在线用户的函数，结果放到 QStringList里
    - 调用数据库函数，得到 QStringList类型的所有在线用户，取出所有用户名放入 caMsg
    - pdu发送给客户端
+ 客户端接收响应
    - OnlineUser 类新增一个更新列表框的函数，参数为QStringList
    - 客户端的接收消息函数中调用更新列表框的函数，从caMsg 中取出所有用户名传给该函数

```cpp
// protocol.cpp
//在线用户
ENUM_MSG_TYPE_ONLINE_USER_REQUEST,
ENUM_MSG_TYPE_ONLINE_USER_RESPEND,

// friend.cpp
void Friend::on_online_PB_clicked()
{
    if(m_pOnlineUser->isHidden()){
        m_pOnlineUser->show();
    }
    PDU * pdu = mkPDU(0);
    pdu->uiMsgtype=ENUM_MSG_TYPE_ONLINE_USER_REQUEST;
    Client::getInstance().getTcpSocket().write((char*)pdu,pdu->uiPDULen);
    free(pdu);
    pdu = NULL;
}

// operatedb.cpp
QStringList OperateDb::handleOnlineUser()
{
    qDebug()<<"handleOnlineUser start";
    QString sql = QString("select name from user_info where online = 1");
    qDebug()<<"在线用户 sql:"<<sql;
    QSqlQuery q;
    q.exec(sql);
    
    QStringList result;
    while(q.next())
    {
        result.append(q.value(0).toString());
    }
    
    return result;
}

//mytcpsocket.cpp
case ENUM_MSG_TYPE_ONLINE_USER_REQUEST:
    {
        //在线用户
        qDebug()<<"ENUM_MSG_TYPE_ONLINE_USER_REQUEST start";

        //数据库处理
        QStringList ret = OperateDb::getInstance().handleOnlineUser();
        qDebug()<<"数据库处理结果 ret:"<<ret;
        //构建pdu并发送
        PDU*respdu = mkPDU(ret.size()*32);
        respdu->uiMsgtype =ENUM_MSG_TYPE_ONLINE_USER_RESPEND;

        for(int i=0;i<ret.size();i++)
        {
            qDebug()<<"ret.at(i)"<<ret.at(i);
            memcpy(respdu->caMsg+i*32,ret.at(i).toStdString().c_str(),32);
        }
        write((char*)respdu,respdu->uiPDULen);
        free(respdu);
        respdu=NULL;
        break;
    }

// onlineuser.h
class OnlineUser : public QWidget
{
public:
    void showOnlineUser(QStringList userList);
}

void OnlineUser::showOnlineUser(QStringList userList)
{
    ui->listWidget->clear();//不清空的话下线不会删除
    ui->listWidget->addItems(userList);
}

//client.cpp
case ENUM_MSG_TYPE_ONLINE_USER_RESPEND:{
    qDebug()<<"ENUM_MSG_TYPE_ONLINE_USER_RESPEND start";
    unit uiSize = pdu->uiMsgLen/32;
    char caTmp[32];
    QStringList userlist;
    for(unit i = 0; i < uiSize ; i++){
        memcpy(caTmp,pdu->caMsg+32*i,32);
        userlist.append(QString(caTmp));
    }
    Index::getInstance().getFriend()->getOnlineUser()->showOnlineUser(userlist);
break;
}

```

### 函数封装
#### 服务器
MyTcpSocket::recvMsg()

1. 读取 pdu 封装一个函数 readPDU
2. 处理消息类型封装一个函数 handlePDU
3. 新建一个类 MsgHander，专门用于处理不同类型的消息，每个功能定义一个函数去处理
4. MyTcpSocket定义一个成员变量 MsgHandler 指针，在构造函数和析构函数中 new 和delete

【注】一个类中的成员变量想要在另一个类中修改 传引用

```cpp
// mytcpSocket.h
public slots:
    PDU * readPDU();
    PDU * handleMsg(PDU * pdu);
    void sendPDU(PDU * pdu);

// mytcpSocket.cpp
void MyTcpSocket::recvMsg()
{
    PDU * pdu =readPDU();//读pdu
    PDU * respdu = handleMsg(pdu);//处理读取消息
    sendPDU(respdu);//发送结果
    free(pdu);//释放
    pdu = NULL;
}

PDU *MyTcpSocket::readPDU()
{
    qDebug()<<"readPDU 接收消息长度："<<this->bytesAvailable();
    //读协议长度
    unit uiPDULen = 0;
    this->read((char*)&uiPDULen,sizeof(unit));//总的协议长度
    //读取协议长度以外的数据
    unit uiMsgLen =uiPDULen - sizeof(unit);
    PDU *pdu = mkPDU(uiMsgLen);
    this->read((char*)pdu + sizeof(unit), uiPDULen - sizeof(unit));//读取协议总长度以外的数据
    qDebug()<<"readPDU 消息类型"<<pdu->uiMsgtype
            <<" 消息内容："<<pdu->caMsg
            <<" 参数1："<<pdu->caData
            <<" 参数2："<<pdu->caData+32;
    return pdu;
}

PDU *MyTcpSocket::handleMsg(PDU *pdu)
{
    PDU*respdu = NULL;
    //根据消息类型进行处理
    switch(pdu->uiMsgtype){
    case ENUM_MSG_TYPE_REGIST_REQUEST:{
        //处理注册
        qDebug()<<"ENUM_MSG_TYPE_REGIST_REQUEST star";
        //读取pdu中的用户名和密码
        char caName[32]={'\0'};//初始化字符串数组
        char caPwd[32]={'\0'};
        memcpy(caName,pdu->caData,32);
        memcpy(caPwd,pdu->caData+32,32);
         qDebug()<<"用户名："<<caName<<" 密码："<<caPwd;
        //数据库处理注册
        bool ret = OperateDb::getInstance().handleRegist(caName,caPwd);
        qDebug()<<"数据库处理结果 ret:"<<ret;
        //构建pdu并发送
        respdu = mkPDU(0);
        respdu->uiMsgtype =ENUM_MSG_TYPE_REGIST_RESPEND;
        memcpy(respdu->caData,&ret,sizeof(bool));
        break;
    }
    case ENUM_MSG_TYPE_LOGIN_REQUEST:{
    //处理登录
    qDebug()<<"ENUM_MSG_TYPE_LOGIN_REQUEST star";
    //读取pdu中的用户名和密码
    char caName[32]={'\0'};//初始化字符串数组
    char caPwd[32]={'\0'};
    memcpy(caName,pdu->caData,32);
    memcpy(caPwd,pdu->caData+32,32);
     qDebug()<<"用户名："<<caName<<" 密码："<<caPwd;
    //数据库处理登录
    bool ret = OperateDb::getInstance().handleLogin(caName,caPwd);
    m_strLoginName = caName;
    qDebug()<<"数据库处理结果 ret:"<<ret;
    //构建pdu并发送
    respdu = mkPDU(0);
    respdu->uiMsgtype =ENUM_MSG_TYPE_LOGIN_RESPEND;
    memcpy(respdu->caData,&ret,sizeof(bool));
    break;
    }
    case ENUM_MSG_TYPE_FIND_USER_REQUEST:{
        //处理用户查找
        qDebug()<<"ENUM_MSG_TYPE_FIND_USER_REQUEST start";
        //读取pdu中的要查找的用户名
        char caName[32]={'\0'};//初始化字符串数组
        memcpy(caName,pdu->caData,32);
         qDebug()<<"要查找的用户名："<<caName;
        //数据库处理
        int ret = OperateDb::getInstance().handleFindUser(caName);
        qDebug()<<"数据库处理结果 ret:"<<ret;
        //构建pdu并发送
        respdu = mkPDU(0);
        respdu->uiMsgtype =ENUM_MSG_TYPE_FIND_USER_RESPEND;
        memcpy(respdu->caData,caName,32);
        memcpy(respdu->caData+32,&ret,sizeof(bool));
        break;
    }
    case ENUM_MSG_TYPE_ONLINE_USER_REQUEST:
    {
        //在线用户
        qDebug()<<"ENUM_MSG_TYPE_ONLINE_USER_REQUEST start";

        //数据库处理
        QStringList ret = OperateDb::getInstance().handleOnlineUser();
        qDebug()<<"数据库处理结果 ret:"<<ret;
        //构建pdu并发送
        respdu = mkPDU(ret.size()*32);
        respdu->uiMsgtype =ENUM_MSG_TYPE_ONLINE_USER_RESPEND;

        for(int i=0;i<ret.size();i++)
        {
            qDebug()<<"ret.at(i)"<<ret.at(i);
            memcpy(respdu->caMsg+i*32,ret.at(i).toStdString().c_str(),32);
        }
  
        break;
    }
    default:
        qDebug()<<"未处理的消息类型"<<pdu->uiMsgtype;
        break;
    }
    return respdu;
}

void MyTcpSocket::sendPDU(PDU *pdu)
{
    if(pdu == NULL){
        return;
    }
    write((char*)pdu,pdu->uiPDULen);
    free(pdu);
    pdu=NULL;
}
```

消息处理再定义一个类MsgHandler：

```cpp
// mytcpsocket.h
class MyTcpSocket : public QTcpSocket
{
    Q_OBJECT
public:
    MyTcpSocket();
    ~MyTcpSocket();
    MsgHandler * m_pmh;//实例调用处理读取消息中的各个功能函数
}

// mytcpsocket.cpp
MyTcpSocket::MyTcpSocket()
{
    m_pmh = new MsgHandler;//需要释放
}

MyTcpSocket::~MyTcpSocket()
{
    delete m_pmh;
}

//msghandler.h
PDU * regist(PDU * pdu);
PDU * login(PDU * pdu,QString& strLoginName);
PDU * findUser(PDU * pdu);
PDU * onlineUser();

//msghandler.cpp
PDU *MsgHandler::regist(PDU *pdu)
{
    //处理注册
    qDebug()<<"ENUM_MSG_TYPE_REGIST_REQUEST star";
    //读取pdu中的用户名和密码
    char caName[32]={'\0'};//初始化字符串数组
    char caPwd[32]={'\0'};
    memcpy(caName,pdu->caData,32);
    memcpy(caPwd,pdu->caData+32,32);
     qDebug()<<"用户名："<<caName<<" 密码："<<caPwd;
    //数据库处理注册
    bool ret = OperateDb::getInstance().handleRegist(caName,caPwd);
    qDebug()<<"数据库处理结果 ret:"<<ret;
    //构建pdu并发送
    PDU * respdu = mkPDU(0);
    respdu->uiMsgtype =ENUM_MSG_TYPE_REGIST_RESPEND;
    memcpy(respdu->caData,&ret,sizeof(bool));
    return respdu;
}

PDU *MsgHandler::login(PDU *pdu, QString &strLoginName)
{
    //处理登录
    qDebug()<<"ENUM_MSG_TYPE_LOGIN_REQUEST star";
    //读取pdu中的用户名和密码
    char caName[32]={'\0'};//初始化字符串数组
    char caPwd[32]={'\0'};
    memcpy(caName,pdu->caData,32);
    memcpy(caPwd,pdu->caData+32,32);
     qDebug()<<"用户名："<<caName<<" 密码："<<caPwd;
    //数据库处理登录
    bool ret = OperateDb::getInstance().handleLogin(caName,caPwd);
    strLoginName = caName;
    qDebug()<<"数据库处理结果 ret:"<<ret;
    //构建pdu并发送
    PDU * respdu = mkPDU(0);
    respdu->uiMsgtype =ENUM_MSG_TYPE_LOGIN_RESPEND;
    memcpy(respdu->caData,&ret,sizeof(bool));
    return respdu;
}

PDU *MsgHandler::findUser(PDU *pdu)
{
    //处理用户查找
    qDebug()<<"ENUM_MSG_TYPE_FIND_USER_REQUEST start";
    //读取pdu中的要查找的用户名
    char caName[32]={'\0'};//初始化字符串数组
    memcpy(caName,pdu->caData,32);
     qDebug()<<"要查找的用户名："<<caName;
    //数据库处理
    int ret = OperateDb::getInstance().handleFindUser(caName);
    qDebug()<<"数据库处理结果 ret:"<<ret;
    //构建pdu并发送
    PDU * respdu = mkPDU(0);
    respdu->uiMsgtype =ENUM_MSG_TYPE_FIND_USER_RESPEND;
    memcpy(respdu->caData,caName,32);
    memcpy(respdu->caData+32,&ret,sizeof(bool));
    return respdu;
}

PDU *MsgHandler::onlineUser()
{
    //在线用户
    qDebug()<<"ENUM_MSG_TYPE_ONLINE_USER_REQUEST start";
    //数据库处理
    QStringList ret = OperateDb::getInstance().handleOnlineUser();
    qDebug()<<"数据库处理结果 ret:"<<ret;
    //构建pdu并发送
    PDU * respdu = mkPDU(ret.size()*32);
    respdu->uiMsgtype =ENUM_MSG_TYPE_ONLINE_USER_RESPEND;
    for(int i=0;i<ret.size();i++)
    {
        qDebug()<<"ret.at(i)"<<ret.at(i);
        memcpy(respdu->caMsg+i*32,ret.at(i).toStdString().c_str(),32);
    }
    return respdu;
}

// mytcpsocket.cpp
PDU *MyTcpSocket::handleMsg(PDU *pdu)
{
    //根据消息类型进行处理
    switch(pdu->uiMsgtype){
    case ENUM_MSG_TYPE_REGIST_REQUEST:
        return m_pmh->regist(pdu);
    case ENUM_MSG_TYPE_LOGIN_REQUEST:
        return m_pmh->login(pdu,m_strLoginName);
    case ENUM_MSG_TYPE_FIND_USER_REQUEST:
        return m_pmh->findUser(pdu);
    case ENUM_MSG_TYPE_ONLINE_USER_REQUEST:
        return m_pmh->onlineUser();
    default:
        qDebug()<<"未处理的消息类型"<<pdu->uiMsgtype;
        break;
    }
    return NULL;
}
```

#### 客户端
Client::recvMsg()

1. 读取 pdu 的代码封装为一个函数readPDU
2. 根据消息类型处理消息封装一个函数 handleRes
3. 新建一个类 ResHandler，专门对不同消息类型进行处理
4. Client定义一个 ResHandler 指针类型的成员变量，在构造函数和析构函数中 new 和delete

```cpp
// client.h
PDU * readPDU();
PDU * handleMsg(PDU * pdu);

// client.cpp
PDU *Client::readPDU()
{
    qDebug()<<"readPDU 接收消息长度："<<m_tcpSocket.bytesAvailable();
    //读协议长度
    unit uiPDULen = 0;
    m_tcpSocket.read((char*)&uiPDULen,sizeof(unit));//总的协议长度

    //读取协议长度以外的数据
    unit uiMsgLen =uiPDULen - sizeof(unit);
    PDU *pdu = mkPDU(uiMsgLen);
    m_tcpSocket.read((char*)pdu + sizeof(unit), uiPDULen - sizeof(unit));//读取协议总长度以外的数据
    qDebug()<<"readPDU 消息类型"<<pdu->uiMsgtype
            <<" 消息内容："<<pdu->caMsg
            <<" 参数1："<<pdu->caData
            <<" 参数2："<<pdu->caData+32;
    return pdu;
}

//发送pdu
void Client::sendPDU(PDU *pdu)
{
    m_tcpSocket.write((char*)pdu,pdu->uiPDULen);//发送pdu
    free(pdu);
    pdu = NULL;
}
```

传参类型或者数量不对报错：

![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1720598677729-6c8187fd-e1b5-430b-89fc-3374ea1c2083.png)

封装类：

```cpp
// client.h
ResHandler * m_prh;

// client.cpp
m_prh = new ResHandler;//构造函数
delete m_prh ;//析构函数

// reshandler.h
class ResHandler
{
public:
    ResHandler();
    void regist(PDU * pdu);
    void login(PDU * pdu);
    void findUser(PDU * pdu);
    void onlineUser(PDU * pdu);
};
// reshandler.cpp
#include "reshandler.h"
#include "client.h"
#include "index.h"

#include <QMessageBox>
#include <string.h>

ResHandler::ResHandler()
{
    
}

void ResHandler::regist(PDU *pdu)
{
    bool ret;
    memcpy(&ret,pdu->caData,sizeof(bool));
    if(ret){
        QMessageBox::information(&Client::getInstance(),"注册","注册成功");
    }else{
        QMessageBox::information(&Client::getInstance(),"注册","注册失败，用户名或密码非法");
    }
}

void ResHandler::login(PDU *pdu)
{
    bool ret;
    memcpy(&ret,pdu->caData,sizeof(bool));
    if(ret){
        QMessageBox::information(&Client::getInstance(),"登录","登录成功");
        //登陆成功然后才会弹出界面
        Index::getInstance().show();
        //登陆成功隐藏登录页面
        Client::getInstance().hide();
    }else{
        QMessageBox::information(&Client::getInstance(),"登录","登录失败，用户名或密码非法");
    }
}

void ResHandler::findUser(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_LOGIN_FIND_USER_RESPEND start";
    char caName[32]={'\0'};
    memcpy(&caName,pdu->caData,32);
    int ret;
    memcpy(&ret,pdu->caData+32,sizeof(int));
    if(ret == -1){
        QMessageBox::information(&Client::getInstance(),"搜索",QString("%1 不存在").arg(caName));

    }else if(ret == 0){
        QMessageBox::information(&Client::getInstance(),"搜索",QString("%1 不在线").arg(caName));
    }else{
        QMessageBox::information(&Client::getInstance(),"搜索",QString("%1 在线").arg(caName));
    }
}

void ResHandler::onlineUser(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_ONLINE_USER_RESPEND start";
    unit uiSize = pdu->uiMsgLen/32;
    char caTmp[32];
    QStringList userlist;
    for(unit i = 0; i < uiSize ; i++){
        memcpy(caTmp,pdu->caMsg+32*i,32);
        userlist.append(QString(caTmp));
    }
    Index::getInstance().getFriend()->getOnlineUser()->showOnlineUser(userlist);
}

// client.cpp
void Client::recvMsg()
{
    PDU * pdu = readPDU();
    handleMsg(pdu);
}

PDU *Client::readPDU()
{
    qDebug()<<"readPDU 接收消息长度："<<m_tcpSocket.bytesAvailable();
    //读协议长度
    unit uiPDULen = 0;
    m_tcpSocket.read((char*)&uiPDULen,sizeof(unit));//总的协议长度

    //读取协议长度以外的数据
    unit uiMsgLen =uiPDULen - sizeof(unit);
    PDU *pdu = mkPDU(uiMsgLen);
    m_tcpSocket.read((char*)pdu + sizeof(unit), uiPDULen - sizeof(unit));//读取协议总长度以外的数据
    qDebug()<<"readPDU 消息类型"<<pdu->uiMsgtype
            <<" 消息内容："<<pdu->caMsg
            <<" 参数1："<<pdu->caData
            <<" 参数2："<<pdu->caData+32;
    return pdu;
}

void Client::handleMsg(PDU *pdu)
{
    //根据消息类型进行处理
    switch(pdu->uiMsgtype){
    case ENUM_MSG_TYPE_REGIST_RESPEND:
        m_prh->regist(pdu);
        break;
    case ENUM_MSG_TYPE_LOGIN_RESPEND:
        m_prh->login(pdu);
    break;
    case ENUM_MSG_TYPE_FIND_USER_RESPEND:
        m_prh->findUser(pdu);
    break;
    case ENUM_MSG_TYPE_ONLINE_USER_RESPEND:
        m_prh->onlineUser(pdu);
    break;
    default:
        qDebug()<<"未处理的消息类型"<<pdu->uiMsgtype;
        break;
    }
    free(pdu);
    pdu = NULL;
}
```

### 好友功能
#### 添加好友
【注】cur发给服务器一个添加请求，服务器接收请求去数据库查找要添加的人是否在线，返回一个结果给cur,在线的话，服务器发送请求给tar，tar响应给服务器同意，同时服务器给客户端发送同意请求

两个客户端，当前用户客户端(cur)和目标用户客户端(tar)

+ cur 向服务器发送添加好友请求
    - 在线用户界面双击当前项转到槽函数
    - 槽函数中，获取当前用户名和目标用户名
    - 构建 pdu，发送给服务器
+ 服务器査询 tar 是否满足添加好友条件，结果响应给 cur，如果满足转发给 tar
    - MyTcpServer 类定义转发函数，参数为目标用户名和 pdu，遍历socket列表找到 tar并发送
    - 数据库操作类增加处理添加好友函数，查询是否已是好友和是否在线
    - 调用添加好友函数，根据返回值调用转发函数转发给 tar
    - 结果响应给 cur
+ cur 显示服务器的响应结果
+ tar创建弹窗，询问用户是否同意添加好友，如果同意发送给服务器同意添加好友的请求
+ 服务器处理同意添加好友的请求
    - 数据库操作类增加同意添加好友函数
    - 调用该函数
    - 结果响应给 tar，结果转发给 cur
+ cur和 tar显示添加成功

cur 向服务器发送添加好友请求：

```cpp
// client.cpp
void OnlineUser::on_listWidget_itemDoubleClicked(QListWidgetItem *item)
{
    QString strCurName = Client::getInstance().m_strLoginName;
    QString strTarName = item->text();
}

//添加好友
ENUM_MSG_TYPE_ADD_FRIEND_REQUEST,
ENUM_MSG_TYPE_ADD_FRIEND_RESPEND,

// onlineuser.cpp
void OnlineUser::on_listWidget_itemDoubleClicked(QListWidgetItem *item)
{
    QString strCurName = Client::getInstance().m_strLoginName;
    QString strTarName = item->text();
    PDU * pdu = mkPDU(0);
    memcpy(pdu->caData,strCurName.toStdString().c_str(),32);
    memcpy(pdu->caData+32,strTarName.toStdString().c_str(),32);
    Client::getInstance().sendPDU(pdu);
}
```

服务器：

```cpp
void MyTcpServer::resend(char *tarName, PDU *pdu)
{
    if(tarName == NULL || pdu == NULL){
        return;
    }
    for(int i=0;i<m_tcpSocketList.size();i++){
        if(tarName == m_tcpSocketList.at(i)->m_strLoginName){
            m_tcpSocketList.at(i)->write((char*)pdu,pdu->uiMsgLen);
            break;
        }
    }
}


int OperateDb::handleAddFriend(const char *curName, const char *tarName)
{
    if(curName == NULL || tarName == NULL){
        return -1; 
    }
    QString sql =QString( "R(select * from friend where"
                          "(user_id =(select id from user_info where name = '%1')"
                          "and "
                          "friend_id = (select id from user_info where name = '%2'))"
                          "or"
                          "(friend_id = (select id from 
user_info where name = '%3')"
                          " and"
                          "user_id = (select id from user_info where name = '%4')"
                          ");)").arg(curName).arg(tarName).arg(curName).arg(tarName);
    qDebug()<<"是否是好友sql : "<<sql;
    QSqlQuery q;
    q.exec(sql);
    if(q.next()){
        return -2;//已经是好友
    }
    sql = QString("select online from user_info where name = '%1'").arg(tarName);
    q.exec(sql);
    qDebug()<<"查找要添加的好友是否在线sql"<<sql;
    if(q.next()){
        return q.value(0).toInt();//1在线，0不在线

    }
    return -1;
}

PDU *MsgHandler::addFriend(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_ADD_FRIEND_REQUEST start";
    char curName[32]={'\0'};
    char tarNmae[32]={'\0'};
    //读取pdu中当前用户和目标用户
    memcpy(curName,pdu->caData,32);
    memcpy(tarNmae,pdu->caData+32,32);
    int ret = OperateDb::getInstance().handleAddFriend(curName,tarNmae);
    qDebug()<<"数据库处理结果 ret:"<<ret;

    if(ret == 1){
        MyTcpServer::getInstance().resend(tarNmae,pdu);
    }
    //构建pdu并发送
    PDU * respdu = mkPDU(0);
    respdu->uiMsgtype =ENUM_MSG_TYPE_ADD_FRIEND_RESPEND;
    memcpy(respdu->caData,&ret,sizeof(int));
    return respdu;
}
case ENUM_MSG_TYPE_ADD_FRIEND_REQUEST:
    return m_pmh->addFriend(pdu);
```

客户端：

```cpp
//同意添加好友
ENUM_MSG_TYPE_ADD_FRIEND_AGREE_REQUEST,
ENUM_MSG_TYPE_ADD_FRIEND_AGREE_RESPEND,

void OnlineUser::on_listWidget_itemDoubleClicked(QListWidgetItem *item)
{
    qDebug()<<"on_listWidget_itemDoubleClicked start";
    QString strCurName = Client::getInstance().m_strLoginName;
    QString strTarName = item->text();
    qDebug()<<"strCurName"<<strCurName
           <<"strTarName"<<strTarName;
    PDU * pdu = mkPDU(0);
    pdu->uiMsgtype = ENUM_MSG_TYPE_ADD_FRIEND_REQUEST;
    memcpy(pdu->caData,strCurName.toStdString().c_str(),32);
    memcpy(pdu->caData+32,strTarName.toStdString().c_str(),32);
    Client::getInstance().sendPDU(pdu);
}

case ENUM_MSG_TYPE_ADD_FRIEND_REQUEST:
        m_prh->addFriendRequest(pdu);
        break;

void ResHandler::addFriendRequest(PDU *pdu)
{
    char caName={'\0'};
    memcpy(&caName,pdu->caData,32);
    int ret = QMessageBox::question(Index::getInstance().getFriend(),"添加好友请求",QString("是否同意 '%1' 的添加好友请求").arg(caName));
    if(ret != QMessageBox::Yes){
        return;
    }
    PDU * respdu = mkPDU(0);
    respdu->uiMsgtype = ENUM_MSG_TYPE_ADD_FRIEND_AGREE_REQUEST;
    memcpy(respdu->caData,pdu->caData,64);
    Client::getInstance().sendPDU(respdu);
}

void Client::sendPDU(PDU *pdu)
{
    qDebug()<<"sendPDU caData:"<<pdu->caData
            <<"caData+32:"<<pdu->caData+32
            <<"uiMsgtype:"<<pdu->uiMsgtype
            <<"caMsg:"<<pdu->caMsg;
    m_tcpSocket.write((char*)pdu,pdu->uiPDULen);//发送pdu
    free(pdu);
    pdu = NULL;
}

case ENUM_MSG_TYPE_ADD_FRIEND_REQUEST:
        return m_pmh->addFriend(pdu);
PDU *MsgHandler::addFriend(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_ADD_FRIEND_REQUEST start";
    char curName[32]={'\0'};
    char tarNmae[32]={'\0'};
    //读取pdu中当前用户和目标用户
    memcpy(curName,pdu->caData,32);
    memcpy(tarNmae,pdu->caData+32,32);
    int ret = OperateDb::getInstance().handleAddFriend(curName,tarNmae);
    qDebug()<<"数据库处理结果 ret:"<<ret;
    if(ret == 1){
        MyTcpServer::getInstance().resend(tarNmae,pdu);
    }
    //构建pdu并发送
    PDU * respdu = mkPDU(0);
    respdu->uiMsgtype =ENUM_MSG_TYPE_ADD_FRIEND_RESPEND;
    memcpy(respdu->caData,&ret,sizeof(int));
    return respdu;
}

case ENUM_MSG_TYPE_ADD_FRIEND_AGREE_RESPEND:
        m_prh->addFriendRequestAgree(pdu);
        break;

void ResHandler::addFriendRequestAgree(PDU *pdu)
{
    int ret;
    memcpy(&ret,pdu->caData,sizeof(int));
    if(ret == 0){
        QMessageBox::information(Index::getInstance().getFriend(),"添加好友","添加失败");
    }else {
         QMessageBox::information(Index::getInstance().getFriend(),"添加好友","成功");
    }
}
```

#### 查找用户时添加好友
查找用户时，添加好友功能:在处理查找好友的响应时发送添加好友请求

```cpp
void ResHandler::findUser(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_LOGIN_FIND_USER_RESPEND start";
    char caName[32]={'\0'};
    memcpy(caName,pdu->caData,32);
    int ret;
    memcpy(&ret,pdu->caData+32,sizeof(int));
    if(ret == 1){
        int ret = QMessageBox::information(Index::getInstance().getFriend(),"搜索",QString("%1 在线").arg(caName),"添加好友","取消");
        if(ret == 0){
            PDU * pdu = mkPDU(0);
            pdu->uiMsgtype = ENUM_MSG_TYPE_ADD_FRIEND_REQUEST;
            memcpy(pdu->caData,Client::getInstance().m_strLoginName.toStdString().c_str(),32);
            memcpy(pdu->caData+32,caName,32);
            Client::getInstance().sendPDU(pdu);
        }
    }else if(ret == 0){
        QMessageBox::information(Index::getInstance().getFriend(),"搜索",QString("%1 不在线").arg(caName));
    }else{
        QMessageBox::information(Index::getInstance().getFriend(),"搜索",QString("%1 不存在").arg(caName));
    }
}
```

#### 刷新在线好友
+ 向服务器发送请求
    - 刷新好友封装函数，发送当前用户名
    - 刷新好友按钮的槽函数(添加好友成功、Friend 的构造函数)调用该函数
+ 服务器获取在线好友并返回给客户端
    - 数据库操作类新增获取在线好友函数，注意当前用户是 user id 还可以是 friend id返回好友名组成的 QStringList
    - 调用该函数，结果发送客户端
+ 客户端显示数据
    - Friend 定义函数，讲数据填入列表框中
    - 消息处理处调用该函数

【注】<font style="color:rgb(71, 101, 130);">UNION</font><font style="color:rgb(36, 41, 47);background-color:rgb(248, 248, 248);">的作用是合并两个子查询的结果</font>

```cpp
//刷新好友
    ENUM_MSG_TYPE_ADD_FLUSH_FRIEND_REQUEST,
    ENUM_MSG_TYPE_ADD_FLUSH_FRIEND_RESPEND,

void OnlineUser::on_listWidget_itemDoubleClicked(QListWidgetItem *item)
{
    qDebug()<<"on_listWidget_itemDoubleClicked start";
    QString strCurName = Client::getInstance().m_strLoginName;
    QString strTarName = item->text();
    qDebug()<<"strCurName"<<strCurName
           <<"strTarName"<<strTarName;
    PDU * pdu = mkPDU(0);
    pdu->uiMsgtype = ENUM_MSG_TYPE_ADD_FRIEND_REQUEST;
    memcpy(pdu->caData,strCurName.toStdString().c_str(),32);
    memcpy(pdu->caData+32,strTarName.toStdString().c_str(),32);
    Client::getInstance().sendPDU(pdu);
}

 case ENUM_MSG_TYPE_ADD_FLUSH_FRIEND_REQUEST:
        return m_pmh->flushFriend(pdu);

PDU *MsgHandler::flushFriend(PDU *pdu)
{
    //刷新好友
    qDebug()<<"ENUM_MSG_TYPE_ADD_FLUSH_FRIEND_REQUES start";
    QStringList ret = OperateDb::getInstance().handleFlushFriend(pdu->caData);
    qDebug()<<"数据库处理结果 ret:"<<ret;
    PDU * respdu = mkPDU(ret.size()*32);
    respdu->uiMsgtype =ENUM_MSG_TYPE_ADD_FLUSH_FRIEND_RESPEND;
    for(int i=0;i<ret.size();i++){
        qDebug()<<"ret.at(i)"<<ret.at(i);
        memcpy(respdu->caMsg+i*32,ret.at(i).toStdString().c_str(),32);
    }
    return respdu;
}

case ENUM_MSG_TYPE_ADD_FLUSH_FRIEND_RESPEND:
        m_prh->onlineFriend(pdu);
        break;

void ResHandler::onlineFriend(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_FLUSH_FRIEND_RESPEND start";
    int size = pdu->uiMsgLen/32;
    QStringList friendList;
    char caTmp[32] = {'\0'};
    for(int i=0;i<size;i++){
        memcpy(caTmp,pdu->caMsg+i*32,32);
        friendList.append(QString(caTmp));
        qDebug()<<"caTmp"<<QString(caTmp);
    }
    Index::getInstance().getFriend()->showOnlineFriend(friendList);
}

void Friend::showOnlineFriend(QStringList friendList)
{
    ui->listWidget->clear();//不清空的话下线不会删除
    ui->listWidget->addItems(friendList);
}

```

#### 删除好友
+ 客户端发送请求
    - 获取要删除的用户名，弹窗让用户再次确认删除
    - 当前用户名和要删除用户名放入 pdu，发送请求
+ 服务器处理并响应
    - 数据库操作类增加删除好友函数，先判断是否是好友
    - 消息处理类中调用该函数，结果发给客户端
+ 客户端显示结果，刷新好友列表

```cpp
//删除好友
    ENUM_MSG_TYPE_ADD_DELETE_FRIEND_REQUEST,
    ENUM_MSG_TYPE_ADD_DELETE_FRIEND_RESPEND,

PDU *MsgHandler::deleteFriend(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_DELETE_FRIEND_REQUES start";
    char curName[32]={'\0'};
    char tarNmae[32]={'\0'};
    //读取pdu中当前用户和目标用户
    memcpy(curName,pdu->caData,32);
    memcpy(tarNmae,pdu->caData+32,32);
    int ret = OperateDb::getInstance().handleDelFriend(curName,tarNmae);
    qDebug()<<"数据库处理结果 ret:"<<ret;
    //构建pdu并发送
    PDU * respdu = mkPDU(0);
    respdu->uiMsgtype =ENUM_MSG_TYPE_DELETE_FRIEND_RESPEND;
    memcpy(respdu->caData,&ret,sizeof(bool));
    return respdu;
}

bool OperateDb::handleDelFriend(const char *curName, const char *tarName)
{
    qDebug()<<"handleDelFriend start";
    if(curName == NULL || tarName == NULL){
        return -1;
    }
    QString sql =QString( R"(
                          select * from friend where
                          (user_id =(select id from user_info where name = '%1')
                          and
                          friend_id = (select id from user_info where name = '%2')
                           )
                          or
                          (friend_id = (select id from user_info where name = '%3')
                           and
                          user_id = (select id from user_info where name = '%4')
                          );
                          )").arg(curName).arg(tarName).arg(curName).arg(tarName);
    qDebug()<<"是否是好友sql : "<<sql;
    QSqlQuery q;
    q.exec(sql);
    if(!q.next()){
        return false;//已经不是好友
    }
    sql =QString( R"(
                  delete from friend where
                  (user_id =(select id from user_info where name = '%1')
                  and
                  friend_id = (select id from user_info where name = '%2')
                   )
                  or
                  (friend_id = (select id from user_info where name = '%3')
                   and
                  user_id = (select id from user_info where name = '%4')
                  );
                  )").arg(curName).arg(tarName).arg(curName).arg(tarName);
    q.exec(sql);
    qDebug()<<"删除的好友sql"<<sql;
    return true;
}

case ENUM_MSG_TYPE_DELETE_FRIEND_RESPEND:
    m_prh->deleteFriend(pdu);
    break;
void ResHandler::deleteFriend(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_DELETE_FRIEND_RESPEND start";
    bool ret;
    memcpy(&ret,pdu->caData,sizeof(int));
    if(!ret){
        QMessageBox::information(Index::getInstance().getFriend(),"删除好友","删除失败");
    }else {
         QMessageBox::information(Index::getInstance().getFriend(),"删除好友","对方已不是好友");
    }
    Index::getInstance().getFriend()->flushFriend();
}
```

### 聊天功能
#### UI界面
新建一个chat设计师界面类![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1720746319670-5dbf2006-4867-4077-bf37-8b6a97704d73.png)

#### 显示界面
1. 好友界面类中创建一个聊天界面类指针的属性，构造函数和析构函数中创建和释放
2. 聊天按钮槽函数，判断是否在好友列表中选择了好友
3. 显示聊天界面

聊天界面类中创建一个聊天好友名的属性，槽函数中将选中的好友名赋值给该属性

#### 聊天接口
+ 客户端发送
    - 发送按钮的槽函数，获取输入框的内容
    - 将聊天内容和双方用户名发送给服务器
+ 服务器转发消息:提取目标用户名，消息转发给目标用户
+ 客户端显示内容
    - 如果聊天框没有显示，显示聊天窗口
    - 在 Chat 类里新建一个函数，更新聊天内容框
    - 更新 Chat 的聊天好友名属性

> 发送的消息的内容存给chat 到chat中定义成员变量m_strChatName(跟谁聊天)
>

客户端：

```cpp
QString m_strChatName;//跟谁聊天
m_pChat = new Chat;

void Friend::on_chat_PB_clicked()
{
    QListWidgetItem * pItem = ui->listWidget->currentItem();
    if(!pItem){
        QMessageBox::information(this,"聊天","请选择想要聊天的好友");
        return;
    }
    if(m_pChat->isHidden()){
        m_pChat->show();
    }
    m_pChat->m_strChatName = pItem->text();//好友名
}


void Chat::on_send_PB_clicked()
{
    //获取发送的消息
    QString strMsg = ui->input_LE->text();
    if(strMsg.isEmpty()){
        return;
    }
    PDU * pdu = mkPDU(sizeof(strMsg));
    pdu->uiMsgtype = ENUM_MSG_TYPE_CHAT_REQUEST;
    //发送的消息放入caMsg
    memcpy(pdu->caMsg,strMsg.toStdString().c_str(),strMsg.toStdString().size());
    //发送者名和发送目标放入caDate]
    memcpy(pdu->caData,Client::getInstance().m_strLoginName.toStdString().c_str(),32);
    memcpy(pdu->caData+32,m_strChatName.toStdString().c_str(),32);
    Client::getInstance().sendPDU(pdu);
    ui->input_LE->clearMask();//输入后清空输入框
}
```

服务器（不需要数据库处理）

```cpp
void MsgHandler::chat(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_CHAT_REQUES start";
    char tarName[32] ={'\0'};
    memcpy(tarName,pdu->caData+32,32);
    MyTcpServer::getInstance().resend(tarName,pdu);
}
```

### 文件系统
#### 创建文件夹
+ 用户目录的初始化
    - 清空好友表和用户表(或写脚本创建已有用户的目录)
    - 在项目服务器的 build 目录创建一个文件夹 filesys，用于存用户文件
    - 服务器和客户端新增了一个配置:./filesys，作为服务器和客户端的成员变量存下来(Server 新增单例模式)
    - 注册时服务器在./filesys 目录下新建一个用户同名的目录
    - File 类定义当前路径和用户路径属性，构造函数里初始化
+ 客户端向服务器发送消息
    - 创建一个输入框，得到用户输入的文件名
    - 文件夹名、当前路径发送给服务器
+ 服务器接收并处理:
    - 检查当前路径是否存在
    - 拼接路径和文件夹名，检查完整路径是否存在
    - 创建文件夹并响应
+ 客户端显示响应结果



创建文件夹是在服务器，当每注册一个用户，给这个用户一个单独的文件夹：

![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1720762651528-c55a6990-7ab3-47b3-8f7f-45a33a157545.png)

讲文件夹名字存到：配置文件中加一行

```cpp
./filesys  （./是当前目录下）
```

获取

```cpp
QString m_strRootPath;

m_strRootPath = strList.at(2);//下标获取
```

注册新建对应用户的文件夹：

```cpp
Server &Server::getInstance()
{
    static Server instance;
    return instance;
}

Server(const Server& instance) = delete;
Server& operator = (const Server&) = delete;

QString Server::getRootPath()
{
    return m_strRootPath;
}

PDU *MsgHandler::login(PDU *pdu, QString &strLoginName){
    。。。
if(ret){
        QDir dir;
        dir.mkdir(QString("%1/%2").arg(Server::getInstance().getRootPath()).arg(caName));
    }
}
```

简单设置ui

![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1720764890011-21448e2e-c341-46ac-8128-591f93ee9a50.png)

客户端

```cpp
void File::on_mkDir_PB_clicked()
{
    QString strNewDir = QInputDialog::getText(this,"新建文件夹","文件夹名:");
    if(strNewDir.isEmpty() || strNewDir.toStdString().size()>32){
        QMessageBox::warning(this,"新建文件夹","文件夹名字长度非法");
        return;
    }
    //文件夹名字放入caData 当前路径放在caMag
    PDU * pdu = mkPDU(m_strCurPath.toStdString().size());
    pdu->uiMsgtype =ENUM_MSG_TYPE_MKDIR_REQUEST;
    memcpy(pdu->caData,strNewDir.toStdString().c_str(),32);
    memcpy(pdu->caMsg,m_strCurPath.toStdString().c_str(),m_strCurPath.toStdString().size());
    Client::getInstance().sendPDU(pdu);
}

void ResHandler::mkdir(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_MKDIR_RESPEND start";
    bool ret;
//    qDebug()<<"ret"<<ret;
    memcpy(&ret,pdu->caData,sizeof(bool));
    if(ret){
        QMessageBox::information(Index::getInstance().getFile(),"创建文件夹","创建成功");
    }else {
        QMessageBox::information(Index::getInstance().getFile(),"创建文件夹","创建失败");
    }
}
```

服务器

```cpp
PDU *MsgHandler::mkdir(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_MKDIR_REQUES start";
    QString strCurPath = pdu->caMsg;
    PDU * respdu = mkPDU(0);
    respdu->uiMsgtype=ENUM_MSG_TYPE_MKDIR_RESPEND;
    bool res = false;
    QDir dir;
    // if -else -if
    if(!dir.exists(strCurPath)){// 检查当前路径是否存在
        memcpy(respdu->caData,&res,sizeof (bool));
        return respdu;
    }
    QString strNewPath = QString("%1/%2").arg(strCurPath).arg(pdu->caData);
    qDebug()<<"mkdir strNewPath"<<strNewPath;
    if(dir.exists(strNewPath)||!dir.mkdir(strNewPath)){//检查新路径是否已存在或尝试创建新路径
        memcpy(respdu->caData,&res,sizeof (bool));
        return respdu;
    }
    //成功创建
    res = true;
    memcpy(respdu->caData,&res,sizeof (bool));
    return respdu;
}


```

#### 刷新文件
+ 客户端发送请求给服务器
    - 刷新文件封装函数（好多地方用到），传当前路径
    - 刷新文件按钮的槽函数和 Flle 的构造函数，创建文件夹中调用刷新文件函数
+ 服务器处理请求
    - 协议中创建文件信息结构体 Filelnfo，包含文件名和文件类型
    - 获取指定路径下的文件
    - 发送给客户端
+ 客户端显示
    - 添加 Qt 资源文件，文件和文件夹的icon 放入资源文件中
    - File 定义一个 QList<Filelnfo”>，存当前路径下的文件信息,
    - File 类定义更新文件列表的函数，清空并更新文件列表框和当前路径的文件信息
    - 接收消息处调用该函数

客户端

```cpp
struct FileInfo{//存储文件属性
    char caName[32];//文件名
    int iFileType;//文件类型
};

void File::updateFileList(QList<FileInfo *> pFileList)
{
    //释放binggengxinm_fileList
    foreach(FileInfo*pFileInfo,m_fileList){
        delete pFileInfo;
    }
    m_fileList.clear();
    m_fileList = pFileList;

    //清空并更新列表框
    ui->listWidget->clear();
    foreach(FileInfo *pFileInfo,pFileList){
        QListWidgetItem * pItem = new QListWidgetItem;//框中的每一个元素
        if(pFileInfo->iFileType==0){
            pItem->setIcon((QIcon(QPixmap(":/dir.png"))));
        }else if(pFileInfo->iFileType==1){
            pItem->setIcon((QIcon(QPixmap(":/file.png"))));
        }
        pItem->setText(pFileInfo->caName);
        qDebug()<<"pFileInfo->caName"<<pFileInfo->caName;
        ui->listWidget->addItem(pItem);//显示到界面
    }
}

void ResHandler::flushFile(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_FLUSH_FILE_RESPEND start";
    int iCount = pdu->uiMsgLen/sizeof (FileInfo);//文件个数
//    qDebug()<<"iCount"<<iCount;
    QList<FileInfo*> pFileList;
    for(int i=0;i<iCount;i++){
        FileInfo *pFileInfo = new FileInfo;
        memcpy(pFileInfo,pdu->caMsg+i*sizeof (FileInfo),sizeof (FileInfo));
        pFileList.append(pFileInfo);
    }
    Index::getInstance().getFile()->updateFileList(pFileList);
}

```

服务器

```cpp
struct FileInfo{//存储文件属性
    char caName[32];//文件名
    int iFileType;//文件类型
};

PDU *MsgHandler::flushFile(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_FLUSH_FILE_REQUEST start";
    QString strPath = pdu->caMsg;
    QDir dir(strPath);//传入路径
    QFileInfoList fileInfoList = dir.entryInfoList();//获取目录下的全部文件文件夹
    int iFileCount = fileInfoList.size();

//    qDebug()<<"iFileCount"<<iFileCount;
    PDU *respdu = mkPDU(sizeof(FileInfo)*(iFileCount-2));
    respdu->uiMsgtype = ENUM_MSG_TYPE_FLUSH_FILE_RESPEND;
    FileInfo * pFileInfo =NULL;//定义指针指向结构体
    QString strFileName;
    for(int i = 0,j=0;i<iFileCount;i++){
        strFileName = fileInfoList[i].fileName();
        if(strFileName == QString(".")||strFileName == QString("..")){
            continue;
        }
        pFileInfo = (FileInfo *)(respdu->caMsg)+j++;
        memcpy(pFileInfo->caName,strFileName.toStdString().c_str(),32);
        if(fileInfoList[i].isDir()){
            pFileInfo->iFileType =0;//目录
        }else if(fileInfoList[i].isFile()){
            pFileInfo->iFileType =1;//文件
        }
        qDebug()<<"flushFIle strFileName"<<pFileInfo->caName
               <<"iFileType"<<pFileInfo->iFileType;
    }
//    qDebug()<<"respdu->uiMsg"<<respdu->uiMsgLen/sizeof(FileInfo);
    return respdu;
}

```

以结构体的方式放入caMsg,转成FileInfo类型的指针，往pFIleInfo指针里存就相当于往地址里存（caMsg里存

） ,+i是指针偏移下一块地址，

QListWidgeItem代表的是列表框中的一个个元素

QPixmap用来打开图片 转成Icon格式，setIcon是放进去

icount--文件个数

> 加上登陆成功刷新文件，添加文件成功刷新文件，会导致部分功能不好使？
>
> 原因是客户端发送多个消息，客户端不一定能全部接收到，本项目这部分是因为客户端同时发送两个信号而服务器只接收到一个只处理了一个
>

#### 处理粘包半包
+ MyTcpSocket定义一个成员变量 QByteArray 类型的 buffer，存 socket 中的数据
+ readAll 读取 socket 中的全部数据
+ 读取的数据 append 到 buffer里
+ 循环处理 buffer，循环条件为 buffer 的大小是否大于等于 PDU 的大小:为了能构建一个完整 PDU，读取协议长度
+ 构建 PDU 指针指向 buffer，判断 buffer 的大小是否大于等于协议长度
+ 调用处理消息函数
+ remove 掉 buffer 中已处理的协议长度

解决粘包（发多个消息没有全接收）、半包（没有发完一条消息）问题：

```cpp
服务器：
QByteArray buffer;//当前所有数据

void MyTcpSocket::recvMsg()
{
    qDebug()<<"\n\n\nrecvMsg 接收消息长度："<<m_tcpSocket.bytesAvailable();
    //解决粘包，半包问题
    QByteArray data = readAll();
    buffer.append(data);
    
    while(buffer.size()>=int(sizeof(PDU))){//判断是否是一个完整的PDU，为了能取PDU中的协议长度 
        PDU *pdu =(PDU*)buffer.data();
        if(buffer.size()<int(pdu->uiPDULen)){//判断能否构成一个完整的协议请求（PDU+柔性数组）
            break;
        }
        PDU * respdu = handleMsg(pdu);//读取消息
        sendPDU(respdu);//发送结果
       buffer.remove(0,pdu->uiPDULen);//释放
    }
}

客户端：
QByteArray buffer;//当前所有数据

void Client::recvMsg()
{
     qDebug()<<"\n\n\nrecvMsg 接收消息长度："<<m_tcpSocket.bytesAvailable();
    //解决粘包，半包问题
    QByteArray data = m_tcpSocket.readAll();
    buffer.append(data);//buffer--当前所有数据

    while(buffer.size()>=int(sizeof(PDU))){//判断是否是一个完整的PDU，为了能取PDU中的协议长度
        PDU *pdu =(PDU*)buffer.data();
        if(buffer.size()<int(pdu->uiPDULen)){//判断能否构成一个完整的协议请求（PDU+柔性数组）
            break;
        }
        handleMsg(pdu);//读取消息
        buffer.remove(0,pdu->uiPDULen);//释放
    }
}

```

#### 删除文件夹/文件
+ 客户端发送请求:当前路径和选中的文件夹拼接后并发送给服务器
+ 服务器处理请求:删除指定文件夹(删除文件使用dir.remove(path))
+ 客户端显示结果

客户端：

```cpp
void File::on_delDir_PB_clicked()
{
    //获取选择的文件夹名
   QListWidgetItem * pItem= ui->listWidget->currentItem();
   if(pItem == NULL){
       QMessageBox::warning(this,"删除文件夹","请选择要删除的文件夹");
       return;
   }
   QString strDelFileName = pItem->text();
    //完判断选择是否为文件夹
   foreach(FileInfo * pFileInfo,m_fileList){
       if(pFileInfo->caName ==strDelFileName && pFileInfo->iFileType !=0){
           QMessageBox::warning(this,"删除文件夹","选择的不是文件夹");
           return;
       }
   }
   //确认是否删除
   int ret =QMessageBox::question(this,"删除文件夹",QString("是否确定删除文件夹：%1").arg(strDelFileName));
   if(ret!= QMessageBox::Yes){
       return;
   }
   //完整发送路径给服务器
   QString strPath =m_strCurPath +"/"+strDelFileName;
   PDU * pdu = mkPDU(strPath.toStdString().size()+1);
   pdu->uiMsgtype =ENUM_MSG_TYPE_DEL_DIR_REQUEST;
   memcpy(pdu->caMsg,strPath.toStdString().c_str(),strPath.toStdString().size());
   Client::getInstance().sendPDU(pdu);
}

void File::on_delFile_PB_clicked()
{
    //获取选择的文件名
   QListWidgetItem * pItem= ui->listWidget->currentItem();
   if(pItem == NULL){
       QMessageBox::warning(this,"删除文件","请选择要删除的文件");
       return;
   }
   QString strDelFileName = pItem->text();
    //完判断选择是否为文件
   foreach(FileInfo * pFileInfo,m_fileList){
       if(pFileInfo->caName ==strDelFileName && pFileInfo->iFileType !=1){
           QMessageBox::warning(this,"删除文件","选择的不是文件");
           return;
       }
   }
   //确认是否删除
   int ret =QMessageBox::question(this,"删除文件",QString("是否确定删除文件：%1").arg(strDelFileName));
   if(ret!= QMessageBox::Yes){
       return;
   }
   //完整发送路径给服务器
   QString strPath =m_strCurPath +"/"+strDelFileName;
   PDU * pdu = mkPDU(strPath.toStdString().size()+1);
   pdu->uiMsgtype =ENUM_MSG_TYPE_DEL_FILE_REQUEST;
   memcpy(pdu->caMsg,strPath.toStdString().c_str(),strPath.toStdString().size());
   Client::getInstance().sendPDU(pdu);
}

void ResHandler::delDir(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_DEL_DIR_RESPEND start";
    bool ret;
//    qDebug()<<"ret"<<ret;
    memcpy(&ret,pdu->caData,sizeof(bool));
    if(ret){
        QMessageBox::information(Index::getInstance().getFile(),"删除文件夹","删除成功");
        Index::getInstance().getFile()->flushFile();
    }else {
        QMessageBox::information(Index::getInstance().getFile(),"删除文件夹","删除失败");
    }
}

void ResHandler::delFile(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_DEL_FILE_RESPEND start";
    bool ret;
//    qDebug()<<"ret"<<ret;
    memcpy(&ret,pdu->caData,sizeof(bool));
    if(ret){
        QMessageBox::information(Index::getInstance().getFile(),"删除文件","删除成功");
        Index::getInstance().getFile()->flushFile();
    }else {
        QMessageBox::information(Index::getInstance().getFile(),"删除文件","删除失败");
    }
}

```

服务器

```cpp
PDU *MsgHandler::delDir(PDU *pdu)
{
    QFileInfo fileInfo(pdu->caMsg);
    bool ret;
    if(fileInfo.isDir()){
        QDir dir(pdu->caMsg);
        ret = dir.removeRecursively();
    }
    PDU *respdu = mkPDU(0);
    respdu->uiMsgtype = ENUM_MSG_TYPE_DEL_DIR_RESPEND;
    memcpy(respdu->caData,&ret,sizeof (bool));
    return respdu;
}

PDU *MsgHandler::delFile(PDU *pdu)
{
    QFileInfo fileInfo(pdu->caMsg);
    bool ret;
    if(fileInfo.isFile()){
        QFile file(pdu->caMsg);
        ret = file.remove();
    }
    PDU *respdu = mkPDU(0);
    respdu->uiMsgtype = ENUM_MSG_TYPE_DEL_FILE_RESPEND;
    memcpy(respdu->caData,&ret,sizeof (bool));
    return respdu;
}
```

#### 重命名文件
+ 客户端发送请求:当前文件路径和要修改的文件路径并发送给服务器
+ 服务器处理请求:重命名指定文件(重命名文件使用dir.rename(oldpath,newpath))
+ 客户端显示结果

客户端

```cpp
void File::on_rename_PB_clicked()
{
    //获取要重命名的文件名
    QListWidgetItem * pItem= ui->listWidget->currentItem();
    if(pItem == NULL){
        QMessageBox::warning(this,"重命名文件","请选择要重命名的文件");
        return;
    }
    QString strOldFileName = pItem->text();
    //获取要修改的文件名
    QString strNewFileName = QInputDialog::getText(this,"重命名文件","文件夹名:");
    if(strNewFileName.isEmpty() || strNewFileName.toStdString().size()>32){
        QMessageBox::warning(this,"重命名文件","文件夹名字长度非法");
        return;
    }
    //新旧文件名字caData 当前路径放在caMag
    PDU * pdu = mkPDU(m_strCurPath.toStdString().size());
    pdu->uiMsgtype =ENUM_MSG_TYPE_RENAME_FILE_REQUEST;
    memcpy(pdu->caData,strOldFileName.toStdString().c_str(),32);
    memcpy(pdu->caData+32,strNewFileName.toStdString().c_str(),32);
    memcpy(pdu->caMsg,m_strCurPath.toStdString().c_str(),m_strCurPath.toStdString().size());
    Client::getInstance().sendPDU(pdu);
}

void ResHandler::renameFile(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_RENAME_FILE_RESPEND start";
    bool ret;
//    qDebug()<<"ret"<<ret;
    memcpy(&ret,pdu->caData,sizeof(bool));
    if(ret){
        Index::getInstance().getFile()->flushFile();
    }else {
        QMessageBox::information(Index::getInstance().getFile(),"重命名文件","重命名文件失败");
    }
}

```

服务器

```cpp
PDU *MsgHandler::renameFile(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_RENAME_FILE_REQUEST start";
    //获取原文件名和新文件名
    char caOldFileName[32] = {'\0'};
    char caNewFileName[32] = {'\0'};
    memcpy(caOldFileName,pdu->caData,32);
    memcpy(caNewFileName,pdu->caData+32,32);
    //取当前路径并拼接完整路径
    char * pPath = pdu->caMsg;
    QString strOldPath = QString("%1/%2").arg(pPath).arg(caOldFileName);
    QString strNewPath = QString("%1/%2").arg(pPath).arg(caNewFileName);
//    qDebug()<<"strOldPath"<<strOldPath;
//    qDebug()<<"strNewPath"<<strOldPath;
    //使用QDir对文件重命名
    QDir dir;
    bool res =dir.rename(strOldPath,strNewPath);
    PDU * respdu = mkPDU(0);
    respdu->uiMsgtype=ENUM_MSG_TYPE_RENAME_FILE_RESPEND;
    memcpy(respdu->caData,&res,sizeof (bool));
    return respdu;
}
```

#### 进入文件夹/返回上一级
只在客户端操作

判断是否为文件夹，更新当前目录，调用刷新函数

```cpp
void File::on_listWidget_itemDoubleClicked(QListWidgetItem *item)
{
    QString strDirName = item->text();
    //判断选择是否为文件夹
    foreach(FileInfo * pFileInfo,m_fileList){
        if(pFileInfo->caName ==strDirName && pFileInfo->iFileType !=0){
            QMessageBox::warning(this,"提示","选择的不是文件夹");
            return;
        }
    }
    m_strCurPath = m_strCurPath +'/' +strDirName;
    flushFile();
}
```

判断是否为用户顶层目录，移除最后一层目录，更新当前路径，调用刷新文件函数

```cpp
void File::on_return_PB_clicked()
{
    if(m_strCurPath == m_strUserPath){
        QMessageBox::warning(this,"返回","返回失败，已在顶层目录");
        return;
    }
    int index = m_strCurPath.lastIndexOf('/');
    m_strCurPath.remove(index,m_strCurPath.size() - index);//从哪开始移除移除多少
    flushFile();
}
```

移动文件

+ 点击“移动文件”，保存当前路径和选中的文件名
+ 提示用户选择要移动到的目录
+ 文件名和完整路径作为 File 的成员变量存下来
+ 按钮文字改为:“确认/取消”
+ 点击“确认/取消”发送原路径和目标路径
+ 判断是否选择了目录
+ 如果选择目录，移动到该目录下:如果没有选择目录，移动到当前目录下
+ 拼接目标完整路径
+ 提示用户确认或取消移动
+ 原路径和目标路径的长度放在 caData，原路径和目标路径放在 caMSg
+ 发送给服务器

```cpp
void File::on_mvFile_PB_clicked()
{
    if(ui->mvFile_PB->text() == "移动文件"){
        //获取要移动的文件名
        QListWidgetItem * pItem= ui->listWidget->currentItem();
        if(pItem == NULL){
            QMessageBox::warning(this,"移动文件","请选择要移动的文件");
            return;
        }
        QMessageBox::information(this,"移动文件","请选择要移动的文件");
        ui->mvFile_PB->setText("确认/取消");
        m_strMvFileName = pItem->text();
        m_strMvPath = m_strCurPath+'/'+m_strMvFileName;
        return;
    }
    QListWidgetItem *pItem = ui->listWidget->currentItem();
    QString m_strTarPath;
    QString boxMsg;
    if(pItem == NULL){
       m_strTarPath = m_strCurPath+'/'+m_strMvFileName;
       boxMsg = QString("是否移动到当前目录下");
    }else{
        //判断选择是否为文件夹
        foreach(FileInfo * pFileInfo,m_fileList){
            if(pFileInfo->caName ==pItem->text() && pFileInfo->iFileType !=0){
                QMessageBox::warning(this,"提示","选择的不是文件夹");
                return;
            }
        }
        m_strTarPath = m_strCurPath+'/'+pItem->text()+'/'+m_strMvFileName;
        boxMsg = QString("是否移动到已选中目录下");
    }
    int ret = QMessageBox::information(this,"移动文件",boxMsg,"确认","取消");
    ui->mvFile_PB->setText("移动文件");
    if(ret != 0){//取消
        return;
    }
    int srcLen = m_strMvPath.toStdString().size();
    int tarLen = m_strTarPath.toStdString().size();
    PDU*pdu = mkPDU(srcLen+tarLen+1);
    pdu->uiMsgtype=ENUM_MSG_TYPE_MOVE_FILE_REQUEST;
    //源路径和目标路径长度放在cadat,源路径和目标路径放在camsg
    memcpy(pdu->caData,&srcLen,sizeof(int));
    memcpy(pdu->caData+sizeof(int),&tarLen,sizeof(int));
    memcpy(pdu->caMsg,m_strMvPath.toStdString().c_str(),srcLen);
    memcpy(pdu->caMsg+srcLen,m_strTarPath.toStdString().c_str(),tarLen);
    Client::getInstance().sendPDU(pdu);
}

void ResHandler::moveFile(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_MOVE_FILE_RESPEND start";
    bool ret;
//    qDebug()<<"ret"<<ret;
    memcpy(&ret,pdu->caData,sizeof(bool));
    if(ret){
        Index::getInstance().getFile()->flushFile();
    }else {
        QMessageBox::information(Index::getInstance().getFile(),"移动文件","移动文件失败");
    }
}
```

```cpp
PDU *MsgHandler::moveFile(PDU *pdu)
{
    int srcLen =0;
    int tarLen =0;

    memcpy(&srcLen,pdu->caData,sizeof (int));
    memcpy(&tarLen,pdu->caData+sizeof (int),sizeof (int));

    char * pSrcPath = new char[srcLen+1];
    char * pTarPath = new char[tarLen+1];

    memset(pSrcPath,'\0',srcLen+1);
     memset(pTarPath,'\0',tarLen+1);

    memcpy(pSrcPath,pdu->caMsg,srcLen);
    memcpy(pTarPath,pdu->caMsg+srcLen,tarLen);

    bool res = QFile::rename(pSrcPath,pTarPath);
    PDU * respdu = mkPDU(0);
    respdu->uiMsgtype=ENUM_MSG_TYPE_MOVE_FILE_RESPEND;
    memcpy(respdu->caData,&res,sizeof (bool));
    return respdu;

    delete [] pSrcPath;
    delete [] pTarPath;
    pSrcPath=NULL;
    pTarPath+NULL;
    return respdu;
}
```

### 文件传输
#### 上传文件
+ 客户端发送上传请求
    - 通过文件选择弹窗获得要上传的文件路径，路径作为 Fie 成员变量存下来
    - 从路径取要上传的文件名，将文件名、大小、路径发送给服务器
    - 每次上传一个文件，定义一个是否在上传的状态的 Fie 成员变量

```cpp
void File::on_upload_PB_clicked()
{
    if(m_bUpload){
        QMessageBox::warning(this,"上传文件","已有文件正在上传，请稍等");
        return;
    }
    m_strUploadPath = QFileDialog::getOpenFileName();
    qDebug()<<"m_strUploadPath"<<m_strUploadPath;
    //获取文件名
    int index = m_strUploadPath.lastIndexOf('/');
    QString strFileName = m_strUploadPath.right(m_strUploadPath.length()-index-1);//size取的长度与中文字符不符
    //获取文件大小
    QFile file(m_strUploadPath);
    qint64 fileSize = file.size();
    //发送PDU文件名和大小放入cadata 文7uiyb件路径放在camsg
    PDU * pdu = mkPDU(m_strUploadPath.toStdString().size()+1);
    pdu->uiMsgtype = ENUM_MSG_TYPE_UPLOAD_FILE_REQUEST;
    memcpy(pdu->caData,strFileName.toStdString().c_str(),32);
    memcpy(pdu->caData+32,&fileSize,sizeof(qint64));
    memcpy(pdu->caMsg,m_strCurPath.toStdString().c_str(),m_strCurPath.toStdString().size());
    qDebug()<<"strFileName"<<strFileName
           <<"fileSize"<<fileSize
          <<"m_strCurPath"<<m_strCurPath;
    Client::getInstance().sendPDU(pdu);
}
```

+ 服务器处理上传请求
    - 分片上传，MsgHanler定义几个属性记录上传:上传文件对象、上传文件大小、已上传大小、是否在上传
    - 返回的结果:已有文件在上传、打开文件失败、可以开始上传
    - 拼接路径和文件名得完整路径
    - 使用 open 函数创建文件，更新关于上传的属性
    - 结果返回给客户端

```cpp
PDU *MsgHandler::uploadFile(PDU *pdu)
{
    PDU * respdu = mkPDU(0);
    respdu->uiMsgtype=ENUM_MSG_TYPE_UPLOAD_FILE_RESPEND;
    int ret;
    if(m_bUpload){
        qDebug()<<"已有文件正在上传";
        ret =1;
        memcpy(respdu->caData,&ret,sizeof(int));
        return respdu;
    }
    char caFileName[32]={'\0'};
    qint64 fileSize = 0;

    memcpy(caFileName,pdu->caData,32);
    memcpy(&fileSize,pdu->caData+32,sizeof (qint64));
    QString strPath = QString("%1/%2").arg(pdu->caMsg).arg(caFileName);
    m_fUploadFile.setFileName(strPath);
    if(m_fUploadFile.open(QIODevice::WriteOnly)){//打开成功,更新上传属性
        m_bUpload = true;
        m_iUploadTotal = fileSize;
        m_inUploadReceived=0;
        qDebug()<<"打开文件成功";
    }else{
        qDebug()<<"打开文件失败";
        ret = -1;
    }
    memcpy(respdu->caData,&ret,sizeof (int));
    return respdu;
}
```

+ 客户端根据结果处理，文件发送给服务器
    - File 定义上传文件函数
        * 打开文件、构建 pdu、循环读取文件放入 caMsg、发送 pdu、关闭文件
        * 发送前和发送后更新是否是上传状态的属性
    - 客户端接收消息处调用文件上传函数

```cpp
void File::uploadFile()
{
    //打开要上传的文件
    QFile file(m_strUploadPath);
    if(!file.open(QIODevice::ReadOnly)){
       QMessageBox::warning(this,"上传文件","打开文件失败") ;
       return;
    }
    //构建PU，每次发送4096字节
    m_bUpload = true;
    PDU*datapdu =mkPDU(4096);
    datapdu->uiMsgtype=ENUM_MSG_TYPE_UPLOAD_FILE_DATA_REQUEST;
    qint64 ret = 0;
    //循环读取文件的数据发送给服务器
    while(true){
        ret = file.read(datapdu->caMsg,4096);
        if(ret == 0){//读取成功
            break;
        }
        if(ret < 0){
            QMessageBox::warning(this,"上传文件","上传文件失败：读取失败");
            break;
        }
        //根据ret设置PDU的长度
        datapdu->uiMsgLen = ret;
        datapdu->uiPDULen = ret +sizeof(PDU);
        Client::getInstance().getTcpSocket().write((char*)datapdu,datapdu->uiPDULen);
    }
    m_bUpload = false;
    file.close();
    free(datapdu);
    datapdu = NULL;
}
void ResHandler::uploadFile(PDU *pdu)
{
    int ret;
    memcpy(&ret,pdu->caData,sizeof (int));
    if(ret == -1){
        QMessageBox::information(Index::getInstance().getFile(),"上传文件","打开文件失败");
    }else if(ret == -2){
        QMessageBox::information(Index::getInstance().getFile(),"上传文件","已有文件在上传");
    }else if(ret == 0){
        Index::getInstance().getFile()->uploadFile();
    }
}
```

+ 服务器接收文件
    - caMsg写入文件，更新已接收文件大小的属性
    - 判断是否全部接收完，响应上传完成，关闭文件，更新上传状态

```cpp
PDU *MsgHandler::uploadFileData(PDU *pdu)
{
   m_fUploadFile.write(pdu->caMsg,pdu->uiMsgLen);
   m_inUploadReceived += pdu->uiMsgLen;
   if(m_inUploadReceived < m_iUploadTotal){//未接收完
       return NULL;
   }
   m_fUploadFile.close();
   m_bUpload = false;
   PDU * respdu = mkPDU(0);
   respdu->uiMsgtype=ENUM_MSG_TYPE_UPLOAD_FILE_DATA_RESPEND;
   bool ret = m_inUploadReceived == m_iUploadTotal;
   memcpy(respdu->caData,&ret,sizeof(bool));
   return respdu;
}

```

+ 客户端将数据接收并写入文件
    - File 定义下载文件数据函数，ResHandler中调用
    - 数据写入文件，更新属性
    - 接收完，关闭文件，更新属性，显示提示信息

```cpp
void ResHandler::uploaxdFileData(PDU *pdu)
{
    qDebug()<<"ENUM_MSG_TYPE_UPLOAD_FILE_DATA_RESPEND start";
    bool ret;
//    qDebug()<<"ret"<<ret;
    memcpy(&ret,pdu->caData,sizeof(bool));
    if(ret){
        QMessageBox::information(Index::getInstance().getFile(),"上传文件","上传文件成功");
        Index::getInstance().getFile()->flushFile();
    }else {
        QMessageBox::information(Index::getInstance().getFile(),"上传文件","上传文件失败");
    }
    Index::getInstance().getFile()->m_bUpload = false;
}

```

#### 分享文件
![](https://cdn.nlark.com/yuque/0/2024/png/40599201/1720925124311-9f129023-fbf2-409b-88f4-3bfe7381430a.png)

+ ui 界面:好友列表框设为可多选
+ 实现全选和取消已选槽函数:遍历列表框元素，调用setSelected 函数
+ File 界面的分享按钮槽函数
    - 判断是否选择了文件，记录选择的文件
    - 显示分享文件界面
    - 分享文件界面定义更新好友列表框的函数，将 Friend 界面的好友列表框的内容给当前列表框

```cpp
// 按钮
void ShareFile::on_allSelect_PB_clicked()
{
    for(int i=0;i<ui->listWidget->count();i++){
        ui->listWidget->item(i)->setSelected(true);
    }
}

void ShareFile::on_cancelSelect_PB_clicked()
{
    for(int i=0;i<ui->listWidget->count();i++){
        ui->listWidget->item(i)->setSelected(false);
    }
}
```

```cpp
void File::on_shareFile_PB_clicked()
{
    QListWidgetItem *pItem = ui->listWidget->currentItem();
    if(pItem == NULL){
        QMessageBox::warning(this,"分享文件","请选择要分享的文件");
        return;
    }
    m_strShareFile = pItem->text();
    m_pShareFile->updateFriend_LW();
    if(m_pShareFile->isHidden()){
        m_pShareFile->show();
    }
}
void ShareFile::updateFriend_LW()
{
    ui->listWidget->clear();
    QListWidget * friend_LW = Index::getInstance().getFriend()->getFriend_LW();
    for(int i=0;i<friend_LW->count();i++){
        QListWidgetItem * friendItem = friend_LW->item(i);
        QListWidgetItem *newItem = new QListWidgetItem(*friendItem);
        ui->listWidget->addItem(newItem);
    }
}

void ShareFile::on_allSelect_PB_clicked()
{
    for(int i=0;i<ui->listWidget->count();i++){
        ui->listWidget->item(i)->setSelected(true);
    }
}

void ShareFile::on_cancelSelect_PB_clicked()
{
    for(int i=0;i<ui->listWidget->count();i++){
        ui->listWidget->item(i)->setSelected(false);
    }
}

```

+ 分享确定按钮的槽函数发送请求
    - 拼接当前路径和分享文件名为完整路径
    - 好友数量和当前用户名放入caData，选择的好友名和路径放入caMsg
    - 发送请求给服务器

```cpp
void ShareFile::on_ok_PB_clicked()
{
    //获取当前路径和当前用户名
    QString strCurPath = Index::getInstance().getFile()->m_strShareFile;
    QString strCurName = Client::getInstance().m_strLoginName;
    QString strPath = strCurPath + '/' + Index::getInstance().getFile()->m_strShareFile;//路径

    //获取选择的好友
    QList<QListWidgetItem*> pItem = ui->listWidget->selectedItems();
    int friendNum = pItem.size();

    //构建PUD，当前用户名和好友数量放入cadata 选择的好友名和路径放入camsg
    PDU * pdu = mkPDU(friendNum*32+strPath.toStdString().size()+1);
    pdu->uiMsgtype =ENUM_MSG_TYPE_SHARE_FILE_REQUEST;

    memcpy(pdu->caData,strCurName.toStdString().c_str(),32);
    memcpy(pdu->caData+32,&friendNum,sizeof(int));

    for(int i=0;i<friendNum;i++){
        memcpy(pdu->caMsg+i*32,pItem.at(i)->text().toStdString().c_str(),32);
    }

    memcpy(pdu->caMsg+friendNum*32,strPath.toStdString().c_str(),strPath.toStdString().size());
    Client::getInstance().sendPDU(pdu);
}

```

+ 服务器转发给分享用户并响应
    - 取出当前用户名和好友数量
    - 遍历好友(caMsg)，进行转发，转发发送者的名字和文件路径
    - 服务器响应给发送者

```cpp
PDU *MsgHandler::shareFile(PDU *pdu)
{
    char caCurName[32]={'\0'};
    int friendNum = 0;
    
    memcpy(caCurName,pdu->caData,32);
    memcpy(&friendNum,pdu->caData+32,sizeof(int));
    
    int size = friendNum*32;
    
    PDU * resendpdu = mkPDU(pdu->uiMsgLen-size);
    resendpdu->uiMsgtype=pdu->uiMsgtype;
    
    char caRecvName[32]={'\0'};
    
    memcpy(resendpdu->caData,caCurName,32);
    memcpy(resendpdu->caMsg,pdu->caMsg+size,pdu->uiMsgLen-size);
    for(int i=0;i<friendNum;i++){
        memcpy(caRecvName,pdu->caMsg+i*32,32);
        MyTcpServer::getInstance().resend(caRecvName,resendpdu);
    }
    free(resendpdu);
    resendpdu = NULL;
    PDU * respdu = mkPDU(0);
    respdu->uiMsgtype=ENUM_MSG_TYPE_SHARE_FILE_RESPEND;
    return respdu;
}
```

+ 发送者客户端弹窗显示 文件已分享
+ 转发好友客户端处理是否接收
    - 从路径中取出文件名，弹窗询问用户是否接收
    - 如果接收，将当前用户名和文件路径发给服务器

```cpp
void ResHandler::shareFileRequest(PDU *pdu)
{
    QString strSharePath = pdu->caMsg;
    int index= strSharePath.lastIndexOf('/');
    QString strFileName = strSharePath.right(strSharePath.size() - index -1);
    
    int ret = QMessageBox::question(Index::getInstance().getFile()->getShareFile(),
               "分享文件",
               QString("%1 分享文件 %2 是否接收？").arg(pdu->caData).arg(strFileName));
    if(ret!=QMessageBox::Yes){
        return;
    }
    PDU * respdu = mkPDU(pdu->uiMsgLen);
    respdu->uiMsgtype = ENUM_MSG_TYPE_SHARE_FILE_AGREE_REQUEST;
    memcpy(respdu->caData,Client::getInstance().m_strLoginName.toStdString().c_str(),32);
    memcpy(respdu->caMsg,pdu->caMsg,pdu->uiMsgLen);
    Client::getInstance().sendPDU(pdu);
}
```

+ 服务器处理同意分享文件
    - 得到接收路径(接收用户的根目录)和分享文件路径
    - 通过分享文件路径得到分享文件名，文件名与接收路径拼接成完整路径
    - 复制分文件与目录分别处理，复制目录定义函数遍历文件处理
    - 结果发送给客户端
+ 客户端显示结果并刷新文件

```cpp
PDU *MsgHandler::shareFileAgree(PDU *pdu)
{
    QString strSharePath = pdu->caMsg;
    QString strRecvPath = QString("%1/%2").arg(Server::getInstance().getRootPath().arg(pdu->caData));

    int index = strSharePath.lastIndexOf('/');
    QString strFileName = strSharePath.right(strSharePath.size() - index - 1);
    strRecvPath = strRecvPath + "/" + strFileName;

    bool ret = QFile::copy(strSharePath,strRecvPath);

    PDU * respdu = mkPDU(0);
    memcpy(respdu->caData,&ret,sizeof (bool));
    respdu->uiMsgtype=ENUM_MSG_TYPE_SHARE_FILE_AGREE_REQUEST;
    return respdu;
}


void ResHandler::shareFileAgree(PDU *pdu)
{
    bool ret;
    memcpy(&ret,pdu->caData,sizeof(bool));
    if(ret){
        QMessageBox::information(Index::getInstance().getFile(),"分享文件","分享文件成功");
        Index::getInstance().getFile()->flushFile();
    }else{
        QMessageBox::information(Index::getInstance().getFile(),"分享文件","分享文件失败");
    }
}

```

