# Superset 搭建和简单使用

[TOC]

# 背景

老板一直反馈说我们给不了他想要的, 关键每次他想要的都不一样, 每实现一个新功能, 他就想要一堆相关的信息, 我要将这些信息都给他做成网页, 用图表展示出来, 各种维度搜索, 这还不得累死这帮开发小伙伴. 所以对于他这种需求, 我基本都予以回绝, 坚决不做. 不过口头说不做, 也深知老板不容易, 他要的信息也不过分, 其实就是要一些数据统计罢啦, 经过一些调研, 一些数据分析部门的朋友建议我用tableau, 我也玩了玩, 功能相当强大, 不过有两个问题:

1. 单机软件
    这种图表总归是希望可以在线查看, 手机查看最完美
2. 付费软件
    不便宜

所以我就一直在寻找开源在线的解决方案, 最开始打算使用grafana, 后来发现他对时序支持比较好, 对于表查询的展示好像怪怪的, 就放弃了. 终于在某天在一个偶然的机会, 发现了Superset这个开源项目. 好东西. 于是乎有了今天的分享

# 简介

> 曾用名Caravel, Panoramix, 是由Airbnb（知名在线房屋短租公司）开源的数据分析与可视化平台, 该工具主要特点是可自助分析, 自定义仪表盘, 分析结果可视化（导出）, 用户/角色权限控制, 还集成了一个SQL编辑器, 可以进行SQL编辑查询等。

# 安装

我使用docker进行安装, 本以为很简单, 中间还是遇到一些坑.

1. 首先安装docker
2. 创建相关目录



```kotlin
mkdir /dockerfs/superset/conf -p
mkdir /dockerfs/superset/data -p
```

1. 创建容器



```jsx
docker run -p 8088:8088 -v /dockerfs/superset/conf:/etc/superset -v mkdir /dockerfs/superset/data:/data  --name superset -d amancevice/superset:0.18.5
```

1. 使用配置文件



```undefined
vi /dockerfs/superset/conf/superset_config.py
```

输入内容



```php
#---------------------------------------------------------
# Superset specific config
#---------------------------------------------------------
ROW_LIMIT = 5000
SUPERSET_WORKERS = 4

SUPERSET_WEBSERVER_PORT = 8088
#---------------------------------------------------------

#---------------------------------------------------------
# Flask App Builder configuration
#---------------------------------------------------------
# Your App secret key
SECRET_KEY = '\2\1thisismyscretkey\1\2\e\y\y\h'

# The SQLAlchemy connection string to your database backend
# This connection defines the path to the database that stores your
# superset metadata (slices, connections, tables, dashboards, ...).
# Note that the connection information to connect to the datasources
# you want to explore are managed directly in the web UI
SQLALCHEMY_DATABASE_URI = 'sqlite:////data/superset.db'

# Flask-WTF flag for CSRF
WTF_CSRF_ENABLED = True

# Set this API key to enable Mapbox visualizations
MAPBOX_API_KEY = ''
```

问题就出现在sqlite的路径上, sqlite默认存储在sqlite:////home/superset/.superset/superset.db, 我这里为了以后升级, 所以切换了存储路径, 这里有两种做法

- 直接将/home/superset/.superset/路径映射出来
- 将/home/superset/.superset/superset.db文件拷贝到/data目录

我这里选择的是第二种, 坑也在这, 使用



```bash
docker exec -it superset /bin/bash
cp /home/superset/.superset/superset.db /data
```

失败, 发现没有权限, ls了一下才发现当前用户是非root用户, 而/data目录是root权限.
 经过一番查找, 发现可以使用以下命令用root账号登陆容器



```bash
docker exec -u 0 -it superset /bin/bash
```

0号用户就是root用户, 剩下来的就简单了



```kotlin
mv /home/superset/.superset/superset.db /data
```

1. 退出容器, 重启容器, 然后进行用户初始化



```bash
docker restart superset 
docker exec -it superset superset-init
```



![img](https:////upload-images.jianshu.io/upload_images/7322003-3395439fe5b7bf70.png?imageMogr2/auto-orient/strip|imageView2/2/w/916/format/webp)

init pwd.png

1. 打开浏览器, 键入地址, 可以使用起来了



![img](https:////upload-images.jianshu.io/upload_images/7322003-da3f10f91f3e7933.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

login.png



![img](https:////upload-images.jianshu.io/upload_images/7322003-2449c39221d985c3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

dashboard.png

# 使用

1. 配置数据源



![img](https:////upload-images.jianshu.io/upload_images/7322003-e06300b81a60e65c.png?imageMogr2/auto-orient/strip|imageView2/2/w/754/format/webp)

datasources.png





![img](https:////upload-images.jianshu.io/upload_images/7322003-54dfb4783727c3ae.png?imageMogr2/auto-orient/strip|imageView2/2/w/1172/format/webp)

connection.png


**注意**



1. 添加要展示的表
2. 创建slice, 然后将创建好的slice加入到dashboard
    我感觉这部分不难, 自己摸索摸索总归能够用起来, 我这里就不详细说了
3. 多表展示
    这里要特别强调一下如何显示多表的展示, 很多文章都说superset不支持多表, 只支持单表, 我刚开始也以为是这样, 后来发现这个方法可以进行基于多表的展示.
    首先记得数据库配置里先勾选"Expose in SQL Lab", 要不然在SQL Lab中是找不到数据源的



![img](https:////upload-images.jianshu.io/upload_images/7322003-e09e85a8f3776a45.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Expose in SQL lab.png



![img](https:////upload-images.jianshu.io/upload_images/7322003-58c364011581c286.png?imageMogr2/auto-orient/strip|imageView2/2/w/491/format/webp)

SQL lab.png

先执行一个语句, 注意查询结果中不要有相同的列, 如果有, 后续会提示错误



![img](https:////upload-images.jianshu.io/upload_images/7322003-d7a4290cf792fb45.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

query.png



点击"Query History", 选择Visualize





![img](https:////upload-images.jianshu.io/upload_images/7322003-1ac728c4f4aa7891.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Visualize.png



勾中一个, 会有以下效果, 我基本都是用默认属性, 然后点击最下方按钮





![img](https:////upload-images.jianshu.io/upload_images/7322003-a5ede48317cf47af.png?imageMogr2/auto-orient/strip|imageView2/2/w/927/format/webp)

modify.png


 他会跳到slice的编辑页, 可以进行编辑, 其实他这个过程是创建了一个以查询为结果的临时表, 后续就是在这个临时表中做展示. 知道这个原理, 也可以通过直接添加表来进行操作, 只是操作流程不允许写入sql, 可以先添加, 然后再填入sql(我没尝试过, 应该行得通) 



![img](https:////upload-images.jianshu.io/upload_images/7322003-579914598dc0e30e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

table detail.png



最终结果如下



![img](https:////upload-images.jianshu.io/upload_images/7322003-a8d463aa9a1f7ba0.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

result.png

# 结语

文章写的有点啰嗦, 如果你有数据可视化的问题, 希望这篇文章能够给予你一定的帮助, 目前我观察下来, 这部分做的好的软件不多, 很多都是靠卖服务赚钱的, 比如数据观(人家做的真好, 其实挺鼓励使用人家的服务的, 如果没什么研发人员, 可以优先考虑使用), 希望这个软件可以满足老板的部分需求. 据说这里面的图表还可以嵌入到其他系统中, 没有仔细研究过, 不过这种不易调优的查询, 最好还是临时用用就好, 别嵌入到系统中, 稳定性和性能都不能有所保障.

最后还是附几张人家的图表截图吧



![img](https:////upload-images.jianshu.io/upload_images/7322003-b45065d311e9f882.png?imageMogr2/auto-orient/strip|imageView2/2/w/1098/format/webp)

spanshot1.png



![img](https:////upload-images.jianshu.io/upload_images/7322003-4b289fe10a6b4909.png?imageMogr2/auto-orient/strip|imageView2/2/w/1106/format/webp)

spanshot2.png



![img](https:////upload-images.jianshu.io/upload_images/7322003-3ef94e862dbe3d3e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1113/format/webp)

spanshot3.png