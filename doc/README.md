# How to set up Hawkular + Cassandra Cluster

## Requirements.

For this tutorial I'm going to assume that you have RedHat Atomic Host 7.2 and  docker installed on your system. In this tutorial we are going to see how we can setup our Cassandra cluster shrink or expand our Cassandra cluster.

## Setup

First of all you need to launch a container of Cassandra..
```
docker run  --name cassandra_seed -d -p 7000:7000 -e CASSANDRA_START_RPC=true \
brew-pulp-docker01.web.prod.ext.phx2.redhat.com:8888/jboss/cassandra:latest
```
With this command you will launch our first Cassandra node. You can see the container running:

```
# docker ps

CONTAINER ID        IMAGE                                                                         COMMAND                  CREATED             STATUS              PORTS                                                                              NAMES
34a75ba45be8        brew-pulp-docker01.web.prod.ext.phx2.redhat.com:8888/jboss/cassandra:latest   "/docker-entrypoint.s"   44 seconds ago      Up 44 seconds       7001/tcp, 7199/tcp, 9042/tcp, 0.0.0.0:7000->7000/tcp, 9160/tcp
```


You can see the node information with this command:
```
 docker exec -it <container_id> /opt/apache-cassandra/bin/nodetool info
```
Now you can launch our hawkular-service container and tell it to connect to Cassandra
```
docker run --name hawkular-service  -d \
          -e HAWKULAR_BACKEND=remote   \
          -e CASSANDRA_NODES=cassandra_seed \
          -p 8080:8080 -p 8443:8443 -p 9990:9990 \
          --link cassandra_seed:cassandra_seed pilhuhn/hawkular-services:latest
```

## How to expand the cluster

If for some reason you need to expand our cluster to have more than one node, if this is gonna be on the same box you can launch a new container with this command:

```
docker run -d -e CASSANDRA_SEEDS=cassandra_seed -d \
              -e CASSANDRA_START_RPC=true  \
              --link cassandra_seed  \
              brew-pulp-docker01.web.prod.ext.phx2.redhat.com:8888/jboss/cassandra:latest
```
After launching the container, you can check the status of the node with nodetool.
```
#  docker exec -it 34a75ba45be8 /opt/apache-cassandra/bin/nodetool status

Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.2  490.35 KB  256          50.2%             ead3a0ee-b040-4873-8b62-aa700d02b0c1  rack1
UN  172.17.0.4  312.28 KB  256          49.8%             8e4a73eb-5545-48e1-9464-ef2728f8852e  rack1
```

Once the node is up, you need to execute a cleanup process on the other nodes of the cluster, you can do this operation latter
Repeat this process for each node you want to add to the cluster.

```
docker exec -it <container_id> /opt/apache-cassandra/bin/nodetool cleanup
```

## How to shrink the cluster

If for some reason you need to remove a node form the cluster, you need to follow the following steps:

1. Select a node you want to remove form the cluster, you can see all containers running with `docker ps`
2. Once you selected a node check whether the node is up or down using `nodetool status`
3. You need to run a decommission process, this will assign the ranges the old node was responsible for to other nodes, and replicate the appropriate data.
```
  docker exec -it <container_id> /usr/bin/nodetool decommission
```
You can monitor the progress using  ` docker exec -it <container_id> /opt/apache-cassandra/bin/nodetool netstats`

```
docker exec -it <container_id> /opt/apache-cassandra/bin/nodetool netstats

Mode: DECOMMISSIONED
Not sending any streams.
Read Repair Statistics:
```

4. Once the process finish you can safely delete the container
```
  docker stop <container_id>
  docker rm <container_id>
```
or
```
  docker rm --force  <container_id>
```
