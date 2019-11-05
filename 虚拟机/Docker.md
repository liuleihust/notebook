# Docker

Docker 利用了 linux 中的 `namespaces` 和 `cgroups` 为一个容器创造了一个虚拟的环境。

**namespaces** ： 封装操作系统的资源，形成多个实例。

**cgroups**:  control groups ,提供了一个机制去审计或限制进程能够访问到的资源。



## Docker internal Security

OS-level 虚拟化应满足：`process isolation`, `filesystem isolation`,`device isolation`,`IPC isolation`, `network isolation` and `limiting of resources`.



### Process Isolation

进程隔离主要为了阻止 被侵害的容器使用进程管理接口干扰别的容器。Docker 通过封装 容器中运行的进程 ，利用 namespace ,并且限制他们的权限和对于其他容器和 下面的host 的进程的可见性。

通过提供 `PID namespaces` ，每个容器拥有自己的 `PID  namespace ` 。

因为 `PID  namespaces` 是分级的，一个进程只能看到自己的namespace 的 进程或它孩子namespace。 一个新的 namespace 被创建并且分配给一个容器，这个host主机可以观察和影响到 这个 新的namespace的进程，但是这个新的进程不能够访问到 HOST 主机里的进程或其他容器里的进程。

这个PID namespace 同样允许容器有自己的 init-like 进程(PID 1) ,当这个 进程终止的话，其他进程也都停止，这可以帮助 管理员完全关闭容器当检测到受到侵害时。



### Filesystem Isolation

为了实现 **文件系统隔离**，主机或者容器的文件系统必须能够阻止非法的访问和修改。

 Doceker 使用 `mount namespaces` ，也可以称做文件系统 命名空间，去隔离 不同容器的文件系统层级。 

**挂载名称空间**为每个容器的进程提供文件系统树的不同视图，并将容器内发生的所有挂载事件限制为只在容器内发生影响。

然而，一些内核的文件系统不是 `namespaced` ，例如，那些  **under** */sys, /proc/sys, /proc/sysrq − trigger, /proc/irq, and /proc/bus*  ， 一个 docker 容器为了运行需要去挂载它们。这就造成一个问题，一个容器需要从 `host`  主机 继承他的文件系统的视图 而且可以直接访问到它们。

Docker使用两种文件系统保护机制，通过这些文件系统限制受危害容器可能对主机造成的威胁：

1. 将对这些文件系统的写权限从容器中移除。
2.  不允许容器的任何进程重新挂载容器内的任何文件系统。

第二种机制通过 从容器中删除 `CAP_SYS_ADMIN`  功能来实现。

Docker 同样采用 被称为 写时复制的文件系统。正如之前提到的，Docker 基于文件系统镜像创建容器。一个容器可以在它自己的 镜像下上写文件。当多个容器由同一个镜像创建的时候，`copy-on-write`  文件系统允许每个容器向特定的文件系统上写内容。 因此阻止别的容器发现容器内部发生的改变。



### Device Isolation 

在 Unix，kernel 和 应用 通过 设备节点(**本质上一个特殊的文件，表现为一个设备驱动的接口**) 访问 硬件，如果一个容器可以访问到一些重要的设备节点，如*/dev/mem (the physical memory), /dev/sd∗ (the storage) or /dev/tty (the terminal)*  

它会对主机系统造成严重的损害。因此，限制 容器能够访问的设备节点是重要的。

*cgroups* 的 设备白名单功能  可以限制 Docker 允许容器访问的设备。它也阻止容器中的进程创建新的设备节点。因此，`Docker` 挂载的容器镜像不带设备，意味着即使设备节点在镜像中被创建，使用该镜像的容器中的进程不能使用 它去和内核进行交流。默认的，Docker 不能给它的容器扩展的权限。因此，它们不能访问到任何设备。 然而，如果一个操作以特权方式执行容器，Docker 保证容器能够访问到所有设备  



### IPC Isolation

IPC 是 进程为了交换数据的一组对象。运行在容器中的进程必须受到限制来使得它们能够通过一些确定的 IPC 资源而且不能干扰其他容器的IPC 资源。

Docker 实现IPC 隔离，通过使用IPC namespaces ，这将运行创建 不同的 *IPC namespaces* 。在一个 IPC namespaces 的进程不能够 读或写  其他 *IPC namespaces* 的IPC 资源 。Docker 为每个容器分配一个IPC 命名空间，因此，能够阻止容器中的进程去阻止其他容器的进程 。



### Network Isolation

网络隔离 对于阻止网络攻击是重要的，例如 *Man-in-the-Middle* （中间人攻击）、*ARP 欺骗*

容器必须能够以某种方式进行配置以至于他们不能够 被窃听或操控其他容器的流量。

对于每个容器，`Docker` 创造了一个独立的网络堆 通过使用 *network namespaces* 。

因此，每个容器有它自己的 ip地址，IP 路由表，网络设备等。这允许容器能够和别人交流通过 他们各自的网络接口，它们采用同样的方式进行外部 主机交流。

​	默认，容器和host主机通信通过 虚拟以太网网桥。

![1561363948632](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561363948632.png)

通过这个方法，Docker 创建了一个 虚拟以太网网桥在host 机器上，名字叫做 *docker0*

,它可以在网络接口之间自动转发数据包。当 docker 创建一个新的容器的时候，它自动创建一个新的虚拟网络接口（**拥有唯一的名字**），并且将这个接口与网桥连接起来。这个接口同样连接 *eth0* 接口，因此可以允许容器发送数据包到网桥上。

默认的连接模式容易遭受 ARP 欺骗 和 MAC洪水攻击，因为网桥转发所有到来的数据包没有任何过滤。

### Limiting of Resources 

*DoS* 是多租户系统上一种常见的工具方式。一个进程或 一组进程 尝试消耗掉所有系统资源，因此扰乱其他进程的操作。为了阻止这种攻击，它应该能够去限制 分配给每个容器的资源。

*Cgroups* 是 Docker 解决此类问题的关键。它控制资源的数量，例如 *CPU* ，*memory* 和 *disk I/O*。确保每个容器能够公平的共享这些资源，阻止任何容器去消耗掉所有资源。它们还允许Docker配置与分配给每个容器的资源相关的限制和约束。例如，其中一个约束是将可用的cpu限制到特定容器。



## Docker and Kernel Security Systems



### Linux Capabilities

 

传统上，Unix systems 将程序分为两类：特权进程（被超级用户拥有或 root）和非特权进程。内核跳过特权进程的所有权限检查，但是对非特权进程执行完全权限检查。然而，自从 version 2.2以来，Linux 内核将超级用户的权限划分为 *capabilities* 。

Docker 容器 共享主机的内核。总的来说，对于大多数情况，没有必要给 容器全部root权限，因此从容器 移除一些 root 权限不影响容器的可用性和功能性，但是可以有效的提升系统的安全性。例如，*CAP_NET_ADMIN* ：提供了修改系统网络的功能。可以被移除，因为所有的网络配置工作可用在容器启动前被 docker daemon 完成。

Docker 允许配置这些 capabilities 。默认，Docker 禁用了很多 linux capabilities ，阻止侵入者损害 host 系统。



### SELinux

selinux 是一个linux 安全增强模块。Linux 提供了 标准的 DAC 机制（owner/group ）。SElinux 提供了 额外的许可检查，称作 MAC，在DAC 执行后被执行。在SElinux ,每个东西都被标签控制，每个 文件/目录,进程和 系统客体拥有 标签。系统管理员使用他们的标签去写规则 去控制进程和系统之间的客体的访问。这些规则被称作 policies。SElinux 策略被划分为三类：类型控制，多级安全控制，和类别控制。

在 DAC 机制下，拥有者拥有对属于它的客体全部的判定。这意味着如果该用户被侵害，攻击者可以控制它的所有客体。相反，在SElinux 中，内核管理和执行所有的访问控制，而不是拥有者。这为容器提供了一个安全隔离，即容器中的拥有root 权限的进程不能对容器外的进行非法访问。



 Docker 使用 类型强制访问控制和 类别强制访问控制。类型强制访问控制保护主机不受容器中的进程影响。类别强制访问控制保护 容器不受另一个容器的损害。



在强制访问控制中，Docker 将所有容器进程标记*svirt_lxc_net_t* 类型，而且所有内容标记为`svirt_sandbox_f ile_t` 类型。标记为 svirt_lxc_net_t 的进程只可以访问/写  标记为`svirt_sandbox_f ile_t` 类型的内容。因为这些标记 只存在 容器中，因此 容器中的进程只可以使用容器中的内容。然而，只有类型强制访问控制是不够的，因为一个容器中的进程可以访问到其他容器中的进程。 MCS 可以解决这个问题，当一个容器启动时，Docker 服务程序挑选一个 随机的 MCS 标记，然后放到它们所有的进程和内容中。内核只允许进程访问相同的内容MCS标签，从而防止一个进程中的损坏攻击其他容器。