<!--
Categories = ["Development", "Others"]
Description = ""
Tags = ["Development", "redis", "redis-cluster", "redis-shake", "redis-proxy", "Others"]
date = "2020-06-12T21:47:31-08:00"
title = "Development Tips"
-->

## Redis cluster and migrate data from standalone redis

Rencently our project need to upgrade redis servers. Before we use master-slave replication redis servers, and now we want to more stable redis cache as session storage. So we need to deploy a redis-cluster and also need to migrate data from standalone to cluster redis.


### I. Setup Redis Cluster
#### 1. download latest redis source code
#### 2. unzip it
   ```
   tar -vxf redis-6.0.5.tar.gz
   ```
#### 3. compilation
```shell
  cd redis-6.0.5/
  make
  make test
  *(You might need to install tcl to run test)
```
#### 4. cluster and server
   we use 3 servers to create the cluster

```
   ____________     ____________      ____________
  |            |   |            |    |            |
  | server 1   |   | server 2   |    | server 3   |
  | master A   |   | master B   |    | master C   |
  | slave  C   |   | slave  A   |    | slave  B   |
  |____________|   |____________|    |____________|

```
   
#### 5. prepare config file for each redis node.(node{name}.conf)
> here's the template
```
port {port}
#each node with a pid_file
pidfile /var/run/redis-cluster/{node_name}.pid
#this node's ip addr
bind {ip}

protected-mode no
cluster-enabled yes

cluster-config-file nodes-7003.conf
cluster-node-timeout 5000
appendonly yes

requirepass {password}
masterauth {password}

dbfilename dump.rdb
dir {path}/redis-cluster/node7003
```

**Use the template and edit it with each node's details.**

#### 6. Start nodes
```
nohup redis-server node_a/redis.conf &
```
#### 7. Create cluster
```
redis-cli --cluster create {node1_ip}:{node1_port}{node2_ip}:{node2_port} {node3_ip}:{node3_port} {node1s_ip}:{node1s_port} {node2s_ip}:{node2s_port} {node3s_ip}:{node3s_port} --cluster-replicas 1 -a {password}
```
**please fill with parameters in {}**

#### 8. test
```
redis-cli -c -h {ip} -p {port} -a {password}

set test 1
get test
keys *
```

### II. Redis proxy

Since we need to migrate old data from old redis to the new cluster. And we try to use **redis-shake** to complete the process. But currently **redis-shake** not support target cluster redis. So we have to use a proxy to bridge them.

1. download redis proxy and compilation
```
   git clone https://github.com/RedisLabs/redis-cluster-proxy.git

   cd redis-cluster-proxy
   make
```
2. start proxy
```
./redis-cluster-proxy --auth {password} {node1_ip}:{node1_port}{node2_ip}:{node2_port} {node3_ip}:{node3_port} {node1s_ip}:{node1s_port} {node2s_ip}:{node2s_port} {node3s_ip}:{node3s_port}
```

### III. migrate/sync data

Please follow this link:[[Use the redis-shake tool to migrate data from an RDB file]](https://partners-intl.aliyun.com/help/doc-detail/116378.htm)

