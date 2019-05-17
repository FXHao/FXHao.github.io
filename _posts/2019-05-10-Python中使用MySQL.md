---
layout: post
title: "Python中使用MySQL"
date: 2019-05-10
description: "MySQL数据库、python"
tag: MySQL数据库
---


## Python中使用MySQL

### Python 中操作 MySQL步骤

![](https://FXHao.github.io/images/posts/MySQL/py使用MySQL.jpeg)

#### **导入模块**

* 需要用到 `pymysql` 模块

  ```python
  from pymysql import *
  ```

#### **创建 Connection 对象**

* 用于建立与数据库的连接

* 创建对象：调用`connect()`方法

  ```python
  conn = connect(参数列表)
  ```

  > 参数host：连接的mysql主机，如果本机是'localhost'
  > 参数port：连接的mysql主机的端口，默认是3306
  > 参数database：数据库的名称
  > 参数user：连接的用户名
  > 参数password：连接的密码
  > 参数charset：通信采用的编码方式，推荐使用utf8

* **conn 的方法**
  * `close()`  : 关闭连接
  * `commit()` : 提交
  * `cursor()`返回 Cursor 对象，用于执行 **sql语句** 并获得结果

#### **Cursor对象**

* 用于执行sql语句，使用频度最高的语句为`select`、`insert`、`update`、`delete`

* 获取 Cursor 对象：调用Connection对象的 `cursor()` 方法

  ```python
  cursor = conn.cursor()
  ```

### 查询

```python
from pymysql import *

def main():
    # 创建Connection连接
    conn = connect(host='localhost',port=3306,user='root',password='mysql',database='jing_dong',charset='utf8')
    # 获得Cursor对象
    cursor = conn.cursor()
    # 执行sql语句
    sql = "select id,name from goods where id>=4"
    cursor.execute(sql)
	# 显示查询结果
    for temp in cursor.fetchall():
        print(temp)
   
    # 关闭Cursor对象
    cs1.close()
    conn.close()

if __name__ == '__main__':
    main()
```

### 增删改

* 只要对数据库里的数据做了修改，修改后就得让连接对象 `conn` 调用一下 `commit()` 方法提交一下，不然在数据库中不会显示修改的数据

* 若修改数据后，在没提交之前，又不想修改了，这是就可以让 `conn` 调用一下 `rollback()` 取消修改

  ```python
  from pymysql import *
  
  def main():
      # 创建Connection连接
      conn = connect(host='localhost',port=3306,database='jing_dong',user='root',password='mysql',charset='utf8')
      # 获得Cursor对象
      cs1 = conn.cursor()
      # 执行insert语句，并返回受影响的行数：添加一条数据
      # 增加
      count = cs1.execute('insert into goods_cates(name) values("硬盘")')
      #打印受影响的行数
      print(count)
  
      count = cs1.execute('insert into goods_cates(name) values("光盘")')
      print(count)
  
      # # 更新
      # count = cs1.execute('update goods_cates set name="机械硬盘" where name="硬盘"')
      # # 删除
      # count = cs1.execute('delete from goods_cates where id=6')
  
      # 提交之前的操作，如果之前已经之执行过多次的execute，那么就都进行提交
      conn.commit()
  
      # 关闭Cursor对象
      cs1.close()
      # 关闭Connection对象
      conn.close()
  
  if __name__ == '__main__':
      main()
  ```

### 综合案例

```python
import pymysql

class JD(object):
    """docstring for JD"""
    def __init__(self):
        # 创建Connect连接
        self.conn = pymysql.connect(host='localhost',port=3306,database='jing_dong',user='root',password='mysql',charset='utf8')
        # 获取Cursor对象
        self.cursor = self.conn.cursor()

    def __del__(self):
        # 关闭对象
        self.cursor.close()
        self.conn.close()

    def execute_sql(self, sql):
        self.cursor.execute(sql)
        for temp in self.cursor.fetchall():
            print(temp)

    def show_all_items(self):
        """显示所有商品"""
        sql = "select * from goods;"
        self.execute_sql(sql)

    def show_cates(self):
        """显示商品分类"""
        sql = "select cate_name from goods;"
        self.execute_sql(sql)

    def show_brands(self):
        """显示品牌分类"""
        sql = "select brand_name from goods;"
        self.execute_sql(sql)

    def add_brands(self):
        item_name = input("请输入新商品分类的名称：")
        sql = """insert into goods(brand_name) values("%s");""" % item_name
        self.cursor.execute(sql)
        self.conn.commit()

    @staticmethod
    def print_menu():
        print("-------京东-----")
        print("[1]:所有商品")
        print("[2]:所有商品分类")
        print("[3]:所有商品品牌分类")
        print("[4]:添加一个商品分类")
        num = input("请输入功能选项：")
        return num

    def run(self):
        while True:
            num = self.print_menu()
            if num == "1":
                # 查询所有商品
                self.show_all_items()
            elif num == "2":
                # 查询商品分类
                self.show_cates()
            elif num == "3":
                # 查询品牌分类
                self.show_brands()
            elif num == "4":
                # 增加商品分类
                self.add_brands()
            else:
                print("输入有误～")

def main():
    # 创建一个京东对象
    jd = JD()

    # 调用这个对象运行
    jd.run()

if __name__ == '__main__':
    main()
```

------

### 参数化

* **SQL** **注入**

  > SQL注入是一种将SQL代码添加到输入参数中，传递到服务器解析并执行的一种攻击手法。
  >
  > SQL注入攻击是输入参数未经过滤，然后直接拼接到SQL语句当中解析，执行达到预想之外的一种行为，称之为SQL注入攻击。

  

  ```python
  find_name = input("请输入物品名称：")
  # 输入 " or 1=1 or "1    (双引号也要输入)
  sql = 'select * from goods where name="%s";' % find_name
  # print("""sql===>%s<====""" % sql)
  # 执行select语句，并返回受影响的行数：查询所有数据
  count = cursor.execute(sql)
  ```

  > 这时的 **sql语句** 就变成了 `select * from goods where name="" or 1=1 or "1"`
  >
  > 而这个语句的条件是恒成立的，所以，它就会执行，返回所有商品的信息，即返回所有的数据

* **防止SQL注入**

  ```python
  find_name = input("请输入物品名称：")
  sql = "select * from goods where name=%s;"
  count = cursor.execute(sql, [find_name])
  ```

  * 把要查询的东西扔到一个列表里，这就是所谓的参数化，防止简单的SQL注入

**END.....** 