---
title:     "MYSQL配置读取"  # 文章标题
subtitle:    "MYSQL配置读取"  # 文章标题
description: "讲解mysql配置文件读取顺序,文件中配置读取顺序"
date:         2019-08-19T15:06:29+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["配置","MySQL"]
url:    "/2019-08-19-MYSQL配置读取"
categories:  ["sql"]
showtoc: true   # 是否显示目录
---

## 配置文件读取

在Unix, Linux 或者 Mac OS X 系统中, MYSQL 从以下配置文件中读取启动配置, 读取顺序由上到下:

|    File Name         | Purpose |
| ----------           | --- |
| <font color=Red> /etc/my.cnf </font>         |  全局配置选项 |
| <font color=Red> /etc/mysql/my.cnf </font>    |  全局配置选项 |
| <font color=Red> SYSCONFDIR/my.cnf </font>   |  全局配置选项 |
| <font color=Red> $MYSQL_HOME/my.cnf </font>   | 特定于服务器的选项（仅限服务器） |
| <font color=Red> defaults-extra-file </font> | 命令行指定的额外配置文件路径|
| <font color=Red> ~/.my.cnf </font>          |  用户特定选项 |
| <font color=Red> ~/.mylogin.cnf </font>     | 用户特定的登录路径选项（仅限客户端）|


--defaults-extra-file
 在读取全局配置文件之后，读取用户配置文件(~/.my.cnf)之前，读取extra指定的参数文件
  
~/.my.cnf  
home目录下面的隐藏文件，my.cnf前面的点，说明my.cnf是隐藏文件 

~/.mylogin.cnf  
配置文件有点儿特殊，它不是一个纯文本文件（其他的配置文件都是纯文本文件），而是使用mysql_config_editor实用程序创建的加密文件。文件中只能包含一些用于启动客户端软件时连接服务器的一些选项，包括 host、user、password、port和 socket。而且它只能被客户端程序所使用。

**<font color=Red>读取顺序:</font>**

* /etc/my.cnf
* basedir/my.cnf
* datadir/my.cnf
* --defaults-extra-file
* ~/.my.cnf
* ~/.mylogin.cnf

最后两个以~开头的路径是用户相关的，**<font color=Red>~</font>** 就代表这个用户目录. 用户可以在自己的用户目录下创建<font color=Red>.my.cnf</font>或者<font color=Red>.mylogin.cnf</font>，换句话说，不同登录用户使用的 <font color=Red>.my.cnf</font>或者 <font color=Red>.mylogin.cnf</font> 配置文件是不同的。

**当多个文件中出现同一个配置时,以最后读取的配置为准** 

比方说<font color=Red>/etc/my.cnf</font>文件的内容是这样的：
```
[server]
default-storage-engine=InnoDB
```
而~/.my.cnf文件中的内容是这样的：

```
[server]
default-storage-engine=MyISAM
```

又因为<font color=Red>~/.my.cnf</font>比<font color=Red>/etc/my.cnf</font>顺序靠后，所以如果两个配置文件中出现相同的启动选项，以<font color=Red>~/.my.cnf</font>中的为准，所以MySQL服务器程序启动之后，<font color=Red>default-storage-engine</font>的值就是<font color=Red>MyISAM</font>。

## 文件中配置读取
与在命令行中指定启动选项不同的是，配置文件中的启动选项被划分为若干个组，每个组有一个组名，用中括号[]扩起来，像这样：

```
[server]
(具体的启动选项...)

[mysqld]
(具体的启动选项...)

[mysqld_safe]
(具体的启动选项...)

[client]
(具体的启动选项...)

[mysql]
(具体的启动选项...)

[mysqladmin]
(具体的启动选项...)
```

配置文件中不同的选项组是给不同的启动命令使用的，<font color=Red>[mysqld]</font>和<font color=Red>[mysql]</font>组分别应用于<font color=Red>mysqld</font>服务器程序和<font color=Red>mysql</font>客户端程序。不过有两个选项组比较特别：

* [server]组下边的启动选项将作用于所有的服务器程序。

* [client]组下边的启动选项将作用于所有的客户端程序。

|   启动命令         | 类别 |  能读取的组 |
| ----------           | --- | ---|
| mysqld	| 启动服务器|	[mysqld]、[server] |
| mysqld_safe	| 启动服务器|	[mysqld]、[server]、[mysqld_safe] |
| mysql.server	| 启动服务器|	[mysqld]、[server]、[mysql.server] |
| mysql	| 启动客户端	| [mysql]、[client]|
| mysqladmin	| 启动客户端|	[mysqladmin]、[client] |
| mysqldump	| 启动客户端 |	[mysqldump]、[client] |


假如我们在<font color=Red>/etc/mysql/my.cnf</font> 中添加配置

```
[server]
skip-networking
default-storage-engine=MyISAM
```

然后直接用mysqld启动服务器程序：

```
mysqld
```

虽然在命令行没有添加启动选项，但是在程序启动的时候，就会默认的到我们上边提到的配置文件路径下查找配置文件，其中就包括<font color=Red>/etc/mysql/my.cnf</font>。又由于<font color=Red>mysqld</font>命令可以读取<font color=Red>[server]</font>选项组的内容，所以<font color=Red>skip-networking</font>和<font color=Red>default-storage-engine=MyISAM</font>这两个选项是生效的。你可以把这些启动选项放在<font color=Red>[client]</font>组里再试试用<font color=Red>mysqld</font>启动服务器程序，结果是**不生效**。


**同一个配置文件中多个组的优先级**

我们说同一个命令可以访问配置文件中的多个组，比如<font color=Red>mysqld</font>可以访问<font color=Red>[mysqld]</font>、<font color=Red>[server]</font>组，如果在同一个配置文件中，比如<font color=Red>~/.my.cnf</font>，在这些组里出现了同样的配置项，比如这样：

```
[server]
default-storage-engine=InnoDB

[mysqld]
default-storage-engine=MyISAM
```

那么，将以最后一个出现的组中的启动选项为准，比方说例子中<font color=Red>default-storage-engine</font>既出现在<font color=Red>[mysqld]</font>组也出现在<font color=Red>[server]</font>组，因为<font color=Red>[mysqld]</font>组在<font color=Red>[server]</font>组后边，就以<font color=Red>[mysqld]</font>组中的配置项为准。

**<font color=Red>如果同一个启动选项既出现在命令行中，又出现在配置文件中，那么以命令行中的启动选项为准。</font>**

例如我们在配置文件中写了：

```
[server]
default-storage-engine=InnoDB
```

而我们的启动命令是：

```
mysql.server start --default-storage-engine=MyISAM
```

那最后<font color=Red>default-storage-engine</font>的值就是<font color=Red>MyISAM</font>！

