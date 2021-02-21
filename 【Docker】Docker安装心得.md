# 【Docker】Docker安装心得

## 1. Docker安装

Docker是一种软件容器平台，用于解决**程序运行环境不同而导致程序出错**的问题。简单来说Docker类似于可以提交运行环境的git。

由于作者操作系统是**Windows 10 家庭版**因此本文以该操作系统为背景进行。

### 1.1 下载Docker并安装

下载[Docker Desktop for Windows](https://link.zhihu.com/?target=https%3A//desktop.docker.com/win/stable/Docker%20Desktop%20Installer.exe)，并双击**Docker Desktop Installer.exe**安装[[1\]](https://zhuanlan.zhihu.com/p/351908026#ref_1)

遇到“需升级WSL才能继续安装”的问题

- 参考 [步骤4 下载Linux内核更新包](https://link.zhihu.com/?target=https%3A//docs.microsoft.com/zh-cn/windows/wsl/install-win10)，升级后即可继续安装Docker

## 2. Docker使用

### 2.1 开通阿里云镜像服务

[阿里云容器服务](https://link.zhihu.com/?target=https%3A//cr.console.aliyun.com/) 完成 “创建地址唯一命名空间”，“创建镜像仓库”两项工作[[2\]](https://zhuanlan.zhihu.com/p/351908026#ref_2)。

其中“镜像仓库”需要根据任务选择相应的地区，“镜像仓库”的公网地址需要记住。

### 2.2 本地文件编辑

在需要提交的项目目录下应至少有如下文件

- Dockerfile（用于在阿里云中快速构建docker image）
- run.sh（运行项目的脚本文件）
- *.py（项目代码文件）
- requirements.txt（项目依赖Python包）



以[“AI Earth”人工智能创新挑战赛--AI助力精准气象和海洋预测-天池大赛-阿里云天池](https://link.zhihu.com/?target=https%3A//tianchi.aliyun.com/competition/entrance/531871/introduction)为例

Dockerfile文件为

```text
# Base Images
## 从天池基础镜像构建
FROM registry.cn-shanghai.aliyuncs.com/tcc-public/python:3

## 把当前文件夹里的文件构建到镜像的根目录下（.后面有空格，不能直接跟/）
ADD . /

## 指定默认工作目录为根目录（需要把run.sh和生成的结果文件都放在该文件夹下，提交后才能运行）
WORKDIR /

## Install Requirements（requirements.txt包含python包的版本）
## 这里使用清华镜像加速安装
RUN pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -r requirements.txt

## 镜像启动后统一执行 sh run.sh
CMD ["sh", "run.sh"]
```

run.sh文件为

```text
python test.py # 项目入口代码文件 
```

requirements.txt文件为

```text
numpy==1.19.2
zipfile36==0.1.3
```



### 2.3 Docker推送并提交

在项目目录下通过Powershell进行Docker推送在项目目录下通过Powershell进行Docker推送

> 构建仓库：docker build -t 镜像仓库公网地址:版本号 .
> 验证是否运行正常：**CPU/GPU**镜像：`docker/nvidia-docker run your_image sh run.sh` 
> 推送到仓库：docker push 镜像仓库公网地址:版本号

在[“AI Earth”人工智能创新挑战赛--AI助力精准气象和海洋预测-天池大赛-阿里云天池](https://link.zhihu.com/?target=https%3A//tianchi.aliyun.com/competition/entrance/531871/submission/) 提交并填写相关信息

![image-20210222003614263](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210222003614263.png)

提交完成后在左侧导航栏“我的成绩”查看得分及日志

该提交结果为[“AI Earth”人工智能创新挑战赛--AI助力精准气象和海洋预测-天池大赛-阿里云天池](https://link.zhihu.com/?target=https%3A//tianchi.aliyun.com/competition/entrance/531871/introduction) 比赛提交

具体结果为 返回 ![[公式]](https://www.zhihu.com/equation?tex=%5Cmathcal+N%28%5Cmu%3D0%2C+%5Csigma%3D0.1%29) 的随机数。

## 3. 遇到的问题及其解决方案

### 3.1 推送镜像不成功

> 日志：Error from server (BadRequest): container "balabala" in pod "balabala" is waiting to start: trying and failing to pull image

可以尝试将镜像仓库类型从**私有**改为**公开**

### **3.2 Pycharm中配置Alibaba Cloud Toolkit遇到问题**

> 作者主要遇到的问题是在"**Alibaba Cloud**"下没有“**Deploy to ACR/ACK**”选项

采用Powershell提交的方式来曲线救国

### 3.3 Docker 占用大量C盘空间

> docker在默认在C盘建立镜像，给本不富裕的C盘雪上加霜

方案一：

- wsl --export docker-desktop-data "D:\wsl\docker-desktop-data.tar"
- wsl --export docker-desktop "D:\wsl\docker-desktop.tar"
- wsl --unregister docker-desktop-data
- wsl --unregister docker-desktop
- wsl --import docker-desktop-data D:\wsl\docker-desktop-data\ "D:\wsl\docker-desktop-data\docker-desktop-data.tar" --version 2
- wsl --import docker-desktop D:\wsl\docker-desktop\ "D:\wsl\docker-desktop-data\docker-desktop.tar" --version 2

方案二：使用**LxRunOffline**管理WSL2，具体参考[win10 docker+WSL2 + LxRunOffline](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/xuanmanstein/p/13630416.html)