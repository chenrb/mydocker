# mydocker
存储docker相关脚本和笔记
## 简介
内存爆炸，服务挂了，重启后部分组件挂了，重装失败，所以做一个容器镜像，以应对日后状况。

[安装docker](安装docker.md)
[安装docker Compose](安装docker.md)

## 常见docker命令
### 帮助启动类命令
```
启动docker: systemctl start docker
停止docker: systemctl stop docker
重启docker: systemctl restart docker
查看docker状态: systemctl status docker
开启启动: systemctl enable docker
查看docker概要信息: docker info
查看docker总体帮助稳定: docker --help
查看docker命令帮助文档: docker 具体命令 --help
```
### 镜像命令
```
docker images
docker search xxx
docker pull xxx
docker system df 查看空间
docker rm xxxID
仓库名/tag都是none的镜像，都是虚悬镜象
```
### 容器命令
```
docker run
-i: 交互式命令
-t: 终端
-d: 后台运行
# 容器内
exit    # 退出容器停止
Ctrl+p+q  # 退出容器不停止

docker logs ID  # 查看日志
docker top ID   # 查看容器内运行进程
docker inspect ID  # 查看容器内部细节
docker exec -it ID xxx  # 进入容器 --help

docker cp ID:/path path  # 拷贝容器内数据
docker export ID > abcd.tar  # 导出镜像
cat XXX.tar | docker import 
```

### 镜像分层
镜像可以通过分层来进行继承，基于基础镜像，可以制作各种具体的应用镜像。
镜像又一层层文件系统组成
只需要内核，最小单元，所以镜像大小小
资源共享，分层复用，多个需要同一层，只需要加载一层就可以复用了。
### commit
docker commit 提交容器副本使之成为一个新的镜像

### 容器数据卷
```
--privileged=true   # 解决挂载目录没有权限问题 
解决数据持久化问题
docker run -it --privileged=true -v /宿主path:/容器path
:rw可读写   :ro只读
宿主机与容器内互通，共享
数据卷规则可继承
```
### mysql
```
docker run -p 3306:3306 -d --name mysql5.7 --env-file /data/mysql/env --privileged=true -v /data/mysql/conf/:/etc/mysql/conf.d -v /data/mysql/log:/var/log/mysql -v /data/mysql/data:/var/lib/mysql mysql:5.7
```
### redis
```
docker run -p 6379:6379 -d --name redis --privileged=true -v /data/redis/conf/redis.conf:/etc/redis/redis.conf -v /data/redis/data:/data redis:latest redis-server /etc/redis/redis.conf
```
### nginx
```
1、先启动一个nginx：docker run -p 80:80 -p 443:443 --name nginx --privileged=true --restart=always -d nginx
2、复制/usr/share/nginx/html至宿主机
3、复制/etc/nginx/conf.d文件夹至宿主机
4、销毁容器
docker run -p 80:80 -p 443:443 --name nginx --network bridge --privileged=true --restart=always -v /data/nginx/conf/conf.d:/etc/nginx/conf.d -v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /data/nginx/html:/usr/share/nginx/html -v /data/nginx/logs:/var/log/nginx -v /data/code/share/static:/usr/share/nginx/html/static -v /data/code/share/media:/usr/share/nginx/html/media -d nginx
```
```markdown
docker run -p 8000:8000 --name share --network bridge --privileged=true --env-file /data/code/env -v /data/code/share:/data/code/share -v/data/code/logs/:/data/code/logs/ -it -d share
```
docker run -d -p 15672:15672 -p 5672:5672 --env-file /data/rabbitmq/env --name rabbitmq --network bridge rabbitmq:management