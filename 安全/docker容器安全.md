---
title: docker容器安全 
tags: Docker,容器,安全策略
grammar_cjkRuby: true
---
# Docker容器的安全性

### 1.安全策略-Cgroup

 1.限制Cpu 
 
   > docker run --rm -ti -c 2000 ubuntu bash
   
 2.限制内存   
   
   > docker run --rm -ti -m 200M ubuntu bash


 3.限制快设备io 
``` nix
 	 docker run --rm -ti --name container1 ubuntu bash
 	 dd if=/dev/zero of=testfile0 bs=8k mount=5000 oflag=direct
```
 	修改 Cgroup文件

``` elixir
# 查找容器挂载的文件系统“/dev/mapper”的位置
$ mount|grep ContainerID
# 查看容器挂载的文件系统中的文件(Path为上条命令取得的返回结果)
# 返回结果“->”标记后会有一个新的路径，我们先称其为DeviceFilePath
$ ls -l Path
# 查询容器挂载的设备号
$ ls DeviceFilePath -l
# 返回结果中“root disk”后面会有一串用“,”隔开的数字，假设是128和256
# 限制容器的写速度
$ sudo echo '128:256 10240000' >/sys/fs/cgroup/blkio/system.slice/docker-DockerID.scope/blkio.throttle.write_bps_device
# 10240000是每秒可写入的最多的字节数。
```

### 2.ulimit


在Docker1.6之后，可以设置全局默认的ulimit，如设置CPU时间

> sudo docker daemon --default-ulimit cpu=1200
或者在启动容器时，单独对其ulimit进行设置

>docker run --rm -ti --ulimit cpu=1200 ubuntu bash
进入容器后可以查看

>ulimit -t
返回结果：1200

### 3. 容器组网

在接入容器隔离不足的情况下，将受信任的和不受信任的容器组网在不同的网络中，可以降低风险。

### 4. 容器 + 全虚拟化

如果将容器运行在全虚拟化环境中(如在虚拟机中运行容器)，这样就算容器被攻破，虚拟机还具有保护作用。目前一些安全需求很高的应用场景采用的就是这种方式，如公有云场景。

### 5. 镜像签名

Docker可信镜像及升级框架(The Update Framework，TUF)是Docker 1.8所提供的一个新功能，可以校验镜像的发布者。当发布者将镜像push到远程仓库时，Docker会对镜像用私钥进行签名，之后其他人pull该镜像的时候，Docker就会用发布者的公钥来校验该镜像是否和发布者所发布的镜像一致，是否被篡改过，是否是最新版。

### 6. 日志审核

> docker run --rm -ti --log-driver="syslog" ubuntu bash
> 通过docker inspect ContainerID可以看到容器使用了哪种日志驱动。另外，只有json-file支持docker logs命令，docker logs ContainerID。

### 7. 监控

>容器的资源使用情况主要指容器对内存、网络I/O、CPU、磁盘I/O的使用情况等，命令：docker stats ContainerID。
>查看容器的运行状态，命令：docker ps -a。

### 8. 文件系统级防护

``` bash
可读写挂载：
$ docker run --rm -ti ubuntu bash
# echo "hello" >/home/test.txt
# cat /home/test.txt

只读挂载：
$ docker run --rm -ti --read-only ubuntu bash
# echo "hello" >/home/test.txt
# 返回结果：bash:/home/test.txt:Read-only file system
```

### 9. capability

Capabilities 链接 https://wiki.archlinux.org/index.php/Capabilities_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

从2.2版开始，Linux有了capability的概念，它打破了Linux操作系统中超级用户/普通用户的概念，让普通用户也可以做只有超级用户才能完成的工作。capability可以作用在进程上，也可以作用在程序文件上。它与sudo不同，sudo可以配置某个用户可以执行某个命令或更改某个文件，而capability则是让程序拥有某种能力。

每个进程有三个和能力有关的位图：Inheritable(I)\Permitted(P)\Effective(E)，我们可以通过/proc/<PID>/status来查看进程的capability。

```bash
命令：cat /proc/$$/status | grep Cap
结果：
# 能够被当前进程执行的程序继承的capability。
CapInh: 0000000000000000
# 进程能够使用的能力，可以包含CapEff中没有的能力，这些能力是被进程自己临时放弃的，因此可以把CapEff看作是CapPrm的一个子集。
CapPrm: ffffffffffffffff
# 当一个进程要进行某个特权操作时，操作系统会检查CapEff的对应位是否有效，而不再是检查进程的有效UID是否为0。
CapEff: ffffffffffffffff

```
Docker启动容器的时候，会通过白名单的方式来设置传递给容器的capability，默认情况下，这个白名单只包含CAP_CHOWN等少数的能力。用户可以通过 -–cap-add 和 -–cap-drop 这两个参数来修改这个白名单。
```bash
$ docker run --rm -ti --cap-drop=chown ubuntu bash
# chown 2.2/etc/hosts
# 返回结果：chown:changing ownership of '/etc/hosts': Operation not permitted
发现禁掉CAP_CHOWN能力后，在容器里就无法改变容器的所有者了。如果不禁掉则正常。如下

$ docker run --rm -ti ubuntu bash
# chown 2.2/etc/hosts
容器应遵循最小权限原则，尽量不要用–privileged参数，不需要的能力全部去掉，甚至禁掉所有的能力。

$ docker run --rm -ti --cap-drop=all ubuntu bash
```
### 10. SElinux 

SElinux 介绍链接 http://www.cnblogs.com/xiaoluo501395377/archive/2013/05/26/3100444.html

SELinux定义了系统中每个用户、进程、应用和文件访问及转变的权限，然后使用一个安全策略来控制这些实体(即用户、进程、应用和文件)之间的交互，安全策略指定了如何严格或宽松的进行检查。另外，SELinux比较复杂。

### 11. AppArmor

文档配置链接 http://manpages.ubuntu.com/manpages/xenial/en/man5/apparmor.d.5.html

AppArmor也是一种MAC控制机制，其主要作用是设置摸个可执行程序的访问控制权限，可以限制程序读/写某个目录/文件，打开/读/写网络端口等。AppArmor是一个高效和易于使用的Linux系统安全特性，它对操作系统和应用程序进行了从内到外的保护，即使是0day漏洞和未知的应用程序漏洞所导致的攻击也可被识破。AppArmor安全策略可以完全定义个别应用程序所能访问的系统资源与各自的特权，它包含了大量的默认策略，并将先进的静态分析和基于学习的工具结合了起来，可以在很短的时间内，为非常复杂的应用制定AppArmor规则。

### 12. Seccomp

Seccomp(secure computing mode)是一种Linux内核提供的安全特性，它可以实现应用程序的沙盒机制，以白名单或黑名单的方式限制进程进行系统调用。

Seccomp首次于内核2.6.12版合入Linux主线。早期的Seccomp只支持过滤少数几个系统调用。较新版本的内核支持动态Seccomp策略，也就是seccomp-bpf，因为支持用BPF生成过滤规则，从而使Seccomp可以限制任意的系统调用，并且可以限制系统调用传入的参数。

Seccomp的使用

生成BPF形式的过滤规则；
调用prctl系统调用将规则传入内核。
在Docker容器启动的过程中，会对Seccomp设置一个默认的配置，但目前还不支持命令行参数做配置。

### 13. grsecurity

grsecurity提供了一个系统的内核patch，使Linux内核的安全性大大增强，并且它提供了一些工具让用户配置、使用这些安全特性。grsecurity可以用来控制资源访问权限。下面是一张关于grsecurity、SELinux和AppArmor的对比图。
![对比图][1]


  [1]: ./images/1477379689943.jpg "1477379689943.jpg"
  
# 相关安全项目

与Docker安全相关的项目

### 1. Notary

Docker对安全模块进行了重构，剥离出了名为Notary的独立项目。Notary的目标是保证server和client之间的交互使用可信任的连接，用于解决互联网的内容发布的安全性。该项目并未局限于容器应用，在容器场景下可以对镜像源认证、镜像完整性等安全需求提供更好的支持。

### 2. docker-bench-security

docker-bench-security提供一个脚本，它可以检测用户的生产环境是否符合Docker的安全实践。

# Docker安全遗留问题

在Docker的安全问题上Docker社区做了很多的工作，但Docker依然有不少跟安全相关的问题尚未解决。

### 1. User Namespace

User Namespace可以将host中的一个普通用户映射成容器里的root用户，不过虽然允许进程在容器里执行特权操作，但这些特权只局限于该容器内。这对容器的安全是一个非常大的提升，恶意程序通过容器入侵host或者其他容器的风险大大降低，但这并不意味着容器就足够安全了。另外，由于内核层面隔离性不足，如果用户在容器的一个特权操作会影响到容器外，那么这个特权操作一般也是不被User Namespace所允许的。

### 2. 非root运行Docker daemon

目前Docker daemon需要由root用户启动，而Docker daemon创建的容器以及容器里运行的应用实际上也是以root用户运行的。实现由普通用户启动Docker daemon和运行容器，有益于Docker的安全。但这个问题很难解决，因为创建容器需要执行很多特权，包括挂载文件系统、配置网络等。目前社区还没有一个好的方案。

### 3. Docker热升级

Docker管理容器的方式是中心式管理，容器由主机上的Docker daemon进程统一管理。中心式管理方式对于第三方的任务编排工具并不友好，因为什么功能都需要跟Docker关联起来。更大的问题是，如果Docker daemon挂掉了，重启daemon后，它无法接管容器，容器也不能运行了。在实际应用中，很多业务都是不能中断的，而停止容器就往往相当于停止业务，但如果因为安全漏洞的原因需要升级Docker，用于就将处于两难的境地。点击了解该问题的进展。

### 3. 磁盘限额

默认情况下，Docker镜像、容器rootfs、数据卷都存放在/var/lib/docker目录里，也就是说跟host是共享同一个文件系统的。如果不对Docker容器做磁盘大小的配额限制，容器就可能用完磁盘的可用空间，导致host和其他容器无法正常工作。
但是目前Docker几乎没有提供任何接口用于限制容器的磁盘大小。但graphdriver为devicemapper时，容器会被默认分配一个100GB的空间。这个空间大小可以在启动Docker daemon时设置为另一个默认值，但无法对每个容器单独设置一个不同的值。

> $ sudo docker daemon --storage-opt dm.basesize=5G

除此之外，用户只能通过其他手段自行做一些隔离措施，例如为/var/lib/docker单独分配一个磁盘或分区。

### 4. 网络I/O

目前同一台机器上的Docker容器会共享宽带，但这可能出现某个容器占用大部分带宽资源，从而影响其他需要网络资源的容器正常工作的情况。Docker需要一个好的网络方案，除了要解决容器跨主机通信的问题，还要解决网络I/O限制的问题。


