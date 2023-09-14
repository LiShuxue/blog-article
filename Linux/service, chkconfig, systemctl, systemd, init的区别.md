## service

在目录/etc/rc.d/init.d 下有许多服务器脚本程序，一般称为服务(service)。

service 命令用于对系统服务进行管理，比如启动（start）、停止（stop）、重启（restart）、查看状态（status）等。

service 命令其实是去/etc/init.d 目录下，去执行相关程序。

```sh
service <service> start
service <service> stop
service <service> restart
service --status-all           # 查看所有的服务
```

## chkconfig

chkconfig 命令用于检查，设置系统的各种服务。

这是 Red Hat 公司遵循 GPL 规则所开发的程序，它可查询操作系统在每一个执行等级中会执行哪些系统服务，其中包括各类常驻服务。

```sh
chkconfig -list         # 列出chkconfig所知道的所有命令。
chkconfig telnet on     # 开启Telnet服务
chkconfig telnet off    # 开启Telnet服务
chkconfig --add xx　    # 增加所指定的系统服务xx，让chkconfig指令得以管理它
chkconfig --del xx　    # 删除所指定的系统服务xx，不再由chkconfig指令管理
```

## init， SystemV

SystemV 架构，以前的 Linux 启动是用 init 来做进程初始化 (Rhel6)，位于 /sbin/init, 是系统中第一个进程, 由内核加载运行。

启动服务时：

```sh
$ sudo /etc/init.d/apache2 start
# 或者
$ service apache2 start
```

缺点：

- 启动时间长。init 进程是串行启动，只有前一个进程启动完，才会启动下一个进程。
- 启动脚本复杂。init 进程只是执行启动脚本，不管其他事情。脚本需要自己处理各种情况，这往往使得脚本变得很长。

## systemd

在较新的 linux 系统上（Rhel7），都使用 systemd 取代了 init，成为系统的第一个进程，来做进程初始化。其他进程都是它的子进程。systemd 为系统启动和管理提供了完整的解决方案。字母 d 是守护进程（daemon）的缩写。

优点：

- 按需启动服务，减少系统资源消耗。
- 尽可能并行启动进程，减少系统启动等待时间

Systemd 可以管理所有系统资源。不同的资源统称为 Unit（单位）。Unit 一共分成 12 种。

- Service unit：系统服务
- Target unit：多个 Unit 构成的一个组
- Device Unit：硬件设备
- Mount Unit：文件系统的挂载点
- Automount Unit：自动挂载点
- Path Unit：文件或路径
- Scope Unit：不是由 Systemd 启动的外部进程
- Slice Unit：进程组
- Snapshot Unit：Systemd 快照，可以切回某个快照
- Socket Unit：进程间通信的 socket
- Swap Unit：swap 文件
- Timer Unit：定时器

## systemctl

systemctl 是 systemd 的主命令，用于管理系统。

用户最常用的是启动和停止 Unit（主要是 service）命令

```sh
# 立即启动一个服务
$ sudo systemctl start apache.service

# 立即停止一个服务
$ sudo systemctl stop apache.service

# 重启一个服务
$ sudo systemctl restart apache.service

# 显示单个 Unit 的状态
$ sudo systemctl status bluetooth.service

# 开机自启动服务（等同于chkconfig httpd on）
$ systemctl enable httpd.service

# 开机时禁用服务（等同于chkconfig httpd on）
$ systemctl disable httpd.service

# 杀死一个服务的所有子进程
$ sudo systemctl kill apache.service

# 重新加载一个服务的配置文件
$ sudo systemctl reload apache.service

# 重载所有修改过的配置文件
$ sudo systemctl daemon-reload

# 显示某个 Unit 的所有底层参数
$ systemctl show httpd.service

# 显示某个 Unit 的指定属性的值
$ systemctl show -p CPUShares httpd.service

# 设置某个 Unit 的指定属性
$ sudo systemctl set-property httpd.service CPUShares=500
```

一些别的命令
<https://blog.csdn.net/weixin_40763897/article/details/106865755>

```sh
# 重启系统
$ sudo systemctl reboot

# 关闭系统，切断电源
$ sudo systemctl poweroff

# CPU停止工作
$ sudo systemctl halt

# 暂停系统
$ sudo systemctl suspend

# 让系统进入冬眠状态
$ sudo systemctl hibernate

# 让系统进入交互式休眠状态
$ sudo systemctl hybrid-sleep

# 启动进入救援状态（单用户状态）
$ sudo systemctl rescue

# 查看启动耗时
$ systemd-analyze

# 查看每个服务的启动耗时
$ systemd-analyze blame

# 显示瀑布状的启动过程流
$ systemd-analyze critical-chain

# 显示指定服务的启动流
$ systemd-analyze critical-chain atd.service

# 显示当前主机的信息
$ hostnamectl

# 设置主机名。
$ sudo hostnamectl set-hostname rhel7

# 列出正在运行的服务 Unit
$ systemctl list-units

# 列出所有Unit，包括没有找到配置文件的或者启动失败的
$ systemctl list-units --all

# 列出所有没有运行的 Unit
$ systemctl list-units --all --state=inactive

# 列出所有加载失败的 Unit
$ systemctl list-units --failed

# 列出所有正在运行的、类型为 service 的 Unit
$ systemctl list-units --type=service

# 显示系统状态
$ systemctl status

# 显示单个 Unit 的状态
$ systemctl status bluetooth.service

# 显示远程主机的某个 Unit 的状态
$ systemctl -H root@rhel7.example.com status httpd.service
```

## 区别

Rhel6 用 service 和 chkconfig 来管理服务，它是 SystemV 架构下的一个工具。

Rhel7 是用 systemctl 来管理服务，它融合了之前的 service 和 chkconfig 的功能于一体。可以使用它永久性或只在当前会话中启用/禁用服务。

systemctl 是 systemd 架构下的一个工具。
