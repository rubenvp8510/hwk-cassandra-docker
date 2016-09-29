# Hawkular+Cassandra Cluster+Docker

## Cluster on the same box (multiple containers)

Assuming that you are on the folder  where you have this repo files.

```
docker-compose up
```
or

```
docker-compose  -f docker-compose.yml up
```

This start a hawkular-service container and two C* nodes, one of them is the seed.

This is the status of the cluster after executing the command:

```
docker exec -it dockercluster_cassandra_seed_1 nodetool status
```

Output:
```
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.2  584.2 KiB  256          49.6%             b50ca5ec-9599-402f-a347-16d5cbfb1790  rack1
UN  172.17.0.4  667.35 KiB  256          50.4%             fa08f43d-b8fd-4d69-9227-6db800e0b29d  rack1
```

For scale the nodes in the same container you can use docker-compose scale option:

```
docker-compose scale cassandra_node=2 (for 2 nodes)
```

## For nodes in different boxes.

### Launch the seed + hawkular services

If you want to be able to communicate to other nodes outside the world you need to set the broadcast address,
this is because you need to tell Cassandra what IP address to advertise to the other nodes.

```
BROADCAST_ADDRESS=<your_public_address> docker-compose -f docker-public.yml up
```
or you can use the script on this repo:
```
./cassandra_public -b <box_public_ip>
```

### Join other nodes to the cluster.

For join other box to the cluster:
```
docker run --name cassandra_node  -d \
      -e CASSANDRA_START_RPC=true \
      -e CASSANDRA_SEEDS=<ip_seed> \
      -e CASSANDRA_BROADCAST_ADDRESS="<box_public_ip>" \
      -p 7000:7000 -p 9160:9160 cassandra:3.7
```

or you can use this command (for lazy people like me.):

```
./launch_node -s <seed_ip> -b <box_public_ip>
```


# Other scripts
You can execute on the seed box and it display cluster information

```
./status
```
Delete ALL docker containers
```
./clear_containers
```
