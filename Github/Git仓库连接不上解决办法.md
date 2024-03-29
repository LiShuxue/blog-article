最近 github 使用过程中总是莫名其妙的出问题，时而有问题，时而没问题，一开始以为是网络的问题，没有管他，但是频繁出现问题，所以在此记录问题及解决方案。

## 1. Git 仓库过大或者 pull,push 的代码量太多

- fatal: The remote end hung up unexpectedly
- error: RPC failed; result=56, HTTP code = 0

上述问题解决方案：增加缓冲区大小

```
git config --global http.postBuffer 50M
```

## 2. 网络问题

- Failed connect to github.com:443; Connection timed out。
- fatal: unable to access '<https://github.com/xxx>': Encountered end of file
- fatal: unable to access '<https://github.com/xxx>': Empty reply from server

这种问题一般是 https 的问题，或者是 github 服务器在国外，中间访问时候网络波动导致连接不上。

网上试了几种方案，都没有解决

```
git config --global http.sslVerify false
git config --global --unset https.proxy
git config --global --unset http.proxy
```

最终下面这种方案解决了问题

```
在项目下打开.git/config文件，将里面的项目的URL，https://改为git:// 。即使用git协议
或者，使用SSH协议，url为 git@github.com:LiShuxue/xxx.git 也应该可以解决（未尝试）
```

## 3. 22 端口问题

今天突然用 ssh 方式 pull GitHub 的项目报错：ssh: connect to host xx.xx.xx.xx port 22: Connection timed out

表明 SSH 连接在尝试通过 22 端口连接到远程服务器时超时。这可能是由于网络环境、防火墙设置或代理配置等原因导致的(很可能端口 22 被防火墙或提供商阻止了)。

为了解决此问题，我们可以尝试将 SSH 连接切换到 443 端口。

~/.ssh/config 文件中添加以下内容（如果没有文件就创建该文件）：

```
Host github.com
  Hostname ssh.github.com
  Port 443
```
