---
title: Docker安装分布式组件
date: 2021-11-25 21:01:59
categories:
- 开发工具
tags:
- 架构师
- Docker
---

> Docker容器中安装MySQL、Redis、Nginx、RabbitMQ、MongoDB、Elasticsearch、Logstash、Kibana，基于CenterOS7.6。

<!-- more -->

**1.Docker环境安装**

- 安装yum-utils：

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

- 为yum源添加docker仓库位置：

```shell
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

- 安装docker：

```shell
yum install docker-ce
```

- 启动docker：

```shell
systemctl start docker
```

---

**2.Java安装**

```shell
docker pull java:8
```

```shell
/**
** run 启动一个镜像容器
** -d 后台运行该容器
** -I 以交互模式运行容器，通常与 -t 同时使用
** -t 为容器重新分配一个伪输入终端
*/
docker run -d -it --name java java:8
```



**3.MySQL安装**

- 下载MySQL5.7的docker镜像：

```shell
docker pull mysql:5.7
```

- 使用如下命令启动MySQL服务：

```shell
docker run -p 3306:3306 --name=mysql \
-v /data/mysql/log:/var/log/mysql \
-v /data/mysql/data:/var/lib/mysql \
-v /data/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7
```

- 参数说明
  - -p 3306:3306：将容器的3306端口映射到主机的3306端口
  - -v /data/mysql/conf:/etc/mysql：将配置文件夹挂在到主机
  - -v /data/mysql/log:/var/log/mysql：将日志文件夹挂载到主机
  - -v /data/mysql/data:/var/lib/mysql/：将数据文件夹挂载到主机
  - -e MYSQL_ROOT_PASSWORD=root：初始化root用户的密码
- 进入运行MySQL的docker容器：

```shell
docker exec -it mysql /bin/bash
```

- 使用MySQL命令打开客户端：

```shell
mysql -uroot -proot --default-character-set=utf8
```

- 创建mall数据库：

```shell
create database mall character set utf8
```

- 安装上传下载插件，并将document/sql/mall.sql上传到Linux服务器上：

```shell
yum -y install lrzsz
```

- 将mall.sql文件拷贝到mysql容器的/目录下：

```shell
docker cp /mydata/mall.sql mysql:/
```

- 将sql文件导入到数据库：

```shell
use mall; source /mall.sql;
```

- 创建一个reader:123456帐号并修改权限，使得任何ip都能访问：

```shell
grant all privileges on *.* to 'reader' @'%' identified by '123456';
```

---

**4.Redis安装**

- 下载Redis5.0的docker镜像：

```shell
docker pull redis:5
```

- 使用如下命令启动Redis服务：

```shell
docker run -p 6379:6379 --name=redis \
-v /data/redis/data:/data \
-d redis:5 redis-server --appendonly yes
```

- 进入Redis容器使用redis-cli命令进行连接：

```shell
docker exec -it redis redis-cli
```

![redis](/Users/wustmz/Documents/redis.jpg)

---

**5.Nginx安装**

- 下载Nginx1.10的docker镜像：

```shell
docker pull nginx:1.10
```

- 先运行一次容器（为了拷贝配置文件）：

```shell
docker run -p 80:80 --name=nginx \
-v /data/nginx/html:/usr/share/nginx/html \
-v /data/nginx/logs:/var/log/nginx  \
-d nginx:1.10
```

- 将容器内的配置文件拷贝到指定目录：

```shell
docker container cp nginx:/etc/nginx /data/nginx/
```

- 修改文件名称：

```shell
mv nginx conf
```

- 终止并删除容器：

```shell
docker stop nginx docker rm nginx
```

- 使用如下命令启动Nginx服务：

```shell
docker run -p 80:80 --name=nginx \
-v /data/nginx/html:/usr/share/nginx/html \
-v /data/nginx/logs:/var/log/nginx  \
-v /data/nginx/conf:/etc/nginx \
-d nginx:1.10

```

---

**6.RabbitMQ安装**

- 下载rabbitmq3.7.15的docker镜像：

```shell
docker pull rabbitmq:3.7.15
```

- 使用如下命令启动RabbitMQ服务：

```shell
docker run -p 5672:5672 -p 15672:15672 --name=rabbitmq \ -d rabbitmq:3.7.15
```

- 进入容器并开启管理功能：

```shell
docker exec -it rabbitmq /bin/bash 
```

```shell
rabbitmq-plugins enable rabbitmq_management
```

![开启管理功能](https://raw.githubusercontent.com/wustmz/oss/main/img/rabbitmq1.png)

- 开启防火墙：
  - 查看开启端口列表`firewall-cmd --list-ports`
  - 查看firewalld状态:`systemctl status firewalld`，如果是dead状态，即防火墙未开启
  - 开启防火墙`systemctl start firewalld`
  - 确认firewalld状态:`systemctl status firewalld`
  


```shell
firewall-cmd --zone=public --add-port=15672/tcp --permanent firewall-cmd --reload
```

- 访问地址查看是否安装成功：[http://192.168.172.111:15672](http://192.168.172.111:15672/)

![mqlogin](/Users/wustmz/Documents/mqlogin.png)

- 输入账号密码并登录：guest guest
- 创建帐号并设置其角色为管理员：mall mall

![add-user](https://raw.githubusercontent.com/wustmz/oss/main/img/add-user.png)

- 创建一个新的虚拟host为：/mall

![add-host](https://raw.githubusercontent.com/wustmz/oss/main/img/add-host.png)

- 点击mall用户进入用户配置页面

![user-admin](https://raw.githubusercontent.com/wustmz/oss/main/img/user-admin.png)

- 给mall用户配置该虚拟host的权限

![add-permission](https://raw.githubusercontent.com/wustmz/oss/main/img/add-permission.png)

---



**7.Elasticsearch安装**

- 下载Elasticsearch7.6.2的docker镜像：

```shell
docker pull elasticsearch:7.6.2
```

- 修改虚拟内存区域大小，否则会因为过小而无法启动:

```shell
sysctl -w vm.max_map_count=262144              
```

- 创建文件夹

```shell
mkdir -p /data/elasticsearch/config 
mkdir -p /data/elasticsearch/data 
echo "http.host: 0.0.0.0" >/data/elasticsearch/config/elasticsearch.yml 
chmod -R 777 /data/elasticsearch/ 
```

- 使用如下命令启动Elasticsearch服务：

```shell
docker run -p 9200:9200 -p 9300:9300 --name elasticsearch \
-e "discovery.type=single-node" \
-e "cluster.name=elasticsearch" \ 
-v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-v /data/elasticsearch/data:/usr/share/elasticsearch/data \
-d elasticsearch:7.6.2 
```

- 启动时会发现报错如下

```shell
docker: Error response from daemon: driver failed programming external connectivity on endpoint elasticsearch (b4ed5b4df7d4d6c9847f111162ffb9bd34cd2fc3cd11fe047a72e3ca1175a0fb):  (iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 9300 -j DNAT --to-destination 172.17.0.6:9300 ! -i docker0: iptables: No chain/target/match by that name.
 (exit status 1)).
```

- 通过[Stack Overflow](https://stackoverflow.com/questions/31667160/running-docker-container-iptables-no-chain-target-match-by-that-name)解决报错后重启docker

```shell
iptables -t filter -F
iptables -t filter -X
```

```
systemctl restart docker
```

- 使用上述命令重启es，`docker ps`查看进程，有如下信息说明启动成功

```shell
[root@root elasticsearch]# docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                                                                                  NAMES
743f299c8c90   elasticsearch:7.6.2   "/usr/local/bin/dock…"   7 seconds ago   Up 5 seconds   0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 0.0.0.0:9300->9300/tcp, :::9300->9300/tcp   elasticsearch
```

- 安装中文分词器IKAnalyzer，并重新启动：

```shell
docker exec -it elasticsearch /bin/bash  
```

```shell
#此命令需要在容器中运行 
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.6.2/elasticsearch-analysis-ik-7.6.2.zip 

```

```shell
docker restart elasticsearch
```

- 开启防火墙：

```shell
firewall-cmd --zone=public --add-port=9200/tcp --permanent firewall-cmd --reload
```

- 访问会返回版本信息：[http://192.168.172.111:9200](http://192.168.172.111:9200/)

![es](https://raw.githubusercontent.com/wustmz/oss/main/img/es.png)

---

**8.Logstash安装**

- 下载Logstash7.6.2的docker镜像：

```shell
docker pull logstash:7.6.2
```

- 创建/data/logstash目录

```shell
mkdir /data/logstash
```

- 创建配置文件logstash.conf在该目录

```shell
input {
  file {
    #标签
    type => "systemlog-localhost"
    #采集点
    path => "/var/log/messages"
    #开始收集点
    start_position => "beginning"
    #扫描间隔时间，默认是1s，建议5s
    stat_interval => "5"
  }
}

output {
  elasticsearch {
    hosts => "es:9200"
    index => "logstash-system-localhost-%{+YYYY.MM.dd}"
 }
}
```

- 使用如下命令启动Logstash服务；

```shell
docker run -p 4560:4560 -p 4561:4561 -p 4562:4562 -p 4563:4563 --name logstash \
--link elasticsearch:es \
-v /data/logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf \
-d logstash:7.6.2
```

- 进入容器内部，安装`json_lines`插件

```shell
docker exec -it logstash /bin/bash 
```

```shell
logstash-plugin install logstash-codec-json_lines
```

---

**9.Kibana安装**

- 下载Kibana7.6.2的docker镜像：

```shell
docker pull kibana:7.6.2
```

- 使用如下命令启动Kibana服务：

```shell
docker run --name kibana -p 5601:5601 \
--link elasticsearch:es \
-e "elasticsearch.hosts=http://es:9200" \
-d kibana:7.6.2
```

开启防火墙：

```shell
firewall-cmd --zone=public --add-port=5601/tcp --permanent
```

重启防火墙：

```shell
firewall-cmd --reload
```

- 访问地址进行测试：[http://192.168.172.111:5601](http://192.168.172.111:5601/)

![kibana](https://raw.githubusercontent.com/wustmz/oss/main/img/kibana.png)

---

**10.MongoDB安装**

- 下载MongoDB4.2.5的docker镜像：

```shell
docker pull mongo:4.2.5
```

- 使用docker命令启动：

```shell
docker run -p 27017:27017 --name mongo \
-v /data/mongo/db:/data/db \
-d mongo:4.2.5
```
