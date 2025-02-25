# PenguinChat 开发日志

## 登陆界面

![image-20250221124831416](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20250221124831416.png)

使用 `Spacer` 将所有输入框和按钮对齐并挤在一起。

`````c++
// 在mainwindow.cpp中添加
	_login_dlg = new LoginDialog();
    setCentralWidget(_login_dlg);
    _login_dlg->show();
`````

## 注册界面

![image-20250221125212676](https://raw.githubusercontent.com/Penguin-SAMA/PicGo/main/image-20250221125212676.png)

```c++
connect(_login_dlg, &LoginDialog::switchRegister, this, &MainWindow::SlotSwitchReg);
    _reg_dlg = new RegisterDialog();
```

## 单例类

```c++
#ifndef SINGLETON_H
#define SINGLETON_H

#include "global.h"

template<typename T>
class Singleton{
protected:
    Singleton() = default;
    Singleton(const Singleton<T>&) = delete;
    Singleton& operator=(const Singleton<T>&) = delete;

    static std::shared_ptr<T> _instance;

public:
    static std::shared_ptr<T> GetInstance() {
        static std::once_flag s_flag;
        std::call_once(s_flag, [&]() {
            // _instance = std::shared_ptr<T>(new T);
            _instance = std::make_shared<T>();
        });

        return _instance;
    }

    void PrintAddress() {
        std::cout << _instance.get() << std::endl;
    }

    ~Singleton() {
        std::cout << "this is singleton destruct" << std::endl;
    }
};

template <typename T>
std::shared_ptr<T> Singleton<T>::_instance = nullptr;

#endif // SINGLETON_H

```

### 单例类

- 默认构造函数定义在 `protected` 中，允许派生类调用，但禁止外部直接实例化。
- 删除拷贝构造函数和赋值运算符。
- `std::once_flag` 和 `std::call_once` 配合，确保lambda表达式只执行一次。
- 单例实例在第一次调用 `GetInstance` 时才创建，节省资源。

### 单例模式优缺点

优点：

1. 通过 `GetInstance` 提供同一访问入口，方便使用
2. 延迟初始化避免了不必要实例化的开销
3. 线程安全

缺点：

1. 单例本质上是一个全局变量，可能导致代码耦合，难以测试和维护
2. 依赖单例的模块可能不显式声明依赖，降低代码透明度
3. 单例实例的销毁时机由智能指针控制，不一定和程序需求匹配

### 疑难点：

1. **std::once_flag** 是一个标志，用于配合 std::call_once 来确保某个函数或代码块在多线程环境下只执行一次。
2. **std::call_once** 是一个函数，它接受一个 std::once_flag 和一个可调用对象（例如 lambda 表达式）。它保证这个可调用对象在程序的生命周期内只被执行一次，即使在多线程环境中。

## http 管理类

首先在 `.pro` 文件中添加网络库。

```c++
QT += core gui network
```

头文件如下：

```c++
#ifndef HTTPMGR_H
#define HTTPMGR_H

#include "singleton.h"
#include <QJsonDocument>
#include <QJsonObject>
#include <QNetworkAccessManager>
#include <QObject>
#include <QString>
#include <QUrl>

// CRTP
class HttpMgr : public QObject,
                public Singleton<HttpMgr>,
                public std::enable_shared_from_this<HttpMgr>
{
    Q_OBJECT
public:
    ~HttpMgr();

private:
    friend class Singleton<HttpMgr>;
    HttpMgr();
    QNetworkAccessManager _manager;

    void PostHttpReq(QUrl url, QJsonObject json, ReqId req_id, Modules mod);

private slots:
    void slot_http_finish(ReqId id, QString res, ErrorCodes err, Modules mod);

signals:
    void sig_http_finish(ReqId id, QString res, ErrorCodes err, Modules mod);
    void sig_reg_mod_finish(ReqId id, QString res, ErrorCodes err);
};

#endif // HTTPMGR_H

```

## 配置 GateServer

在 VS 中配置了 boost 和 jsoncpp。后面在 linux 上配置一下，把网关服务器配置到 linux 上。

VS 的配置实在繁琐，后面试试用 xmake 重新配一下。这段就省略了。

