---
title: Docker 学习笔记
tags: [docker]
date: 2020-01-10 15:11:02
updated: 2020-01-10 15:11:02
categories: docker
---

## 初体验

<!-- more -->

安装一个 nginx 并运行
执行命令

```sh
docker run -d -p 8080:80 nginx
```

查看当前运行的进程

```sh
docker ps
```

访问`http://localhost:8080`

执行原理

1. 检测本地是否有 nginx 镜像
2. 如果没有，则去仓库下载最新的镜像
3. 下载成功后，运行镜像
4. 把本地的8080端口映射到容器的80端口

运行ubuntu容器，并进入`bash`控制台

```sh
docker run -it ubuntu /bin/bash
```

* -t：分配一个伪终端并绑定到容器标准输入输出上
* -i：让容器的标准输入保持打开，可以一直监听输入

## 基本概念

### 镜像

相当于操作系统安装文件，提供容器所需要的程序，依赖库，资源，配置等信息

镜像可以嵌套镜像，一个镜像可以由多个镜像组成

### 容器

相当于一个独立的操作系统，拥有特定的运行环境，镜像与容器类似于类和实例的关系，容器的本质是进程

### 仓库

集中管理和分发给用户下载的镜像的服务

## 基础实战

### 镜像

1. 搜索镜像

    ```sh
    docker search nginx
    ```

2. 下载镜像，(如果本地不存在)

    ```sh
    docker pull fuhai/jpress:v2.0.7
    ```

3. 查看镜像

    ```sh
    docker images
    ```

4. 删除镜像

    ```sh
    docker rmi fuhai/jpress:v2.0.7
    ```

5. 运行镜像

    ```sh
    docker run
    ```

6. 查看命令

    ```sh
    docker run --help
    ```

7. 实例

    ```sh
    docker run -d -p 8080:80 nginx
    ```

    -p：端口映射
    -d：后台运行，并返回ContainerId，不加的话是直接运行

8. 停止容器

    ```sh
    # 停止容器
    docker stop <container_id>

    # 启动容器
    docker start <container_id>

    # 暂停容器
    docker pause <container_id>

    # 继续已经暂停的容器
    docker unpause <container_id>

    # 重启容器
    docker restart <container_id>
    ```

9. 容器操作

    ```sh
    # 列出容器
    docker ps

    # 查看容器详细信息
    docker inspect <container_id>

    # 删除容器
    docker rm <container_id>
    ```

10. 进入容器

    ```sh
    docker exec -it <container_id> bash
    ```

11. 拷贝文件

    ```sh
    # 从本地拷贝到容器
    docker cp /srcdir/srcfile <container_id>:/usr/local/webapp/...

    # 从容器拷贝到本地
    docker cp <container_id>:/usr/local/webapp/... /local/testdir/...
    ```

12. 把容器做成镜像

    ```sh
    docker commit <conainer_id> mynginx:v1.0
    ```
