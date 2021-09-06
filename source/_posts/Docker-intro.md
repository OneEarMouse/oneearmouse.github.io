---
title: Docker常用命令介绍
date: 2021-09-06 21:20:53
categories:
tags:
description: 介绍Docker服务启动停止，Docker镜像管理，以及Docker容器管理的常用命令
---

# Docker主程序管理
## 设置Docker自启动
```bash
systemctl enable docker
```

## Docker的启动停止重启
```bash
# 重启Docker服务
service docker restart
# 停止Docker服务
service docker stop
# 启动Docker服务
service docker start
```

# Docker镜像管理
## 搜索
```bash
docker search <image>
```
用于搜索线上镜像仓库中的镜像。会显示出所有名称中带有<image>的镜像

## 拉取镜像
```bash
docker pull <image>
```

## 查看本地镜像
```bash
# 列出本地镜像
docker images
# 列出本地镜像（包括历史记录）
docker images -a
# 删除本地镜像
docker rmi <image ID>
```

# Docker容器管理
## 创建容器
```bash
Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]  
 
  -d, --detach=false         指定容器运行于前台还是后台，默认为false   
  -i, --interactive=false   打开STDIN，用于控制台交互  
  -t, --tty=false            分配tty设备，该可以支持终端登录，默认为false  
  -u, --user=""              指定容器的用户  
  -a, --attach=[]            标准输入输出流和错误信息（必须是以非docker run -d启动的容器）
  -w, --workdir=""           指定容器的工作目录 
  -c, --cpu-shares=0        设置容器CPU权重，在CPU共享场景使用  
  -e, --env=[]               指定环境变量，容器中可以使用该环境变量  
  -m, --memory=""            指定容器的内存上限  
  -P, --publish-all=false    指定容器暴露的端口  
  -p, --publish=[]           指定容器暴露的端口 
  -h, --hostname=""          指定容器的主机名  
  -v, --volume=[]            给容器挂载存储卷，挂载到容器的某个目录  
  --volumes-from=[]          给容器挂载其他容器上的卷，挂载到容器的某个目录
  --cap-add=[]               添加权限，权限清单详见：http://linux.die.net/man/7/capabilities  
  --cap-drop=[]              删除权限，权限清单详见：http://linux.die.net/man/7/capabilities  
  --cidfile=""               运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法  
  --cpuset=""                设置容器可以使用哪些CPU，此参数可以用来容器独占CPU  
  --device=[]                添加主机设备给容器，相当于设备直通  
  --dns=[]                   指定容器的dns服务器  
  --dns-search=[]            指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件  
  --entrypoint=""            覆盖image的入口点  
  --env-file=[]              指定环境变量文件，文件格式为每行一个环境变量  
  --expose=[]                指定容器暴露的端口，即修改镜像的暴露端口  
  --link=[]                  指定容器间的关联，使用其他容器的IP、env等信息  
  --lxc-conf=[]              指定容器的配置文件，只有在指定--exec-driver=lxc时使用  
  --name=""                  指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字  
  --net="bridge"             容器网络设置:
                                bridge 使用docker daemon指定的网桥     
                                host     //容器使用主机的网络  
                                container:NAME_or_ID  >//使用其他容器的网路，共享IP和PORT等网络资源  
                                none 容器使用自己的网络（类似--net=bridge），但是不进行配置 
  --privileged=false         指定容器是否为特权容器，特权容器拥有所有的capabilities  
  --restart="no"             指定容器停止后的重启策略:
                                no：容器退出时不重启  
                                on-failure：容器故障退出（返回值非零）时重启 
                                always：容器退出时总是重启  
  --rm=false                 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)  
  --sig-proxy=true           设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理 
```

```console
Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

Options:
      --add-host list                  Add a custom host-to-IP mapping (host:ip)
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR
      --blkio-weight uint16            Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
      --blkio-weight-device list       Block IO weight (relative device weight) (default [])
      --cap-add list                   Add Linux capabilities
      --cap-drop list                  Drop Linux capabilities
      --cgroup-parent string           Optional parent cgroup for the container
      --cgroupns string                Cgroup namespace to use (host|private)
                                       'host':    Run the container in the Docker host's cgroup namespace
                                       'private': Run the container in its own private cgroup namespace
                                       '':        Use the cgroup namespace as configured by the
                                                  default-cgroupns-mode option on the daemon (default)
      --cidfile string                 Write the container ID to the file
      --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int              Limit CPU real-time period in microseconds
      --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds
  -c, --cpu-shares int                 CPU shares (relative weight)
      --cpus decimal                   Number of CPUs
      --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
  -d, --detach                         Run container in background and print container ID
      --detach-keys string             Override the key sequence for detaching a container
      --device list                    Add a host device to the container
      --device-cgroup-rule list        Add a rule to the cgroup allowed devices list
      --device-read-bps list           Limit read rate (bytes per second) from a device (default [])
      --device-read-iops list          Limit read rate (IO per second) from a device (default [])
      --device-write-bps list          Limit write rate (bytes per second) to a device (default [])
      --device-write-iops list         Limit write rate (IO per second) to a device (default [])
      --disable-content-trust          Skip image verification (default true)
      --dns list                       Set custom DNS servers
      --dns-option list                Set DNS options
      --dns-search list                Set custom DNS search domains
      --domainname string              Container NIS domain name
      --entrypoint string              Overwrite the default ENTRYPOINT of the image
  -e, --env list                       Set environment variables
      --env-file list                  Read in a file of environment variables
      --expose list                    Expose a port or a range of ports
      --gpus gpu-request               GPU devices to add to the container ('all' to pass all GPUs)
      --group-add list                 Add additional groups to join
      --health-cmd string              Command to run to check health
      --health-interval duration       Time between running the check (ms|s|m|h) (default 0s)
      --health-retries int             Consecutive failures needed to report unhealthy
      --health-start-period duration   Start period for the container to initialize before starting health-retries countdown (ms|s|m|h) (default 0s)
      --health-timeout duration        Maximum time to allow one check to run (ms|s|m|h) (default 0s)
      --help                           Print usage
  -h, --hostname string                Container host name
      --init                           Run an init inside the container that forwards signals and reaps processes
  -i, --interactive                    Keep STDIN open even if not attached
      --ip string                      IPv4 address (e.g., 172.30.100.104)
      --ip6 string                     IPv6 address (e.g., 2001:db8::33)
      --ipc string                     IPC mode to use
      --isolation string               Container isolation technology
      --kernel-memory bytes            Kernel memory limit
  -l, --label list                     Set meta data on a container
      --label-file list                Read in a line delimited file of labels
      --link list                      Add link to another container
      --link-local-ip list             Container IPv4/IPv6 link-local addresses
      --log-driver string              Logging driver for the container
      --log-opt list                   Log driver options
      --mac-address string             Container MAC address (e.g., 92:d0:c6:0a:29:33)
  -m, --memory bytes                   Memory limit
      --memory-reservation bytes       Memory soft limit
      --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
      --mount mount                    Attach a filesystem mount to the container
      --name string                    Assign a name to the container
      --network network                Connect a container to a network
      --network-alias list             Add network-scoped alias for the container
      --no-healthcheck                 Disable any container-specified HEALTHCHECK
      --oom-kill-disable               Disable OOM Killer
      --oom-score-adj int              Tune host's OOM preferences (-1000 to 1000)
      --pid string                     PID namespace to use
      --pids-limit int                 Tune container pids limit (set -1 for unlimited)
      --platform string                Set platform if server is multi-platform capable
      --privileged                     Give extended privileges to this container
  -p, --publish list                   Publish a container's port(s) to the host
  -P, --publish-all                    Publish all exposed ports to random ports
      --pull string                    Pull image before running ("always"|"missing"|"never") (default "missing")
      --read-only                      Mount the container's root filesystem as read only
      --restart string                 Restart policy to apply when a container exits (default "no")
      --rm                             Automatically remove the container when it exits
      --runtime string                 Runtime to use for this container
      --security-opt list              Security Options
      --shm-size bytes                 Size of /dev/shm
      --sig-proxy                      Proxy received signals to the process (default true)
      --stop-signal string             Signal to stop a container (default "SIGTERM")
      --stop-timeout int               Timeout (in seconds) to stop a container
      --storage-opt list               Storage driver options for the container
      --sysctl map                     Sysctl options (default map[])
      --tmpfs list                     Mount a tmpfs directory
  -t, --tty                            Allocate a pseudo-TTY
      --ulimit ulimit                  Ulimit options (default [])
  -u, --user string                    Username or UID (format: <name|uid>[:<group|gid>])
      --userns string                  User namespace to use
      --uts string                     UTS namespace to use
  -v, --volume list                    Bind mount a volume
      --volume-driver string           Optional volume driver for the container
      --volumes-from list              Mount volumes from the specified container(s)
  -w, --workdir string                 Working directory inside the container
```

### 实例
运行一个spring项目，并替换其中的启动项
```bash
docker run -v /home/user/application_job.properties:/home/user/local/application.properties -p 8085:8085 -e "APP_OPTS=-Dspring.config.additional-location=/home/user/local/application.properties" -d lbs-automation-job:latest
```

解释：
- -v选项：将宿主机的```/home/user/application.properties```文件映射到容器中的```/home/user/local/application.properties```文件夹中，从而使容器能加载配置文件
- -p选项：将宿主机的8085端口映射到容器的8085端口
- -e选项：给容器配置环境变量。这里我们将```spring.config.additional-location=/home/user/local/application.properties```配置为环境变量，使spring项目加载挂载到容器中的对应配置项
- -d选项：指定后台运行
- 镜像名称：指定容器镜像

## 查看Docker容器
```bash
# 查看正在运行的容器
docker ps
```
```console
CONTAINER ID   IMAGE                                      COMMAND                  CREATED         STATUS         PORTS                                       NAMES
abc37d87cf03   lbs-devops/xp-lbs-automation-case:latest   "sh -c 'java $APP_OP…"   4 minutes ago   Up 4 minutes   0.0.0.0:8082->8082/tcp, :::8082->8082/tcp   compassionate_ptolemy
```

```bash
# 查看所有容器（包括已退出）
docker ps -a
```

```concole
CONTAINER ID   IMAGE                                      COMMAND                  CREATED         STATUS                     PORTS                                       NAMES
abc37d87cf03   lbs-devops/xp-lbs-automation-case:latest   "sh -c 'java $APP_OP…"   5 minutes ago   Up 4 minutes               0.0.0.0:8082->8082/tcp, :::8082->8082/tcp   compassionate_ptolemy
51906f38b230   atxserver2_web                             "bash scripts/wait-f…"   6 weeks ago     Exited (255) 3 weeks ago   0.0.0.0:4000->4000/tcp, :::4000->4000/tcp   atxserver2_web_1
```

## 容器管理
```bash
#启动容器
docker start <ContainerId(或者name)>
#停止容器
docker stop <ContainerId(或者name)>
#重启容器
docker restart <ContainerId(或者name)>
#删除容器
docker rm <ContainerId(或者name)>
#删除所有容器
docker rm $(docker ps -a -q)
```

## 操作Docker容器
```bash
docker exec -i <ContainerId> <command>
```
在容器中执行command命令，更多用法可以使用```exec --help```查看

## 查看容器日志
```bash
# 直接查看
docker logs <ContainerId>
# 查看最后n行
docker logs -f -t --tail <行数> <containerId>
```

