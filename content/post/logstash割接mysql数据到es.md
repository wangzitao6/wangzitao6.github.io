---
title:     "Logstash割接mysql数据到es"  # 文章标题
subtitle:    "Logstash割接mysql数据到es"  # 文章标题
description: ""
date:         2020-06-24T19:56:25+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["elk"]
url:    "/2020-06-24-logstash割接mysql数据到es/"
categories:  ["elk"]
showtoc: true   # 是否显示目录
---

## ElasticSearch创建索引

创建ES索引

```
PUT logstash_demo
{
    "mappings" : {
      "properties" : {
        "email" : {
          "type" : "text"
        },
        "first_name" : {
          "type" : "text"
        },
        "last_name" : {
          "type" : "text"
        },
        "uid" : {
          "type" : "long"
        }
      }
    }
  }
```

创建好后查询索引数据

```
GET logstash_demo/_search
{
  "query": {
    "match_all": {}
  }
}
```
这时查询熟的数据为空。

## MySql创建表

创建表并插入数据
  ```sql
create table contacts (
    uid serial,
    email VARCHAR(80) not null,
    first_name VARCHAR(80) NOT NULL,
    last_name VARCHAR(80) NOT NULL
);

INSERT INTO contacts(email, first_name, last_name) VALUES('jim@example.com', 'Jim', 'Smith');
INSERT INTO contacts(email, first_name, last_name) VALUES('john@example.com', 'John', 'Smith');
INSERT INTO contacts(email, first_name, last_name) VALUES('carol@example.com', 'Carol', 'Smith');
INSERT INTO contacts(email, first_name, last_name) VALUES('sam@example.com', 'Sam', Smith);
  ```

## 准备LogStash

### 安装LogStash

[去官网](https://www.elastic.co/cn/downloads/logstash)下载LogStash安装包，并解压。

### 安装jdbc插件

解压后进入logstash解压包的bin目录，执行命令

```
./logstash-plugin install logstash-input-jdbc
```
如何是在win上，则执行
```
./logstash-plugin.bat install logstash-input-jdbc
```

如果这个命令安装不了，可以去[这里](https://mvnrepository.com/artifact/mysql/mysql-connector-java)下载 mysql-connector-java  jar包，我这里下载的是mysql-connector-java-8.0.18.jar。

然后把jar包放在LogStash文件夹下的vendor下，创建/jar/jdbc ，放入jar。

### 编写LogStash脚本

#### demo1

简单的脚本，把数据库中数据打印出来

``` conf
input {
    jdbc {
        # Postgres jdbc connection string to our database, mydb
        jdbc_connection_string => "jdbc:mysql://127.0.0.1:3306/demo"
        # 用户名密码
	    jdbc_user => "root"
	    jdbc_password => "123456"
	    # jar包的位置
	    jdbc_driver_library => "../vendor/jar/jdbc/mysql-connector-java-8.0.18.jar"
	    # mysql的Driver
	    jdbc_driver_class => "com.mysql.jdbc.Driver"
        # our query
        statement => "SELECT * FROM contacts"
    }
}
output {
    stdout { codec => json_lines }
}

```

#### demo2

把MySql中数据割进ElasticSearch中

```conf
input {
    jdbc {
        # Postgres jdbc connection string to our database, mydb
        jdbc_connection_string => "jdbc:mysql://127.0.0.1:3306/demo"
        # 用户名密码
	    jdbc_user => "root"
	    jdbc_password => "123456"
	    # jar包的位置
	    jdbc_driver_library => "../vendor/jar/jdbc/mysql-connector-java-8.0.18.jar"
	    # mysql的Driver
	    jdbc_driver_class => "com.mysql.jdbc.Driver"
        # our query
        statement => "SELECT * FROM contacts"
    }
}

filter {
    json {
        source => "message"
        remove_field => ["message"]
    }

    mutate {
        remove_field => "@version"
        remove_field => "@timestamp"
    }
}

output {
    elasticsearch {
        index => "contact1"
		action => "create"
        document_type => "_doc"
        document_id => "%{uid}"
        hosts => "127.0.0.1:9200"
       }
} 

```

## 参考
[www.elastic.co](https://www.elastic.co/cn/blog/logstash-jdbc-input-plugin)



