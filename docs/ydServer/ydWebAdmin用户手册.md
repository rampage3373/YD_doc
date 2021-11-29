引言
==

易达投资者账户管理软件（简称帐户管理，或ydWebAdmin）运行在Linux服务器上，主要用于管理易达交易系统软件的投资者账户和App授权码。

易达投资者账户管理软件的系统结构如下图所示。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_arch.png)

易达投资者账户管理软件需要安装在易达交易系统软件（下简称易达服务器）同一台服务器上。

本文档面向期货公司的业务和技术人员。

安装和配置
=====

将整个安装包解压在易达服务器安装目录(product\_env)下，与ydServer同级目录。

根据ydServer面向的交易所不同，目录名可能增加了交易所代码作为后缀，如：ydServer.shfe。建议将ydAdmin目录同样增加后缀，如：ydAdmin.shfe。

解压后，需要进行配置才能正常工作。

注意，必须使用和操作ydServer同样的操作系统用户进行操作，不建议使用root用户进行操作。

ydAdmin.ini
-----------

    ServerDir=/home/trade/product_env/ydServer.shfe
    OperationDir=adminOperations
    logDir=adminLog
    

此ini文件是配置文件，定义了易达交易系统的位置：

*   ServerDir=/home/trade/product\_env/ydServer.shfe

定义需要管理的易达交易服务器的安装路径。

*   OperationDir=adminOperations

账户管理软件的管理员操作流水，必须事先创建好。

*   logDir=adminLog

账户管理软件的管理员操作日志，必须事先创建好。

ydWeb.ini
---------

    address=0.0.0.0
    port=8080
    

此ini文件是易达投资者账户管理软件的启动参数配置文件：

*   address=0.0.0.0
    
    启动管理软件时，绑定的IP地址。可以定义服务启动时绑定的IP。  
    如果绑定服务器上全部的IP，可以用0.0.0.0。  
    如考虑到安全及减少对服务器性能的影响，可以仅绑定管理网卡的IP。
*   port=8080
    
    启动管理软件时，绑定的端口。

启动
--

在ydAdmin目录下运行：

    $ ./run.sh
    

如果需要确认管理软件是否在运行，可以查找进程”ydWebAdmin” ：

    $ ps –ef | grep ydWebAdmin
    

如果存在进程，则表示正在运行。

注意：有可能有多个ydWebAdmin进程，这是正常现象，服务器会根据请求情况产生多个进程。

主界面
===

登录
--

在管理工作站上，使用浏览器输入http://<ip>:<port>，即可进入管理软件。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_login01.png)

第一次打开时，会提示输入用户名和密码。用户名和密码是易达交易服务器软件的管理员账户。

登录以后，进入如下页面。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_account01.png)

投资者帐号管理
-------

### 投资者帐号列表

点击“投资者账号管理”，会出现投资者帐号列表。列出当前易达服务器上的所有投资者帐号。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_account01.png)

在这个界面可以修改投资者资金使用量、警告等级1、警告等级2、  
最大订单数、最大连接数、是否允许发布席位优选结果、是否由系统自动组合持仓、是否允许使用裸协议，  
可以重置投资者密码，还可以删除投资者帐号。

每个帐号的状态有以下几种：

**状态**

**说明**

1

有效

该投资者帐号是有效的。

2

将新增

该投资者帐号将在下一个交易日生效。

3

将修改

该投资者帐号信息将在下一次易达服务器重启后生效。

4

将删除

该投资者帐号将在下一次易达服务器重启后删除。

### 修改投资者资金使用量

点击一个投资者的资金使用量，即可修改该投资者的资金使用量数值。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_account_fund.png)

点击“确定”保存，点击“取消”恢复原来的值。

### 修改投资者警告等级1

点击警告等级1数值，即可修改该投资者的警告等级1的值。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_account_warning1.png)

点击“确定”保存，点击“取消”恢复原来的值。

### 修改投资者警告等级2

点击警告等级2数值，即可修改该投资者的警告等级2的值。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_account_warning2.png)

点击“确定”保存，点击“取消”恢复原来的值。

### 修改投资者最大订单数

点击最大订单数的数值，即可修改该投资者最大订单数的值。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_account_maxorders.png)

点击“确定”保存，点击“取消”恢复原来的值。

### 修改投资者最大连接数

点击最大连接数的数值，即可修改该投资者最大连接数的值。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_account_maxsessions.png)

点击“确定”保存，点击“取消”恢复原来的值。

### 修改是否允许投资者发布席位优选结果

点击是否允许复选框，即可以允许或者禁止投资者发布席位优选结果。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_account_xwselect.png)

### 修改是否允许系统帮投资者自动组合持仓

点击是否允许复选框，即可以允许或者禁止由系统自动帮投资者组合持仓。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_account_autocp.png)

### 修改是否允许投资者使用裸协议

点击是否允许复选框，即可以允许或者禁止投资者使用裸协议。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_account_rawprotocol01.png)

### 重置投资者帐号口令

点击“重置口令”，弹出重置口令对话框。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_account_resetpasswd.png)

输入新口令和重复口令，如果相符即可修改口令。

注意，这里修改的口令是投资者登录易达服务器的口令。

### 删除投资者帐号

点击“删除”，会弹出确认框。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_account_del.png)

点击“确定”，则会删除此账号。

### 创建投资者帐号

点击“新建“可以创建投资者帐户。或者从菜单选择”创建账号“来新增一个投资者帐号。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_account_new.png)

### 生效时间

需要注意的是，针对投资者帐号的操作，不是实时生效的，需要等到易达交易服务器下一次重新启动时候生效。

AppID和授权码管理
-----------

### 授权码列表

显示易达交易系统已经注册的授权码列表，可以删除（不能修改）。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_appid01.png)

每个授权码的状态有以下几种可能：

**状态**

**说明**

1

有效

该授权码是有效的。

2

将新增

该授权码将在下一次易达服务器重启后生效。

3

将删除

该授权码将在下一次易达服务器重启后删除。

### 新增授权码

点击“新增”按钮，弹出对话框。输入“终端厂商代码”、“终端软件代码”、“版本号”。点击新建，系统会自动生成授权码。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_appid_new.png)

### 删除授权码

在授权码列表中，点击“删除”，删除相应的授权码。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydWebAdmin_appid_del.png)

注意，部分易达交易系统自身使用的授权码，不允许通过本管理软件删除。

### 生效时间

需要注意的是，新增和删除授权码操作，均需要等到下一次易达服务器重启后生效。

服务器日志
-----

查看服务器当前日志。日志内容会随着易达服务器日志的产生实时更新。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/serverlog01.png)

可以勾选“Error & Warning”，仅显示错误和报警的日志。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/serverlog_error_warning.png)