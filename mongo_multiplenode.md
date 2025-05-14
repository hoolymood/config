###### Для работы с транзакциями монго требует наличие механизма replicaSet. Ниже вариант как поднять локально

#####  multiple node replica set by docker

```bash
   docker network create mongodb-network

docker run -d -p 27017:27017 --name mongodb  --network mongodb-network mongo:6.0.4 mongod --replSet myReplicaSet --bind_ip localhost,mongodb
docker run -d -p 27018:27017 --name mongodb2 --network mongodb-network mongo:6.0.4 mongod --replSet myReplicaSet --bind_ip localhost,mongodb2
docker run -d -p 27019:27017 --name mongodb3 --network mongodb-network mongo:6.0.4 mongod --replSet myReplicaSet --bind_ip localhost,mongodb3

docker exec -it mongodb mongosh --eval "rs.initiate({
 _id: \"myReplicaSet\",
 members: [
   {_id: 0, host: \"mongodb\"},
   {_id: 1, host: \"mongodb2\"},
   {_id: 2, host: \"mongodb3\"}
 ]
})"

docker exec -it mongodb mongosh --eval "rs.status()"

```
 
#####   single node replica set by docker

```bash
docker run -d -p 27017:27017 --name mongodb mongo:6.0.4 mongod --replSet myReplicaSet

docker exec -it mongodb mongosh --eval "rs.initiate({
 _id: \"myReplicaSet\",
 members: [
   {_id: 0, host: \"localhost\"}
 ]
})"

docker exec -it mongodb mongosh --eval "rs.status()"


```
#####   single node replica by compose

```
version: "3.9"
networks:
  mongodb-network:
    name: "mongodb-network"
    driver: bridge
services:
  mongodb:
    image: "mongo:6.0.4"
    container_name: "mongodb"
    networks:
      - mongodb-network
    hostname: "mongodb"
    ports:
      - "27017:27017"
    restart: "always"
    entrypoint: [ "/usr/bin/mongod", "--bind_ip_all", "--replSet", "myReplicaSet" ]
  mongoinit:
    image: "mongo:6.0.4"
    container_name: "mongodb_replSet_initializer"
    restart: "no"
    depends_on:
      - mongodb
    networks:
      - mongodb-network
    command: >
      mongosh --host mongodb:27017 --eval "rs.initiate({
       _id: \"myReplicaSet\",
       members: [
         {_id: 0, host: \"mongodb\"}
       ]
      })"


      docker exec -it mongodb mongosh --eval "rs.status()"
```

Спасибо https://pakisan.github.io/posts/docker-compose-mongodb-single-node-replica-set/
  