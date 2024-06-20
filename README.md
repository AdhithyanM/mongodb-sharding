## Set up Sharding using Docker Containers

![sharding and replica sets img](https://raw.githubusercontent.com/minhhungit/mongodb-cluster-docker-compose/master/images/sharding-and-replica-sets.png)

### Config servers

Start config servers (3 member replica set)

```
docker-compose -f config-server/docker-compose.yml up -d
```

Initiate replica set

```
mongo mongodb://192.168.1.81:40001
```

```
rs.initiate(
  {
    _id: "cfgrs",
    configsvr: true,
    members: [
      { _id : 0, host : "192.168.1.81:40001" },
      { _id : 1, host : "192.168.1.81:40002" },
      { _id : 2, host : "192.168.1.81:40003" }
    ]
  }
)

rs.status()
```

### Shard 1 servers

Start shard 1 servers (3 member replicas set)

```
docker-compose -f shard1/docker-compose.yml up -d
```

Initiate replica set

```
mongo mongodb://192.168.1.81:50001
```

```
rs.initiate(
  {
    _id: "shard1rs",
    members: [
      { _id : 0, host : "192.168.1.81:50001" },
      { _id : 1, host : "192.168.1.81:50002" },
      { _id : 2, host : "192.168.1.81:50003" }
    ]
  }
)

rs.status()
```

### Mongos Router

Start mongos query router

```
docker-compose -f mongos/docker-compose.yml up -d
```

### Add shard to the cluster

Connect to mongos

```
mongo mongodb://192.168.1.81:60000
```

Add shard

```
mongos> sh.addShard("shard1rs/192.168.1.81:50001,192.168.1.81:50002,192.168.1.81:50003")
mongos> sh.status()
```

## Adding another shard

### Shard 2 servers

Start shard 2 servers (3 member replicas set)

```
docker-compose -f shard2/docker-compose.yml up -d
```

Initiate replica set

```
mongo mongodb://192.168.1.81:50004
```

```
rs.initiate(
  {
    _id: "shard2rs",
    members: [
      { _id : 0, host : "192.168.1.81:50004" },
      { _id : 1, host : "192.168.1.81:50005" },
      { _id : 2, host : "192.168.1.81:50006" }
    ]
  }
)

rs.status()
```

### Add shard to the cluster

Connect to mongos

```
mongo mongodb://192.168.1.81:60000
```

Add shard

```
mongos> sh.addShard("shard2rs/192.168.1.81:50004,192.168.1.81:50005,192.168.1.81:50006")
mongos> sh.status()
```

## References

- [Database Sharding](https://www.mongodb.com/features/database-sharding-explained)
- [Sharding in MongoDB](https://www.mongodb.com/docs/manual/sharding/)
- [Sharding Setup in MongoDB](https://youtu.be/7Lp6R4CmuKE?si=-HgYAX6_wtO12zEV)
- [Adding a Shard to Sharded Cluster](https://youtu.be/LGERGvEaPW0?si=W2o4pKjxuSfZN7RQ)
- [Sharding a MongoDB Collection](https://youtu.be/Rwg26U0Zs1o?si=AnEBEMW5_21rGUba)
