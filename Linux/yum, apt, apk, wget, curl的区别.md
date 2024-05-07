## Linux 分类

RedHat 系列：Redhat、Centos、Fedora 等

Debian 系列：Debian、Ubuntu 等

Alpine Linux 基础镜像非常小，通常用于 Docker 容器。

## yum

RedHat 系列 包管理工具 yum

常见的安装包格式 rpm 包，安装 rpm 包的命令是 `rpm -参数`

支持 tar 包

Fedora 已逐渐转向使用 dnf（Dandified YUM），一种新的包管理器，它保留了 yum 的主要特性但提供了更好的性能和功能。Fedora 22 及之后版本默认使用 dnf。

## apt

Debian 系列 包管理工具 apt

常见的安装包格式 deb 包，安装 deb 包的命令是 `dpkg -参数`

支持 tar 包

## apk

Alpine Linux 的官方包管理器是 apk（Alpine Package Keeper）。它是为了在 Alpine Linux 上安装、更新和删除软件包而设计的，类似于其他发行版的 apt 或 yum/dnf。

在 Alpine Linux 系统中，您可以通过查看 /etc/apk/repositories 文件来获取当前配置的包下载地址：`cat /etc/apk/repositories`，进入以后可以下载各个包：<https://dl-cdn.alpinelinux.org/alpine/v3.19/main/x86_64/>

也可以在 Alpine Linux 包仓库网站来看详细的包信息：<https://pkgs.alpinelinux.org/packages>

## 包地址

<https://pkgs.org/> 这个网站提供了各种 Linux 发行版的软件包信息。

## curl wget

wget 和 curl 都可以下载内容。

curl 由于可自定义各种请求参数所以在模拟 web 请求方面更擅长。wget 由于支持 ftp 和 Recursive 所以在下载文件方面更擅长。

curl 像是一个精简的命令行网页浏览器，而 wget 是迅雷。
