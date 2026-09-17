---
title: "尝试新方式，重新部署Blog"
date: 2024-10-02
categories: 
  - "study"
---

用到的参考文档，按照遇到问题的顺序：

dockerDocs：[https://docs.docker.com/desktop/install/linux/ubuntu/](https://docs.docker.com/desktop/install/linux/ubuntu/)

docker命令走代理获取镜像源（参考走代理部分）：[https://blog.csdn.net/Lichen0196/article/details/137355517](https://blog.csdn.net/Lichen0196/article/details/137355517)

阮一峰docker教程：[https://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html](https://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

一步到位wordpress：[https://www.cnblogs.com/tdsj/p/16820590.html](https://www.cnblogs.com/tdsj/p/16820590.html)

mysql旧数据迁移：[https://blog.harumonia.moe/sql-frm-myi-myd-files/](https://blog.harumonia.moe/sql-frm-myi-myd-files/)

过了这么久，终于重新把博客部署了。自从电脑坏了修过、换了系统后，博客就再也无法启动了。趁着十一的几天休息，完成了这件事。

第一次部署是纯手工的方式，在Windows系统装了nginx、PHP环境，很久才弄出来大约花了2天的时间。这次用更接近新技术的工具--docker，省时省力，也不会有那么多错误，不过也花了1天。主要原因是第一次使用docker，完全不会，只因系统是Linux，手动装一些deb包很麻烦，命令行下载才是最快、最可靠不会出错的方式，感恩Linux。

最首先得一个困难就是Docker的资源库在中国被屏蔽了，想要的镜像完全没办法访问。真是出师未捷身先死，技术学习上也要面对政治障碍，可悲。装docker简单，直接apt获取，但访问资源却困难重重，看了网上的方法：要么换源，要么用代理。网络上提供的源都存在不稳定性，不知道那一天就面临被封锁。所以我选择了代理，研究了许久，在一片中文文章里学习到，**_如果本地配置好了代理，docker的网络直接设置为localhost就可以了_**，这样可以直连dockerhub拉取镜像资源。

拉取了mysql8和wordpress的镜像，配置、启动很艰难的完成了，最后数据库总是连不上，原来是localhost的问题，在docker里面mysql的ip地址不是121.0.0.1，而是需要通过inspect找出来的。这点耗费了近3小时，因为数据库连不上导致我反复重新配置实例，删完装，装好连不上又删。再就是迁移数据了，之前dump的不是sql文件，导致无法直接运行，需要用mysql5复制文件夹到数据库软件里展示，也花了不少时间。到最终大工告成已是第二天的下午了。

记录下期间提供帮助的文章和相关的指令

```
1.解决apt-get update的错误：在软件管理软件中，取消勾选对应出错的包，对于publickey出错的，解决办法
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys xxxxxxxx

2.使用docker pull时出现404错误：Get "https://registry-1.docker.io/v2/": context deadline exceeded ，原因：dockerhub被ban，解决办法配置代理
$ /etc/systemd/system/docker.service.d/proxy.conf
写入
[service]
Environment="HTTP_PROXY=localhost:7890"
Environment="HTTPS_PROXY=localhost:7890"

3.装好docker后配置：使用非root用户管理docker
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world

4.安装配置wordpress和数据库连接的关键点--数据库的ip地址

      不能在宿主机上进入mysql，先进mysql的docker容器，再进数据库：docker exec -it mysql-cillian bash

    查找容器的IP     ：docker inspect mysql-cillian  |grep 172

   再进数据库：   mysql -h 172.17.0.3 -uroot -p

5.迁移数据，使用mysql5.x，在mysql workbench中导出sql文件，重新导入mysql8.x中
INSERT INTO new_table_name
SELECT * FROM old_table_name
WHERE xxx=xxx
```
