### 是什么
用来构建docker镜像的文本文件，是由一条条构建镜像所需的指令和参数构成的脚本文件
### 过程
```markdown
1、编写DockerFile文件  
    1、保留字必须是大写字母，且至少带一个参数
    2、从上到下，顺序执行
    3、#表示注释
    4、每条指令都会创建一个新的镜像层并对镜像进行提交
2、构建镜像  
3、运行镜像  
```
### docker执行Dockerfile的大致流程
```markdown
1、docker从基础镜像运行一个容器
2、执行一条指令并对容器作出修改
3、执行类似的docker commit的操作提交一个新的镜像
4、docker再基于刚提交的镜像运行一个新容器
5、执行dockerfile中的下一条指令知道所有指令执行完
```
### docker体系
```markdown
1、Dockerfile面向开发
2、Docker镜像成为交付标准
3、Docker容器涉及部署与运维
```
### 常见保留字
```markdown
# 基础镜像来自
FROM image

# 镜像维护者的名字和邮箱地址
MAINTAINER

# 容器构建的时候会运行的命令
# shell/exec
# docker在构建的时候运行，build时
RUN

# 当前容器对外暴露的端口
EXPOSE

# 指定在创建容器后，终端默认登录的进来工作目录，一个落脚点
WORKDIR

# 指定该镜像以什么用户去执行，默认root
USER

# 用来在构建镜像的时候使用的环境变量
ENV

# 容器数据卷，用于数据保存和持久化工作
VOLUME

# 将宿主机目录下的文件拷贝进镜像且会自动处理URL和解压tar压缩包
ADD
# 拷贝
COPY

# 启动后需要运行的事情，格式与RUN相似
# 可以有多个，只有最后一个生效
# 在docker run的时候运行
CMD

# 定参数，与CMD联合使用
ENTRYPOINT
```
### 虚悬镜像
```markdown
仓库和TAG都是NONE的镜像
错误的镜像
docker image prune
```