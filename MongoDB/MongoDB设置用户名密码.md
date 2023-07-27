## 场景再现

服务器上部署了 MongoDB，上面存储了自己网站的东西。

突然有一天，网站 down 了，检查数据库发现数据库中的内容全部没了，只有一个 RECOVERY 表，里面留了黑客的邮箱。

我知道数据库肯定是被黑了，被勒索比特币换取数据库恢复。

还好我之前就留了一手，每周一自动备份数据库到本地。所以我及时恢复了数据库，才没有导致太大损失。

数据库备份

```sh
mongodump --host localhost --port 27017 --db db001 --authenticationDatabase admin --username username --password password --out /root/db-backup
```

数据库恢复

```sh
mongorestore --host localhost --port 27017 --db db001 --authenticationDatabase admin --username username --password password  /root/db-backup/db001
# mongorestore --host localhost --port 27017 --db journey --authenticationDatabase admin --username lishuxue --password lishuxue  /backup/journey
```

按日期压缩数据库备份文件

```sh
zip -r db001-`date +%Y-%m-%d-%H-%M-%S`.zip db001
```

## 问题原因

经过搜索，发现 MongoDB 默认是不需要认证就可以访问的，再加上我为了可以远程访问服务器的数据库，设置了`bind_ip=0.0.0.0`，也就是可以被网上的任何 ip 访问到。

## 问题解决

1. 数据库必须开启验证，只允许管理员和用户登陆
2. 还原`bindIp=127.0.0.1`，只监听服务器本地的访问。(当然有时候我们需要本地远程访问，所以这个我们需要开启，那就必须设置数据库用户名密码了)

## 设置管理员账户与普通账户

### 创建超级管理员账户

```js
use admin

db.createUser({
  user: 'lishuxue',
  pwd: 'lishuxue',
  roles: [{
    role: 'root',
    db: 'admin'
  }]
})
```

### 创建只能访问某个数据库的用户，具有读写权限

```js
use journey

db.createUser({
  user: 'journey',
  pwd: 'journey',
  roles: [{
    role: 'readWrite',
    db: 'journey'
  }]
})
```

### 配置文件启用身份验证

```sh
# 设置MongoDB服务器监听的IP地址和端口，将bindIp配置为0.0.0.0，表示接受任何IP的连接，可以解决宿主机访问不到或者远程访问不到的问题
net:
  bindIp: 127.0.0.1
  port: 27017

# 设置数据存储目录
storage:
  dbPath: /data/db
  journal:
      enabled: true

# 设置日志文件
systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true

# 设置身份验证
security:
  authorization: enabled
```

### 用这个配置文件重启数据库

```sh
# docker 启动
docker run -d --rm --name journey-mongodb -p 27017:27017 -v /Users/lishuxue/Documents/software/mongodb/data/db:/data/db -v /Users/lishuxue/Documents/software/mongodb/data/configdb:/data/configdb -e MONGO_INITDB_ROOT_USERNAME=lishuxue -e MONGO_INITDB_ROOT_PASSWORD=lishuxue mongo --config /data/configdb/mongod.conf
```

## 使用用户名密码登陆数据库

```js
// 使用管理员账号
use admin
db.auth('lishuxue', 'lishuxue')
// 之后会有所有的权限，如
show dbs

// 使用特定的数据库的账号
use journey
db.auth('journey', 'journey')
// 现在只有journey的权限，如
show tables
```

## mongoose 连接需要认证的数据库

```js
mongoose.connect('mongodb://username:password@localhost:27017/db001');
```

## mongod，mongo，mongosh

我们安装好 domgodb 数据库后，需要 mongod 命令来启动数据库服务。启动之后，使用 mongo，打开 shell 命令行工具，可以查看或者删除数据库等。mongosh 是新版的 mongo，提供了更友好的操作。

可以使用下面的命令直接连接数据库。

```sh
mongosh --host localhost -u lishuxue -p lishuxue --authenticationDatabase admin
```

## mongosh 中的增删改查

查看：`db.collection.find({ filter })`，filter 是查询条件，可以是空对象 {} 表示查询所有数据，也可以是具体的查询条件。collection 代表表名。

使用 insertOne() 或 insertMany() 方法来插入数据，`db.collection.insertOne({ data })`

使用 updateOne() 或 updateMany() 方法来更新数据，`db.collection.updateOne({ filter }, { $set: { updateData } })`

使用 deleteOne() 或 deleteMany() 方法来删除数据，`db.collection.deleteOne({ filter })`

查看所有列名：

在 MongoDB 中，集合（表）是无模式（schemaless）的，意味着每个文档（记录）可以具有不同的字段。因此，MongoDB 没有严格的表结构，也没有列的概念。但是可以使用 find() 方法来查询文档，并查看文档的字段。

```
db.User.find().forEach((user) => printjson(Object.keys(user)) );
```
