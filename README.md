# mongodb-sharding

## docker compose problem issue with volume

### First, we will configure our config servers replica set:
rs.initiate({_id: "mongors1conf",configsvr: true, members: [{ _id : 0, host : "mongocfg1" },{ _id : 1, host : "mongocfg2" }, { _id : 2, host : "mongocfg3" }]})


### After running the above command, we can check the status by running the following command on our first config server.
docker exec -it mongocfg1 bash -c "echo 'rs.status()' | mongo"


### Next, we will build our shard replica set.
docker exec -it mongors1n1 bash -c "echo 'rs.initiate({_id : \"mongors1\", members: [{ _id : 0, host : \"mongors1n1\" },{ _id : 1, host : \"mongors1n2\" },{ _id : 2, host : \"mongors1n3\" }]})' | mongo"

### Do this for the second shard too.
docker exec -it mongors2n1 bash -c "echo 'rs.initiate({_id : \"mongors2\", members: [{ _id : 0, host : \"mongors2n1\" },{ _id : 1, host : \"mongors2n2\" },{ _id : 2, host : \"mongors2n3\" }]})' | mongo"


### check status replica shard
docker exec -it mongors1n1 bash -c "echo 'rs.status()' | mongo"

### Finally, we will introduce our shard to the routers:
docker exec -it mongos1 bash -c "echo 'sh.addShard(\"mongors1/mongors1n1\")' | mongo "

### Now our routers, which are the interfaces of our cluster to the clients, have the knowledge about our shard. We can check the shard status by running the command below on the first router node:
docker exec -it mongos1 bash -c "echo 'sh.status()' | mongo "


### Now, our sharded cluster configuration is complete. We don’t have any databases yet. We will create a database and will enable sharding.
docker exec -it mongors1n1 bash -c "echo 'use someDb' | mongo"

### we should enable sharding on our newly created database.
docker exec -it mongos1 bash -c "echo 'sh.enableSharding(\"someDb\")' | mongo "

### Now, we have a sharding-enabled database on our sharded cluster! It’s time to create a collection on our sharded database.
docker exec -it mongors1n1 bash -c "echo 'db.createCollection(\"someDb.someCollection\")' | mongo "

### The collection is not sharded yet. We will shard it on a field named someField.
docker exec -it mongos1 bash -c "echo 'sh.shardCollection(\"someDb.someCollection\", {\"someField\" : \"hashed\"})' | mongo "

### The sharding key must be chosen very carefully because it is for distributing the documents throughout the cluster.