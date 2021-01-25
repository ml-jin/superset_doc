##  Superset 概要介绍

Apache Superset是一个可用于数据展示与数据可视化的开源软件，在处理大量数据方面效果显著。Superset最招为Airbnb所开发，在2017年成为Apache的孵化项目。

# 概要信息

Apache Superset的概要信息如下表所示：

| 项目             | 说明                                              |
| ---------------- | ------------------------------------------------- |
| 官网             | https://superset.apache.org/                      |
| 开源/闭源        | 开源                                              |
| 源码管理地址     | https://github.com/apache/incubator-superset/wiki |
| License类别      | Mozilla Public License 2.0                        |
| 开发语言         | Go                                                |
| 操作系统支持     | 跨平台，支持多种操作系统                          |
| 当前稳定版本     | 0.36 （2020/04/17）                               |
| github的star数量 | 3万左右(2020/09/16)                               |

# 主要特性

Apache Superset是一款快速直观的轻量级工具，具有丰富的功能选项，各种用户都可以轻松地以可视化的方式浏览数据，从简单的折线图到高度详细的地理空间图，Apache Superset无所不能，提供了如下主要特性：

- 强大易用：可以快速容易地集成，从而浏览数据，而这一切通过SQL IDE或者无需编写代码，通过可视化构建器即可完成。
- 支持多种数据库：可以通过SQL Alchemy连接到任何基于SQL的数据源，包括云原生的数据库以及PB级的数据引擎。
- 架构设计：Superset轻巧且极具可扩展性，它可以利用既存的数据基础框架而不需要另外一个接收层。
- 丰富的可视化方式与仪表盘：Superset提供了多种精美的可视化效果。可视化插件体系结构使得构建自定义的可视化变得更为容易。
- 支持多种数据库：比如
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916222012717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdW1pYW9jbg==,size_16,color_FFFFFF,t_70#pic_center)

# 安装

本文使用最为简单的方式进行Apache Superset的安装与环境准备，首先事前准备docker和docker-compose：

```
liumiaocn:~ liumiao$ docker-compose version
docker-compose version 1.24.1, build 4667896b
docker-py version: 3.7.3
CPython version: 3.6.8
OpenSSL version: OpenSSL 1.1.0j  20 Nov 2018
liumiaocn:~ liumiao$ docker version
Client: Docker Engine - Community
 Version:           19.03.5
 API version:       1.40
 Go version:        go1.12.12
 Git commit:        633a0ea
 Built:             Wed Nov 13 07:22:34 2019
 OS/Arch:           darwin/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.5
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.12
  Git commit:       633a0ea
  Built:            Wed Nov 13 07:29:19 2019
  OS/Arch:          linux/amd64
  Experimental:     true
 containerd:
  Version:          v1.2.10
  GitCommit:        b34a5c8af56e510852c35414db4c1f4fa6172339
 runc:
  Version:          1.0.0-rc8+dev
  GitCommit:        3e425f80a8c931f88e6d94a8c831b9d5aa481657
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
liumiaocn:~ liumiao$ 
12345678910111213141516171819202122232425262728293031323334
```

## 步骤1: 获取docker-compose.yml文件

> 执行命令：git clone https://github.com/apache/incubator-superset

## 步骤2: 启动服务

> 执行命令：cd incubator-superset && docker-compose up -d

```
liumiaocn:~ liumiao$ cd incubator-superset && docker-compose up -d
...省略
Pulling redis (redis:3.2)...
...省略
Pulling db (postgres:10)...
...省略
 ---> 5e7667de08c2
Step 2/6 : COPY ./requirements/*.txt ./docker/requirements-*.txt /app/requirements/
 ---> 71d6a9aeddb2
Step 3/6 : COPY ./setup.py ./MANIFEST.in /app/
 ---> aeb1c1f7a740
 ...省略
 Pulling superset-node (node:12)...
 ...省略
Creating superset_db    ... done
Creating superset_cache ... done
Creating superset_tests_worker ... done
Creating superset_node         ... done
Creating superset_app          ... done
Creating superset_worker       ... done
Creating superset_init         ... done
liumiaocn:incubator-superset liumiao$
12345678910111213141516171819202122
```

## 步骤3: 结果确认

```
liumiaocn:incubator-superset liumiao$ docker-compose ps
        Name                       Command                       State                        Ports              
-----------------------------------------------------------------------------------------------------------------
superset_app            /usr/bin/docker-entrypoint ...   Up (healthy)            8080/tcp, 0.0.0.0:8088->8088/tcp
superset_cache          docker-entrypoint.sh redis ...   Up                      127.0.0.1:6379->6379/tcp        
superset_db             docker-entrypoint.sh postgres    Up                      127.0.0.1:5432->5432/tcp        
superset_init           /usr/bin/docker-entrypoint ...   Up (health: starting)   8080/tcp                        
superset_node           docker-entrypoint.sh bash  ...   Up                                                      
superset_tests_worker   /usr/bin/docker-entrypoint ...   Exit 1                                                  
superset_worker         /usr/bin/docker-entrypoint ...   Up (health: starting)   8080/tcp                        
liumiaocn:incubator-superset liumiao$
1234567891011
```

# 结果确认

使用如下地址登录Apache Superset

> 登录地址：http://localhost:8088/

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916223734253.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdW1pYW9jbg==,size_16,color_FFFFFF,t_70#pic_center)
使用admin/admin然后点击Sign in按钮进行登录即可

# 参考内容

https://superset.apache.org/
https://github.com/apache/incubator-superset