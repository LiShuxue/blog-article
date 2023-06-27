## Yum

Yum（全称为 Yellow dog Updater, Modified）是一个在 Fedora 和 RedHat 以及 CentOS 中的 Shell 前端软件包管理器。基于 RPM 包管理，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。

Centos 7 自带 yum。

## 配置镜像源

由于安装 centos 后的默认 yum 源为 centos 的官方地址，所以在国内使用很慢甚至无法访问，所以一般的做法都是把默认的 yum 源替换成 aliyun 的 yum 源或者 163 等国内的 yum 源。

### 1. 备份

`mkdir /etc/yum.repos.d/repo-bck`

`mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/repo-bck`

### 2. 下载新的阿里镜像源

`wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo`

如果没有 wget，先安装 wget (-y 对所有的提问都回答“yes”)

`yum install -y wget`

### 3. 清楚缓存 和 重新生成缓存

`yum clean all`

`yum makecache`

## 选项

- -h：显示帮助信息；
- -y：对所有的提问都回答“yes”；
- -c：指定配置文件；
- -q：安静模式；
- -v：详细模式；
- -d：设置调试等级（0-10）；
- -e：设置错误等级（0-10）；
- -R：设置 yum 处理一个命令的最大等待时间；
- -C：完全从缓存中运行，而不去下载或者更新任何头文件。

## 命令

```sh
yum install package1     # 安装指定的安装包package1

yum update package1      # 更新指定程序包package1 (保留旧版本)

yum upgrade package1     # 升级指定程序包package1（不保留旧版本）

yum info package1        # 显示安装包信息package1

yum list package1        # 显示指定程序包安装情况package1

yum remove package1      # 删除指定程序包package1

yum clean packages       # 清除缓存目录下的软件包
yum clean headers        # 清除缓存目录下的 headers
yum clean all            # 清除缓存目录下的所有

yum search package1      # 检查指定程序包package1信息
```
