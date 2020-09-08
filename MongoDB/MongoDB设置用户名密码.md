## 场景再现
服务器上部署了MongoDB，上面存储了自己网站的东西。  

突然有一天，网站down了，检查数据库发现数据库中的内容全部没了，只有一个RECOVERY表，里面留了黑客的邮箱。  

我知道数据库肯定是被黑了，被勒索比特币换取数据库恢复。

还好我之前就留了一手，每周一自动备份数据库到本地。所以我及时恢复了数据库，才没有导致太大损失。

> 数据库备份  
> ```sh
> mongodump -h localhost:27017 -d db001 -o /root/db-backup
> ```  
> 数据库恢复  
> ```sh
> mongorestore -h localhost:27017 -d db001 /root/db-backup/db001
> ```   
> 按日期压缩数据库备份文件   
> ```sh
> zip -r db001-`date +%Y-%m-%d-%H-%M-%S`.zip db001
> ``` 

## 问题原因
经过搜索，发现MongoDB默认是不需要认证就可以访问的，再加上我为了可以远程访问服务器的数据库，设置了`bind_ip=0.0.0.0`，也就是可以被网上的任何ip访问到。

## 问题解决
1. 数据库必须开启验证，只允许管理员和用户登陆
2. 还原`bind_ip=127.0.0.1`，只监听服务器本地的访问。(当然有时候我们需要本地远程访问，所以这个我们需要开启，那就必须设置数据库用户名密码了)

## 设置管理员账户与普通账户
### 创建超级管理员账户
```js
use admin

db.createUser({
  user: 'zhangsan',
  pwd: 'password',
  roles: [{
    role: 'root',
    db: 'admin'
  }]
})
```
### 创建只能访问某个数据库的用户，具有读写权限
```js
use db001

db.createUser({
  user: 'lisi',
  pwd: 'password',
  roles: [{
    role: 'readWrite',
    db: 'db001'
  }]
})
```

### 配置文件启用身份验证
```sh
dbpath=/data/db/
logpath=/data/db/mongo.log
logappend=true
fork=true
port=27017

#默认为127.0.0.1，默认只有本机可以连接。 
#将bind_ip配置为0.0.0.0，表示接受任何IP的连接，可以解决远程访问不到的问题
bind_ip=127.0.0.1

# 开启或者关闭登陆认证
auth=true
#noauth=true
```
### 关闭数据库服务，并用这个配置文件重启数据库
```sh
mongod -f /data/db/mongodb.cnf
```

## 使用用户名密码登陆数据库
```js
// 使用管理员账号
use admin
db.auth('zhangsan', 'password')
// 之后会有所有的权限，如
show dbs

// 使用特定的数据库的账号
use db001
db.auth('lisi', 'password')
// 现在只有db001的权限，如
show tables
```

## mongoose连接需要认证的数据库
```js
mongoose.connect('mongodb://username:password@localhost:27017/db001');
```

