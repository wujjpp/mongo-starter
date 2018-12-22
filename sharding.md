# Mongo安装&配置

## Sharding

### 准备目录

```shell
mkdir -p /data/sharding/configReplSet/21001/data
mkdir -p /data/sharding/configReplSet/21002/data
mkdir -p /data/sharding/configReplSet/21003/data
mkdir -p /data/sharding/configReplSet/21004/data

mkdir -p /data/sharding/shardReplSet1/22001/data
mkdir -p /data/sharding/shardReplSet1/22002/data
mkdir -p /data/sharding/shardReplSet1/22003/data
mkdir -p /data/sharding/shardReplSet1/22004/data

mkdir -p /data/sharding/shardReplSet2/23001/data
mkdir -p /data/sharding/shardReplSet2/23002/data
mkdir -p /data/sharding/shardReplSet2/23003/data
mkdir -p /data/sharding/shardReplSet2/23004/data

mkdir -p /data/sharding/mongos/20001
mkdir -p /data/sharding/mongos/20002
mkdir -p /data/sharding/mongos/20003

mkdir -p /data/sharding/conf
mkdir -p /data/sharding/keyfile

```

### 创建key文件

```shell

openssl rand -base64 666 > /data/sharding/keyfile/mongo.key
chmod 600 /data/sharding/keyfile/mongo.key

```

### 创建配置文件

#### mongod.conf

```shell
vim /data/sharding/conf/mongod.conf
```

添加如下内容

```yml
storage:
  engine: wiredTiger
  directoryPerDB: true
  journal:
    enabled: true

systemLog:
  destination: file
  logAppend: true

operationProfiling:
  slowOpThresholdMs: 10000

replication:
  oplogSizeMB: 10240

processManagement:
  fork: true

net:
  bindIpAll: true

security:
  authorization: enabled
```

#### mongos.conf

```shell
vim /data/sharding/conf/mongos.conf
```

添加如下内容

```yml
systemLog:
  destination: file
  logAppend: true

processManagement:
  fork: true

net:
  bindIpAll: true
```

详细配置可参考 [Configuration File Options](https://docs.mongodb.com/manual/reference/configuration-options/index.html)



### 创建config副本集

#### 启动config副本集mongod

```shell
WORK_DIR=/data/sharding
KEYFILE=$WORK_DIR/keyfile/mongo.key
CONFFILE=$WORK_DIR/conf/mongod.conf
MONGOD=/usr/local/mongo/bin/mongod

$MONGOD --port 21001 --configsvr --replSet configReplSet --keyFile $KEYFILE --dbpath $WORK_DIR/configReplSet/21001/data --pidfilepath $WORK_DIR/configReplSet/21001/db.pid --logpath $WORK_DIR/configReplSet/21001/db.log --config $CONFFILE

$MONGOD --port 21002 --configsvr --replSet configReplSet --keyFile $KEYFILE --dbpath $WORK_DIR/configReplSet/21002/data --pidfilepath $WORK_DIR/configReplSet/21002/db.pid --logpath $WORK_DIR/configReplSet/21002/db.log --config $CONFFILE

$MONGOD --port 21003 --configsvr --replSet configReplSet --keyFile $KEYFILE --dbpath $WORK_DIR/configReplSet/21003/data --pidfilepath $WORK_DIR/configReplSet/21003/db.pid --logpath $WORK_DIR/configReplSet/21003/db.log --config $CONFFILE

$MONGOD --port 21004 --configsvr --replSet configReplSet --keyFile $KEYFILE --dbpath $WORK_DIR/configReplSet/21004/data --pidfilepath $WORK_DIR/configReplSet/21004/db.pid --logpath $WORK_DIR/configReplSet/21004/db.log --config $CONFFILE
```

#### 配置config副本集

```shell

mongo --port 21001

# 配置rs
rsconf = {
  _id: 'configReplSet',
  members: [
    {
      _id: 0,
      host: '192.168.31.20:21001'
    }
  ]
}

# 启用配置
rs.initiate(rsconf)

# 创建admin账号
use admin

db.createUser(
  {
    user: "admin",
    pwd: "admin",
    roles: [
      { role: "__system", db: "admin" }
    ]
  }
)

# auth
db.auth('admin', 'admin')

# 添加 secondary
rs.add('192.168.31.20:21002')
rs.add('192.168.31.20:21003')

# 添加Arbiter
rs.addArb('192.168.31.20:21004')
```

### 创建shard1副本集

#### 启动shard1副本集mongod

```shell
WORK_DIR=/data/sharding
KEYFILE=$WORK_DIR/keyfile/mongo.key
CONFFILE=$WORK_DIR/conf/mongod.conf
MONGOD=/usr/local/mongo/bin/mongod

$MONGOD --port 22001 --shardsvr  --replSet shardReplSet1 --keyFile $KEYFILE --dbpath $WORK_DIR/shardReplSet1/22001/data --pidfilepath $WORK_DIR/shardReplSet1/22001/db.pid --logpath $WORK_DIR/shardReplSet1/22001/db.log --config $CONFFILE

$MONGOD --port 22002 --shardsvr  --replSet shardReplSet1 --keyFile $KEYFILE --dbpath $WORK_DIR/shardReplSet1/22002/data --pidfilepath $WORK_DIR/shardReplSet1/22002/db.pid --logpath $WORK_DIR/shardReplSet1/22002/db.log --config $CONFFILE

$MONGOD --port 22003 --shardsvr  --replSet shardReplSet1 --keyFile $KEYFILE --dbpath $WORK_DIR/shardReplSet1/22003/data --pidfilepath $WORK_DIR/shardReplSet1/22003/db.pid --logpath $WORK_DIR/shardReplSet1/22003/db.log --config $CONFFILE

$MONGOD --port 22004 --shardsvr  --replSet shardReplSet1 --keyFile $KEYFILE --dbpath $WORK_DIR/shardReplSet1/22004/data --pidfilepath $WORK_DIR/shardReplSet1/22004/db.pid --logpath $WORK_DIR/shardReplSet1/22004/db.log --config $CONFFILE

```

#### 配置shard1副本集

```shell

mongo --port 22001

# 配置rs
rsconf = {
  _id: 'shardReplSet1',
  members: [
    {
      _id: 0,
      host: '192.168.31.20:22001'
    }
  ]
}

# 启用配置
rs.initiate(rsconf)

# 创建admin账号
use admin

db.createUser(
  {
    user: "admin",
    pwd: "admin",
    roles: [
      { role: "__system", db: "admin" }
    ]
  }
)

# auth
db.auth('admin', 'admin')

# 添加 secondary
rs.add('192.168.31.20:22002')
rs.add('192.168.31.20:22003')

# 添加Arbiter
rs.addArb('192.168.31.20:22004')
```

### 创建shard2副本集

#### 启动shard2副本集mongod

```shell
WORK_DIR=/data/sharding
KEYFILE=$WORK_DIR/keyfile/mongo.key
CONFFILE=$WORK_DIR/conf/mongod.conf
MONGOD=/usr/local/mongo/bin/mongod

$MONGOD --port 23001 --shardsvr  --replSet shardReplSet2 --keyFile $KEYFILE --dbpath $WORK_DIR/shardReplSet2/23001/data --pidfilepath $WORK_DIR/shardReplSet2/23001/db.pid --logpath $WORK_DIR/shardReplSet2/23001/db.log --config $CONFFILE

$MONGOD --port 23002 --shardsvr  --replSet shardReplSet2 --keyFile $KEYFILE --dbpath $WORK_DIR/shardReplSet2/23002/data --pidfilepath $WORK_DIR/shardReplSet2/23002/db.pid --logpath $WORK_DIR/shardReplSet2/23002/db.log --config $CONFFILE

$MONGOD --port 23003 --shardsvr  --replSet shardReplSet2 --keyFile $KEYFILE --dbpath $WORK_DIR/shardReplSet2/23003/data --pidfilepath $WORK_DIR/shardReplSet2/23003/db.pid --logpath $WORK_DIR/shardReplSet2/23003/db.log --config $CONFFILE

$MONGOD --port 23004 --shardsvr  --replSet shardReplSet2 --keyFile $KEYFILE --dbpath $WORK_DIR/shardReplSet2/23004/data --pidfilepath $WORK_DIR/shardReplSet2/23004/db.pid --logpath $WORK_DIR/shardReplSet2/23004/db.log --config $CONFFILE

```

#### 配置shard2副本集

```shell

mongo --port 23001

# 配置rs
rsconf = {
  _id: 'shardReplSet2',
  members: [
    {
      _id: 0,
      host: '192.168.31.20:23001'
    }
  ]
}

# 启用配置
rs.initiate(rsconf)

# 创建admin账号
use admin

db.createUser(
  {
    user: "admin",
    pwd: "admin",
    roles: [
      { role: "__system", db: "admin" }
    ]
  }
)

# auth
db.auth('admin', 'admin')

# 添加 secondary
rs.add('192.168.31.20:23002')
rs.add('192.168.31.20:23003')

# 添加Arbiter
rs.addArb('192.168.31.20:23004')
```

### 配置mongos路由

#### 启动mongos进程

```shell
WORK_DIR=/data/sharding
KEYFILE=$WORK_DIR/keyfile/mongo.key
CONFFILE=$WORK_DIR/conf/mongos.conf
MONGOS=/usr/local/mongo/bin/mongos

$MONGOS --port 20001 --configdb configReplSet/192.168.31.20:21001,192.168.31.20:21002,192.168.31.20:21003 --keyFile $KEYFILE --pidfilepath $WORK_DIR/mongos/20001/db.pid --logpath $WORK_DIR/mongos/20001/db.log --config $CONFFILE

$MONGOS --port 20002 --configdb configReplSet/192.168.31.20:21001,192.168.31.20:21002,192.168.31.20:21003 --keyFile $KEYFILE --pidfilepath $WORK_DIR/mongos/20002/db.pid --logpath $WORK_DIR/mongos/20002/db.log --config $CONFFILE

$MONGOS --port 20003 --configdb configReplSet/192.168.31.20:21001,192.168.31.20:21002,192.168.31.20:21003 --keyFile $KEYFILE --pidfilepath $WORK_DIR/mongos/20003/db.pid --logpath $WORK_DIR/mongos/20003/db.log --config $CONFFILE

```

#### 添加分片

```shell
mongo --port 20001

use admin
db.auth('admin', 'admin')

sh.addShard("shardReplSet1/192.168.31.20:22001")

sh.addShard("shardReplSet2/192.168.31.20:23001")

```

### 启用分片

```shell
use admin
db.auth('admin', 'admin')
use mydb
sh.enableSharding("mydb")

db.createCollection('books')
db.books.ensureIndex({createTime:1})
sh.shardCollection('mydb.books', { bookId: 'hashed' }, false, { numInitialChunks: 4} )

```

### 创建数据库用户

```shell
use admin
db.auth('admin', 'admin')

use mydb
db.createUser(
  {
    user: 'user',
    pwd: 'user',
    roles:[
      { role:'readWrite', db:'mydb' }
    ]
  }
)
```

### 查看数据分布

```shell
db.books.getShardDistribution()
```