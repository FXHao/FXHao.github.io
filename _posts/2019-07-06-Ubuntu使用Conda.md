---
layout: post
title: "Ubuntu使用Conda"
date: 2019-07-06
description: "Conda"
tag: Conda
---

## Ubuntu使用Conda

### Conda是做什么的

* 它是一个软件包和环境的管理系统，用于安装多个版本的软件包及其依赖关系，并可以在它们之间轻松切换

### 安装

* [官网下载对应的版本](https://www.anaconda.com/distribution/#linux)

* 然后就是安装

  ```shell
  bash 下载文件的位置
  ```

* 然后就是一路跟着**回车**或**yes**

* 看到![](https://FXHao.github.io/images/posts/conda.png)那么恭喜你安装成功了

### 配置环境

* 打开`.bashrc`

  ```shell
  sudo gedit ~/.bashrc
  ```

* 在下面添加一行内容

  ```shell
  export PATH=/home/fxh/anaconda3/bin:$PATH
  ```

  > 注意：这里是你*anaconda3*安装的路径，自行更改

* 更新下环境变量

  ```shell
  source ~/.bashrc
  ```

* 要想知道自己安装成功没有，可以输入`conda list`来测试一下

### Conda的使用

```shell
conda --version   # 查看conda 版本
conda update conda   # 更新conda
conda create --name <env_name> <package_names>  # 新建虚拟环境
source activate env_name  # 切换conda环境
source deactivate  # 退出虚拟环境
conda env list  # 显示安装过的所有虚拟环境
conda create --name new_env_name --clone copied_env_name  # 复制环境
conda remove --name env_name --all  # 删除环境
conda install 要安装的包名  # 在当前环境中安装包
```

