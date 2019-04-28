---
layout: post
title: "socket之udp聊天器"
date: 2019-04-18 
description: "python、网络编程、socket、python进阶"
tag: python、socket
---

## socket之udp聊天器

* **socket**是进程间通信的一种方式，它可以实现不同的主机间进程的通信，总之，网络通信之必备

### 创建socket

```python
import socket
udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
```

> 注：socket内参数，第一项为固定参数，第二项是选择创建udp socket还是tcp socket

### 使用socket

#### 1.发送数据

* 发送数据要调用方法 `sendto`

```python
udp_socket.sendto(b"要发送的数据", (对方的IP, port)
```

>  如果是简单的直接发送固定的数据，要在该数据前加一个 `b` ，转换一下格式

* 也可用一变量来获取要发送的数据

```python
send_data = input("请输入要发送的数据：")
udp_socket.sendto(send_data.encode("gbk"), ("192.168.43.159", 8080))
```

> Windows编码是gbk, ubuntu编码是utf-8
>
> 注：若发送时没绑定端口，每次发送时都是系统随机分配的端口

#### 2.接受数据

* 接受时调用的方法 `recvfro`

* 步骤：

  1.绑定一个本地信息

  ```python
  udp_socket.bind(("", 7788))  
  ```

  > 该信息是一个元组，IP和端口号

  2.开始接受数据

  ```python
  recv_data = udp_socket.recvfrom(1024)
  ```

  > recv_data 这个变量储存的是一个元组`(接受到的数据,(发送方的IP，port))`

  3.打印接受数据

  ```python
  print("%s:%s" % recv_data[1], recv_data[0].decode("gbk")))
  ```

  > `recv_data[0]` 之中存储的是接受的数据
  >
  > `recv_data[1]`  之中存储的是发送方的地址信息

### 关闭socket

```python
udp_socket.close()
```

> > > > > > > > > > > > > > > > > > > > > > > > > > > > > > > > > > > > > > 

### udp聊天器

```python
import socket


def send_smg(udp_socket):
    """发送消息"""
    # 获取目标ip/port
    dest_ip = input("请输入目标ip：")
    dest_port = int(input("请输入目标端口："))
    # 获取要发送的内容
    send_data = input("请输入要发送的消息：")
    udp_socket.sendto(send_data.encode("gbk"), (dest_ip, dest_port))


def recv_msg(udp_socket):
    """接受数据"""
    # 接受显示
    recv_data = udp_socket.recvfrom(1024)
    print("%s : %s" % (recv_data[1], recv_data[0].decode("gbk")))


def main():
    # 创建socket
    udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    # 绑定信息
    udp_socket.bind(("", 7788))

    # 循环来进行处理事情
    while True:
        print("udp聊天器v1.0".center(30, "-"))
        print("1.发送消息")
        print("2.接受消息")
        print("0.退出系统")
        op = input("请输入功能选项：")

        if op == "1":
            # 发送
            send_smg(udp_socket)
        elif op == "2":
            # 接受
            recv_msg(udp_socket)
        elif op == "0":
            break
        else:
            print("输入有误请重新输入...")

    udp_socket.close()


if __name__ == "__main__":
    main()

```

*******************************************************************************************************************************************************************************************************************************************************

### 总结

**socket** 是全双工的通信，但此处只实现了半双工，往后可以改进改进

还有就是此处用到一个软件 **网络调试助手** 可向它收发的数据，看其中的过程