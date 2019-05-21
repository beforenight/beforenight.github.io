---
layout: post
title: "Python3连接MySQL数据库"
subtitle: "Python3 with MySQL Database"
author: "beforenight"
header-img: "img/post-bg-dreamer.jpg"
header-mask: 0.4
tags:
  - Python
  - MySQL
  - CRUD
---

# Python3连接MySQL数据库

<a name="OQ8vy"></a>
## 适用环境
python版本 >=2.6或3.3<br />mysql版本>=4.1

<a name="nUcAV"></a>
## 安装
可以使用pip安装也可以手动下载安装。<br />使用pip安装，在命令行执行如下命令：

```bash
pip install PyMySQL
```
手动安装，请先下载。[下载地址](https://github.com/PyMySQL/PyMySQL/tarball/pymysql-X.X)：https://github.com/PyMySQL/PyMySQL/tarball/pymysql-X.X。<br />其中的X.X是版本（目前可以获取的最新版本是0.6.6）。<br />下载后解压压缩包。在命令行中进入解压后的目录，执行如下的指令：

```bash
python setup.py install
```
<a name="Gx4eX"></a>
## 使用示例
一般方式：
```python
import pymysql.cursors
 
# Connect to the database
connection = pymysql.connect(host='127.0.0.1',
                             port=3306,
                             user='root',
                             password='zhyea.com',
                             db='employees',
                             charset='utf8mb4',
                             cursorclass=pymysql.cursors.DictCursor)
```

更优雅得方式：也可以使用字典进行连接参数的管理，我觉得这样子更优雅一些。
```python
import pymysql.cursors
 
config = {
          'host':'127.0.0.1',
          'port':3306,
          'user':'root',
          'password':'zhyea.com',
          'db':'employees',
          'charset':'utf8mb4',
          'cursorclass':pymysql.cursors.DictCursor,
          }
 
# Connect to the database
connection = pymysql.connect(**config)
```
<a name="Fftzb"></a>
## 在django中使用
在django中使用是我找这个的最初目的。目前同时支持python3.4、django1.8的数据库backend并不好找。这个是我目前找到的最好用的。<br />设置DATABASES和官方推荐使用的MySQLdb的设置没什么区别：

```python
DATABASES = {
   'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'mytest',
        'USER': 'root',
        'PASSWORD': 'zhyea.com',
        'HOST': '127.0.0.1',
        'PORT': '3306',
    }
}
```
关键是这里：我们还需要在站点的__init__.py文件中添加如下的内容：

```python
import pymysql
pymysql.install_as_MySQLdb()
```

<a name="1s9Ez"></a>
## 核心代码
```python
# !/usr/bin/python
# -*- coding: utf-8 -*-
__author__ = 'zhangswings'
__time__ = '2019/5/21 14:03'

import pymysql.cursors
from datetime import date, datetime, timedelta

# # 连接数据库
# connect = pymysql.Connect(host='192.168.65.130', port=3306,
#                           user='root', password='admin', database='ganguo',
#                           charset='utf8', cursorclass=pymysql.cursors.DictCursor)
# # ,encoding='utf8'
# # 获取游标
# cursor = connect.cursor()

# 连接配置信息
config = {
    'host': '192.168.65.130',
    'port': 3306,
    'user': 'root',
    'password': 'admin',
    'database': 'ganguo',
    'charset': 'utf8',
    'cursorclass': pymysql.cursors.DictCursor,
}
# 连接数据库 connect(**config)
connection = pymysql.connect(**config)
# # 获取游标
# cursor = connection.cursor()

# try:
#     with connection.cursor() as cursor:
#         # 执行sql
#         sql = '''
#         INSERT INTO `user_table` VALUES ('zhangswings', '111222', '18610121232', '上海市杨浦区军工路', '13322-3444-13434', %s, '10298-ythlhjlsddnsk-888');
# '''
#         now__date = datetime.now()
#         cursor.execute(sql, (now__date))
#     # 没有设置默认自动提交，需要主动提交，以保存所执行的语句
#     connection.commit()
# finally:
#     connection.close()

# 插入操作
# try:
#     with connection.cursor() as cursor:
#         sql = '''
#             INSERT INTO user_table (userName,userPwd,userPhone,userAddress,userDeviceId,UID,userToken)
#             VALUES(%s,%s,%s,%s,%s,%s,%s)
#         '''
#         cursor.execute(sql, (
#             'zhangswings', '111222', '18610121232', '上海市杨浦区军工路', '13322-3444-13434', datetime.now(),
#             '10298-ythlhjlsddnsk-888'))
#
#     connection.commit()
#
# finally:
#     connection.close()

# 查询操作
try:
    with connection.cursor() as cursor:
        # 执行sql查询
        sql = '''
            SELECT userName,userPwd,userPhone,userAddress,userDeviceId,UID,userToken FROM user_table WHERE
            userName=%s
        '''
        cursor.execute(sql, ('zhangswings'))
        # 获取查询结果
        # result_one = cursor.fetchone()
        result_one = cursor.fetchmany(2)
        # result_one = cursor.fetchall()

        print(result_one)

    connection.commit()
finally:
    connection.close()

```


