# Mongo安装&配置

## Replication

### 准备目录

```shell

mkdir -p /data/replication/logs
mkdir -p /data/replication/conf
mkdir -p /data/replication/keyfile

mkdir -p /data/replication/data/27017
mkdir -p /data/replication/data/27018
mkdir -p /data/replication/data/27019
mkdir -p /data/replication/data/arb

```

### 创建key文件

```shell

openssl rand -base64 666 > /data/replication/keyfile/mongo.key
chmod 600 /data/replication/keyfile/mongo.key

```

### 创建配置文件

```shell
vim /data/replication/conf/mongod.conf

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
  replSetName: rs0

processManagement:
  fork: true

net:
  bindIpAll: true

security:
  authorization: enabled

```

详细配置可参考 [Configuration File Options](https://docs.mongodb.com/manual/reference/configuration-options/index.html)

### 启动mongod

```shell
WORK_DIR=/data/replication
KEYFILE=$WORK_DIR/keyfile/mongo.key
CONFFILE=$WORK_DIR/conf/mongod.conf
MONGOD=/usr/local/mongo/bin/mongod

$MONGOD --port 27017 --keyFile $KEYFILE --dbpath $WORK_DIR/data/27017 --logpath $WORK_DIR/logs/27017.log --config $CONFFILE

$MONGOD --port 27018 --keyFile $KEYFILE --dbpath $WORK_DIR/data/27018 --logpath $WORK_DIR/logs/27018.log --config $CONFFILE

$MONGOD --port 27019 --keyFile $KEYFILE --dbpath $WORK_DIR/data/27019 --logpath $WORK_DIR/logs/27019.log --config $CONFFILE

$MONGOD --port 27020 --keyFile $KEYFILE --dbpath $WORK_DIR/data/arb --logpath $WORK_DIR/logs/arb.log --config $CONFFILE

```

### 配置副本集

```shell

mongo --port 27017

# 配置rs
rsconf = {
  _id: 'rs0',
  members: [
    {
      _id: 0,
      host: '192.168.31.20:27017'
    }
  ]
}

# 启用配置
rs.initiate(rsconf)

# 创建超级账号
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
rs.add('192.168.31.20:27018')
rs.add('192.168.31.20:27019')

# 添加Arbiter
rs.addArb('192.168.31.20:27020')

# 查看副本集配置信息
rs.conf()

# 查看master oplog状态
rs.printReplicationInfo()

# 查看slave oplog(同步)状态
rs.printSlaveReplicationInfo()

# 补充，赋权
db.grantRolesToUser("admin",
  [
    { role: "dbOwner", db: "admin" },
    { role: "clusterAdmin", db: "admin" },
    { role: "userAdminAnyDatabase", db: "admin" },
    { role: "dbAdminAnyDatabase", db: "admin" },
    { role: "root", db: "admin" },
    { role: "__system", db: "admin" }
  ]
)

```

### 创建数据库账号

```shell
use mydb
db.createUser(
  {
    user: "user",
    pwd: "user",
    roles: [
       { role: "readWrite", db: "mydb" }
    ]
  }
)
```

### 清理

rm -rf /data/replication/data/27017/*
rm -rf /data/replication/data/27018/*
rm -rf /data/replication/data/27019/*
rm -rf /data/replication/data/arb/*
rm -rf /data/replication/logs/*
