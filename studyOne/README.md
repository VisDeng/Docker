# Docker学习

文档地址：https://docs.docker.com/engine/  

## Docker的基本组成

![Snipaste_2021-02-08_20-02-32](images\Snipaste_2021-02-08_20-02-32.png)

## 镜像（image）：

docker镜像就好比是一个模板，可以通过这个模板来创建容器服务，tomcat镜像 ===> run ===> tomcat1容器（提供服务器），通过这个镜像可以创建多个容器（最终服务运行或者项目运行就是在容器中的）

## 容器（container）：

Docker利用容器技术，独立运行一个或者一个组应用，通过镜像来创建的。

启动，停止，删除，基本命令。

目前就是可以把这个容器理解为就是一个简易的linux系统。

## 仓库（reponsitory）：

仓库就是存放镜像的地方。

仓库分为公有仓库和私有仓库。

Docker HUb （默认是国外的）。

阿里云...都有容器服务器（配置镜像加速）。

## 安装Docker

> 环境准备

1、会一点点的Linux的基础

2、CentOS 7

3、服务器连接工具

> 环境查看

![Snipaste_2021-02-08_20-38-35](images\Snipaste_2021-02-08_20-38-35.png)

![Snipaste_2021-02-08_20-41-49](images\Snipaste_2021-02-08_20-41-49.png)

> 安装

帮助文档

```shell
# 1、先卸载原Docker
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
# 2、需要的安装包
yum install -y yum-utils
# 3、设置镜像的仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo # 官方默认，不推荐使用
    
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo # 阿里云，推荐使用

# 更新 yum 软件包索引（最好）
yum makecache fast

# 4、安装 docker docker-ce 社区    ee 企业版
yum install docker-ce docker-ce-cli containerd.io

# 5、启动 docker
systemctl start docker
# 6、使用docker version 查看版本号
docker version
```

```shell
# 7、测试 hello word
docker run hello-world
# 7.1（附加）、如果报错 error pulling image configuration: （时间不同步）
执行一下命令：ntpdate time.windows.com
如遇到命令不存在： yum -y install netdate
```

![docker-run-helloword](F:\Docker\docker-run-helloword.png)

```shell
# 查看镜像
[root@VM-0-4-centos /]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    bf756fb1ae65   13 months ago   13.3kB
```

```shell
# 卸载docker
yum remove docker-ce docker-ce-cli containerd.io # 删除依赖
rm -rf /var/lib/docker  # 删除目录
```

## 配置阿里云镜像加速

https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://2lrywg60.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload 
sudo systemctl restart docker
```

## Run 的流程

![run流程](images\run流程.png)



## 底层原理

### Docker是怎么工作的？

Docker是一个 Client - Server 结构的系统，Docker的守护进程运行在主机上，通过 Socket 从客户端访问。

DockerServer 接收到 Docker-Clinet 的指令，就会执行这个命令。

![docker是怎么样工作的](images\docker是怎么样工作的.png)

### Docker为什么比 VM 快？

1、Docker有着比虚拟机更少的抽象层。

2、Docker 利用的是宿主机的内核，vm需要的是 Guest OS。

![Snipaste_2021-02-09_21-04-24](images\Snipaste_2021-02-09_21-04-24.png)

所以说，新建一个容器的时候，doker不需要像虚拟机一样重新加载一个操作系统内核，避免引导。虚拟机是加载 Guest OS，分钟级别的。而 Docker 是利用宿主机的操作系统，省略了这个复杂的过程，秒级。

> 学习完毕所有命令，再回顾理论，就会很清晰。

## Docker的常用命令

### 帮助命令

```shell
docker version    # 查看版本信息
docker info 	  # 显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help # 帮助命令
```

帮助文档的地址：https://docs.docker.com/reference/

### 镜像命令

**`docker images` : 查看所有镜像**

```shell
[root@VM-0-4-centos ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    bf756fb1ae65   13 months ago   13.3kB

REPOSITORY： 镜像的仓库源
TAG：		镜像的标签
IMAGE ID：	镜像的ID
CREATED：	镜像的创建时间
SIZE：		镜像的大小

# 可选项
-a, --all     # 列出所有镜像
-q, --quiet   # 只显示镜像id
```

**`docker searsh` 搜索镜像**

```shell
#docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   10475
mariadb                           MariaDB is a community-developed fork of MyS…   3896
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   769  
percona                           Percona Server is a fork of the MySQL relati…   526 
centos/mysql-57-centos7           MySQL 5.7 SQL database server                   87        

# 可选项
-filter=STARS=3000  #搜索出来的镜像就是 stars 大于 3000
[root@VM-0-4-centos ~]# docker search mysql --filter=STARS=3000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   10475     [OK]       
mariadb   MariaDB is a community-developed fork of MyS…   3896      [OK]   
```

`docker pull` **下载镜像**

```shell
# 下载镜像 docker pull 镜像名 [:tag]
[root@VM-0-4-centos ~]# docker pull mysql
Using default tag: latest # 如果不写 tag，默认就是latest
latest: Pulling from library/mysql
a076a628af6f: Pull complete  # 分层下载，docker image 的核心，联合核减系统
f6c208f3f991: Pull complete 
88a9455a9165: Pull complete 
406c9b8427c6: Pull complete 
7c88599c0b25: Pull complete 
Digest: sha256:feada149cb8ff54eade1336da7c1d080c4a1c7ed82b5e320efb5beebed85ae8c # 签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest # 真实地址

# 等价于它
docker pull mysql
Status: Downloaded newer image for mysql:latest

# 制定版本下载
[root@VM-0-4-centos ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
# Already exists  已经存在的可以共用（牛逼）
a076a628af6f: Already exists
1831ac1245f4: Pull complete 
Digest: sha256:b3d1eff023f698cd433695c9506171f0d08a8f92a0c8063c1a4d9db9a55808df
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7

# 查看所有下载好的镜像
[root@VM-0-4-centos ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
mysql         5.7       a70d36bc331a   3 weeks ago     449MB
mysql         latest    c8562eaf9d81   3 weeks ago     546MB
hello-world   latest    bf756fb1ae65   13 months ago   13.3kB
```

`docker rmi` **删除镜像**

```shell
# docker rmi 镜像id  根据id删除镜像
# docker rmi 镜像id 镜像id  根据多个id删除镜像
[root@VM-0-4-centos ~]# docker rmi c8562eaf9d81
Untagged: mysql:latest
Untagged: mysql@sha256:feada149cb8ff54eade1336da7c1d080c4a1c7ed82b5e320efb5beebed85ae8c

# 骚操作 递归删除所有镜像
# docker rmi -f $(docker images -aq)
Untagged: mysql:5.7
Untagged: mysql@sha256:b3d1eff023f698cd433695c9506171f0d08a8f92a0c8063c1a4d9db9a55808df
Deleted: sha256:a70d36bc331a13d297f882d3d63137d24b804f29fa67158c40ad91d5050c39c5
Untagged: hello-world:latest
Untagged: hello-world@sha256:31b9c7d48790f0d8c50ab433d9c3b7e17666d6993084c002c2ff1ca09b96391d
Deleted: sha256:bf756fb1ae65adf866bd8c456593cd24beb6a0a061dedf42b26a993176745f6b
```

### 容器命令

**说明：我们有了镜像才可以创建容器，Linux，下载一个 centos 镜像来学习。**

```shell
docker pull centos
```

**新建容器并启动**

```shell
docker run [可选参数] image

# 参数说明
--name="Name"  	容器名字，tomcat01  tomcat02 用来区分容器
-d     			后台方式运行
-it				使用交互方式运行，进入容器查看内容
-p				制定容器端口 -p 8080:8080
		-p ip 主机端口：容器端口
		-p 主机端口：容器端口（常用）
		-p 容器端口
		容器端口
-P				随机制定端口

# 测试，启动并进入容器
[root@VM-0-4-centos ~]# docker run -it centos /bin/bash
[root@911c6a4376a6 /]# ls # 查看容器内的 centos 很多命令都是不完善的
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
# 从容器中退回主机
[root@911c6a4376a6 /]# exit
exit
```

**列出所有运行中的容器**

```shell
[root@VM-0-4-centos ~]# docker ps
-a		# 列出当前正在运行中的容器 + 带出历史运行过的容器
-n=?	# 显示最新创建的容器
-q		# 只显示容器的编号
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@VM-0-4-centos ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED              STATUS                      PORTS     NAMES
911c6a4376a6   centos         "/bin/bash"   About a minute ago   Exited (0) 58 seconds ago             frosty_mclaren
c53e78ce61bd   bf756fb1ae65   "/hello"      12 hours ago         Exited (0) 12 hours ago               thirsty_fermat
```

**退出容器**

```shell
exit  # 直接容器停止并退出
Ctrl + P + Q # 退出不停止
```

**删除容器**

```shell
docker rm 容器id	# 删除制定容器，不能删除正在运行的容器，如果要强制删除就 rm -f
docker rm -f $(docker ps -aq)	  # 递归删除所有容器
docker ps -a -q|xargs docker rm   # 删除所有容器
```

**启动/停止容器操作**

```shell
docker start 容器id   # 启动容器
docker restart 容器id		# 重启容器
docker stop 容器id   	# 停止正在运行容器
docker kill 容器id    # 强制停止当前容器
```

### 常用其他命令

**后台启动容器**

```shell
# 命令 docker run -d 镜像名字
[root@VM-0-4-centos ~]# docker run -d centos
7fc42aaf4d5d2a7c9713639b47a9711ae9c8bc4f79e37a087676e2c5b6bbc5f7
# 问题 docker ps，发现centos 停止了
# 常见的坑 docker 容器使用后台运行，就必须要有个前台进程，docker 发现没有应用，就会自动停止
# nginx 容器启动后，发现自己没有提供服务，就会立刻停止，就是没有程序另了
```

**查看日志**

```shell
docker logs -f -t --tail number(条数) 容器

# 编写一个shell脚本模拟日志
[root@VM-0-4-centos ~]# docker run -d centos /bin/sh -c "while true;do eche dengxian;sleep 1;done"
 
# 显示日志
-tf			# 显示日志
--tail number（显示日志条数）
[root@VM-0-4-centos ~]# docker logs -tf --tail 2  12057bc3ef72
2021-02-10T02:01:53.559123855Z /bin/sh: dengxian
2021-02-10T02:01:54.561216524Z /bin/sh: dengxian
```

**查看容器中进程信息**

```shell
# 命令 docker top 容器id
[root@VM-0-4-centos ~]# docker top 12057bc3ef72
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                18700               18682               0                   09:59              
root                20286               18700               0                   10:03  /usr/bin/sleep 1
```

**查看镜像元数据**

 ```shell
#docker inspect 容器id

# 测试
[root@VM-0-4-centos ~]# docker inspect 12057bc3ef72
[
    {
        "Id": "12057bc3ef72faa754b53de68c279fd4cd1f5248002ee3b25d37f80447ecb164",
        "Created": "2021-02-10T01:59:02.74332898Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while true;do eche dengxian;sleep 1;done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 18700,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-02-10T01:59:03.117114088Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        .......
 ```

**进入当前正在运行的容器**

```shell
# docker exec -it 容器id /bin/bash
[root@VM-0-4-centos ~]# docker exec -it 12057bc3ef72 /bin/bash
[root@12057bc3ef72 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@12057bc3ef72 /]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 01:59 ?        00:00:00 /bin/sh -c while true;do eche dengxian;sleep 1;done
root      1803     0  0 02:14 pts/0    00:00:00 /bin/bash
root      1829     1  0 02:14 ?        00:00:00 /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1
root      1830  1803  0 02:14 pts/0    00:00:00 ps -ef

# 方式二 
docker attach 容器id
[root@VM-0-4-centos ~]# docker attach 12057bc3ef72
正在执行当前的代码...

# docker exec	# 进入容器后开启一个新的终端，可以在里面操作
# docker attch	# 进入容器正在执行的终端，不会启动新的进程！
```

**从容器内拷贝文件到主机中**

```shell
[root@VM-0-4-centos ~]# docker attach 9d01abc47f10
[root@9d01abc47f10 /]# cd home    
[root@9d01abc47f10 home]# touch test.java
# 退出容器
[root@VM-0-4-centos ~]# docker cp 9d01abc47f10:/home/test.java /home
[root@VM-0-4-centos ~]# cd /
[root@VM-0-4-centos /]# cd home
[root@VM-0-4-centos home]# ls 
test.java  www

# 拷贝是一个手动的过程，之后可以使用 -v 数据卷的技术，可以实现自动拷贝
```

## 小结

![Snipaste_2021-02-10_10-38-56](images\Snipaste_2021-02-10_10-38-56.png)

## 练习

> docker 安装nginx

```shell
# 1、建议先 docker search 搜索nginx
# 2、docker pull nginx  安装
# 3、 启动：docker
[root@VM-0-4-centos ~]# docker run -d --name nginx01 -p 3344:80 nginx
faa1cee0f25551006d6cb3661d14d16daef327ba9b4b43ea04e5e1a19a4d5062


```

![端口暴露的概念](images\端口暴露的概念.png)

### 可视化

protainer（玩一玩）

```shell
[root@VM-0-4-centos ~]# docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
Unable to find image 'portainer/portainer:latest' locally
latest: Pulling from portainer/portainer
d1e017099d17: Pull complete 
717377b83d5c: Pull complete 
Digest: sha256:f8c2b0a9ca640edf508a8a0830cf1963a1e0d2fd9936a64104b3f658e120b868
Status: Downloaded newer image for portainer/portainer:latest
83f7c54a6ccaa79acb607d980ee4f4330eb2fbef8019b4418213c99cc83ffb8d
```
