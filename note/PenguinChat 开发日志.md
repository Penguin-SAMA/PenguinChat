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