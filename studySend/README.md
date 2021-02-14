# Docker镜像讲解

## 镜像是什么

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软甲你所需的所有内容，包括代码、运行时、库、环境变量和配置文件

所有的应用，直接打包 docker 镜像，就可以直接跑起来。

如何得到镜像：

- 远程仓库下载
- 拷贝
- 自己制作一个镜像 Docker File

## Docker 镜像加载原理

> UnionFS（联合文件系统）

UnionFS（联合文件系统）：Union文件系统(UnionFS) 是一种分层、轻量级并且高性能的文件系统，他支持对文件系统的修改作为一次提交来一层层叠加，同事可以将不同目录挂在到同一虚拟文件系统下（Unite Several Directories Into A Single Virtual fileSystem）。Union文件系统是 Docker 镜像的基础，镜像可以通过分层来进行继承，基于基础镜像（没有父镜像）可以制作具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

> Docker 镜像加载原理

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS.

`Bootfs(Boot File System)`主要包含 `BootLoader` 和 `kernel`, `BootLoader`主要是引导加载 `Kernel`, Linux刚启动时会加载`bootfs`文件系统，在Docker镜像的最底层是boots。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs,
`RootFs (Root File System)`，在bootfs之上。包含的就是典型Linux.系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。roots就是各种不同的操作系统发行版，比如`Ubuntu , Centos`等等。

![docker镜像加载原理](https://github.com/VisDeng/Docker/blob/master/studySend/images/docker%E9%95%9C%E5%83%8F%E5%8A%A0%E8%BD%BD%E5%8E%9F%E7%90%86.png)

平时安装进虚拟机的 CentOS 都是好几G，为什么Docker这里才 200M！

![image-20210212210407135](images\image-20210212210407135.png)

对于一个精简的 OS，rootfs可以很小，只需要包括最基本的命令，工具和程序库就可以了，因为底层直接用Host 的 Karnel，自己只需要提供 bootfs 就可以了，由此可见对于不同的 Linux 发行版，rootfs 基本是一致的，rootfs会有差别，因此不同的发行分版可以共用 BootFS。

**虚拟机是分钟级，容器是秒级。**
