## 问题记录

用 Github Actions 去 server 部署项目，使用私钥登录服务器的时候，一直连接失败，本地尝试用私钥连接，也是提示还要输入密码。

主要原因是没有把服务器的公钥放到 authorized_keys，导致私钥验证不了。

## 生成密钥

在服务器端，客户端，分别执行下面命令，分别生成一对密钥对。

```
ssh-keygen
```

## 客户端电脑使用公钥登录服务器

1. 进入客户端电脑.ssh 下面
2. 复制公钥 id_rsa.pub 内容
3. 登录服务器，进入.ssh 下面
4. 创建 authorized_keys，并将复制的客户端的公钥内容，粘贴进去
5. 回到客户端电脑，ssh 登录服务器即可免密登录

## 客户端电脑使用私钥登录服务器

1. 登录服务器，进入.ssh 下面
2. 复制公钥 id_rsa.pub 内容
3. 打开 authorized_keys，并将复制的服务端的公钥内容，粘贴进去（这一步是重点，否则私钥登录不了）
4. 编辑 /etc/ssh/sshd_config 文件，将 RSAAuthentication 和 PubkeyAuthentication 改为 yes
5. 重启 sshd 服务。`systemctl restart sshd.service`
6. 复制私钥 id_rsa 内容
7. 回到客户端，在桌面创建文件 private.key，并将复制的服务端的私钥内容，粘贴进去
8. 使用私钥登录, -i 文件 `ssh root@lishuxue.site -i ~/Desktop/private.key`

公钥登录，需要将每台客户端的公钥全部放到服务器的 authorized_keys 里面。

私钥登录，只需每台客户端保留服务器的私钥即可。但是私钥一般不要泄露。
