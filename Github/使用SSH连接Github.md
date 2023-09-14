## 1. 生成 SSH key

```
cd ~/.ssh

ssh-keygen
```

在 /Users/you/.ssh/ 下会生成 id_rsa 和 id_rsa.pub 两个文件

## 2. 复制 SSH 公钥到 github

1. github 右上角头像出点击，点击 Settings
2. 点击 SSH and GPG keys, New SSH Key
3. 将上一步骤生成的 id_rsa.pub 的内容复制到这里的 key 输入框中。

## 3. 确认

```
ssh -T git@github.com
```

## 4. 修改已有仓库的 url

```shell
# 1. 查看当前的url
git remote -v

# 2. 修改url
git remote set-url origin git@github.com:LiShuxue/xxx.git
```

## 5. 或者一开始 clone 项目的时候就用 SSH 的方式克隆

## 6. 拉取提交代码时候报错

如果突然某一天拉代码或者 push 代码的时候报下图的错，主要是.ssh/known_hosts 中记录的 github 的信息不是正确的信息，原因可能是 github 远程修改了，比如修改了公钥等。我们删掉这一条就行，或者删掉整个 known_host。

![failed](https://cdn.lishuxue.site/blog/image/Github/failed.png)
