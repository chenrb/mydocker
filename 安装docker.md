## 安装docker

[官网](https://docs.docker.com/engine/install/centos/)
[菜鸟](https://www.runoob.com/docker/centos-docker-install.html)

可以使用yum安装或下载官方提供的安装脚本一键安装

# 脚本一键安装
```
curl -fsSL get.docker.com -o docker_install.sh
bash docker_install.sh --mirror Aliyun
```

## 版本
```
centos 7.6
Docker version 20.10.12, build e91ed57
```

# 加入systemd管理&#启动docker
```
systemctl enable docker
systemctl start docker
```

# 测试  docker run hello-world
```
[root@chenrb /]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete 
Digest: sha256:97a379f4f88575512824f3b352bc03cd75e239179eea0fecc38e597b2209f49a
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

# 镜像加速
# 使用阿里镜像：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors，获取url
```
1. 安装／升级Docker客户端
推荐安装1.10.0以上版本的Docker客户端，参考文档docker-ce

2. 配置镜像加速器
针对Docker客户端版本大于 1.10.0 的用户

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["{{url}}"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

