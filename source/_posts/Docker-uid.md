---
title: Docker容器中的uid和gid
date: 2024-02-14 20:25:31
categories:
tags:
description: Docker容器中的uid和gid的概念以及和基础Linux系统的关系
---
# 参考资料来源

[# 理解 docker 容器中的 uid 和 gid](https://www.cnblogs.com/sparkdev/p/9614164.html)

# 原理
## 什么是uid和gid
uid和gid由Linux内核负责管理，通过内核级别的调用来决定是否为某个请求授予特权。

例子：表面上，用户在某个文件夹中写入文件时，Linux系统会检测用户的修改权限。实际上，内核使用的是uid和gid而不是用户名和组名

docker经常被理解为轻量虚拟机，但事实上docker与虚拟机有重要的不同：同一主机上运行的所有容器共享一个内核（宿主机的内核）。即使docker宿主机上运行了成百上千的容器，内核控制的uid和gid只有一套。

显示用户名的Linux工具不属于内核层，可能会看到同一个uid在不同容器中显示为不同的用户名。

## 容器中默认使用的用户（root用户）
如果不进行特殊配置，容器中的进程默认以root用户权限启动。
```bash
docker run -d  --name sleepme ubuntu sleep infinity
```
在主机中使用id命令，可以看到如下内容（ 我的主机用户名是codersh)
```bash
uid=1000(codersh) gid=1000(codersh) groups=1000(codersh),4(adm),20(dialout),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),116(netdev),1001(docker)
```
在主机中使用su切换为root用户，再使用id命令，可以看到:
```
uid=0(root) gid=0(root) groups=0(root)
```
在容器中使用id命令，可以看到
```
uid=0(root) gid=0(root) groups=0(root)
```

容器中的用户uid直接对应到了0，即宿主机中的root权限


验证方式：可以在宿主机中创建一个只有root权限可以修改的文件，然后挂载到容器中，会发现容器中可以直接读写该文件

# 在Docker中指定用户身份
## 方式1
在docker打包镜像时，在Dockerfile中指定用户。例如
```Dockerfile
FROM ubuntu
RUN useradd -r -u 1000 -g appuser
USER appuser
ENTRYPOINT ["sleep", "infinity"]
```

以该镜像启动该容器后，默认使用的用户即变为了uid=1000的用户appuser，即宿主机中uid=1000的用户codersh

dokcer在启动时，使用宿主机内核。因此宿主机中uid=1000的用户codersh，被用于docker容器中uid=1000的用户appuser。两者其实是同一用户，对挂载到docker中的文件有相同的读写权限

## 方式2
从docker run命令中的`--user`参数指定进程用户身份。例如
```bash
docker run -d --uid 1000 --gid 1000 --name sleepme ubuntu sleep infinity
```

需要注意的是，在创建容器时通过```docker run --user```指定的用户身份会覆盖掉 Dockerfile 中指定的值。

与docker run等价的docker-compose也可以用类似的方式指定用户。

# 总结
如果启动Docker容器时，没有指定容器的uid与gid，那么Docker容器就会以root权限启动。这种情况下，Docker容器拥有修改挂载到容器中的任意文件的权限。建议在生产环境中给Docker容器分配uid，并将需要挂载的文件分配给uid对应的用户，做好权限隔离。