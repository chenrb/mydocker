### 场景
1~2亿条数据需要缓存，请问如何设计这个存储案例
```
单机无法存储，需要使用分布式存储
· 哈希取余分区
    hash(key) % N计算出hash值，用来决定数据存储在哪个节点上
    缺点：该方案最大的问题是，当新增或删减节点时，节点数量发生变化，系统中所有的数据都需要重新计算映射关系，引发大规模数据迁移。
· 一致性哈希算法
    目的：解决分布式缓存数据变动核映射问题，当服务器个数发生变化时，尽量减少影响客户端到服务器的映射关系。
    哈希环：一致性Hash算法将整个哈希值空间组织成一个虚拟的圆环，[0, 2^32-1]
    节点映射：将集群重的各个IP节点映射到某个环上
    落键规则：当需要存储一个kv键值对时，首先计算key的hash值，hash(key)，将这个key使用相同的函数hash计算出哈希值并确定此数据在环上的位置，从此位置沿环顺时针走，遇到的第一台服务器就是应该存储的服务器。
    如果一台服务器受到影响，则受到影响的数据仅仅是此服务器到其环空间中前一台服务器。（即沿着逆时针方向行走遇到的第一台服务器）之间的数据。
    优点：加入或删除节点只影响哈希环中顺时针方向的相邻的节点，对其他节点无影响
    缺点：数据倾斜问题，数据的分布和节点的位置相关，因为这些节点不是均匀的分布在哈希环上的，所以数据在进行存储时达不到均匀分布的效果
· 哈希槽分区
    哈希槽实质上是一个数组，数组[0,2^14-1]形成hash slot空间。
    在数据和节点之间又加入了一层，把这层称为哈希槽(slot)，用于管理数据和节点之间的关系。Redis集群中内置了16384个哈希槽，redis会根据节点数量大致均等的将哈希槽映射到不同的节点。当需要在Redis集群中放置一个key-value时，redis先对key使用crc16算法算出一个结果，然后把结果对16384求余数，这样每个key都会对应一个编号在0--16383之间的哈希槽，也就是映射到某个节点上。
    优点：哈希取余分区思路非常简单：计算key的hash值，然后对节点数量进行取余，从而决定数据映射到哪个节点上。
    特点：
        解耦数据和节点之间直接关系, 一致性hash计算方式是key & 节点个数, 映射的结果和节点个数有关系, 但是使用hash槽计算方式是 CRC16(key) % 槽的个数, 所以就解耦了数据和节点的关系
        使用哈希槽的好处就在于可以方便的添加或移除节点。
        1、当需要增加节点时，只需要把其他节点的某些哈希槽挪到新节点就可以了；
        2、当需要移除节点时，只需要把移除节点上的哈希槽挪到其他节点就行了；
        3、在这一点上，我们以后新增或移除节点的时候不用先停掉所有的 redis 服务甚至不用停到任何一个节点的服务。对应槽的采用直接复制数据过去显然比rehash快很多
节点自身维护槽的映射关系, 不需要客户端和代理服务器进行维护处理
```

### 3主3从redis集群扩缩容篇日志案例说明
```
步骤
1、起6个redis容器
2、构建容器主从关系
docker run -d --name redis-node-1 --net host --privileged=true -v /data/redis/share/redis-node-1:/data/redis-cluster-data redis --cluster-enabled yes --appendonly yes --port 6381 
docker run -d --name redis-node-2 --net host --privileged=true -v /data/redis/share/redis-node-2:/data/redis-cluster-data redis --cluster-enabled yes --appendonly yes --port 6382 
docker run -d --name redis-node-3 --net host --privileged=true -v /data/redis/share/redis-node-3:/data/redis-cluster-data redis --cluster-enabled yes --appendonly yes --port 6383 
docker run -d --name redis-node-4 --net host --privileged=true -v /data/redis/share/redis-node-4:/data/redis-cluster-data redis --cluster-enabled yes --appendonly yes --port 6384 
docker run -d --name redis-node-5 --net host --privileged=true -v /data/redis/share/redis-node-5:/data/redis-cluster-data redis --cluster-enabled yes --appendonly yes --port 6385 
docker run -d --name redis-node-6 --net host --privileged=true -v /data/redis/share/redis-node-6:/data/redis-cluster-data redis --cluster-enabled yes --appendonly yes --port 6386 
## 构建主从关系
docker exec -it 231965d874a4 /bin/bash
redis-cli --cluster create 127.0.0.1(host):6381 127.0.0.1(host):6382 127.0.0.1(host):6383 127.0.0.1(host):6384 127.0.0.1(host):6385 127.0.0.1(host):6386 --cluster-replicas 1
root@chenrb:/data# redis-cli -p 6381
127.0.0.1:6381> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:95
cluster_stats_messages_pong_sent:99
cluster_stats_messages_sent:194
cluster_stats_messages_ping_received:94
cluster_stats_messages_pong_received:95
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:194
127.0.0.1:6381> cluster nodes
9ce7b3610cf8932546099063bbeb052cfbfbab43 127.0.0.1(host):6386@16386 slave 7d36b5199264a5c32245e319aafdca08bee893ff 0 1647676935507 1 connected
139579173558e30da9c2398739f7b5ea68dbf682 127.0.0.1(host):6385@16385 slave 90e0479a453910887a0bad5a638852c1f57c5092 0 1647676934504 3 connected
1f3deadf85af5b9164ae2ef7fdfcc3355d8c1ae6 127.0.0.1(host):6382@16382 master - 0 1647676933000 2 connected 5461-10922
90e0479a453910887a0bad5a638852c1f57c5092 127.0.0.1(host):6383@16383 master - 0 1647676936510 3 connected 10923-16383
7d36b5199264a5c32245e319aafdca08bee893ff 127.0.0.1(host):6381@16381 myself,master - 0 1647676935000 1 connected 0-5460
cc4fd514ff8feb4bd1b3307b40bf0d77c4fd2252 127.0.0.1(host):6384@16384 slave 1f3deadf85af5b9164ae2ef7fdfcc3355d8c1ae6 0 1647676935000 2 connected
127.0.0.1:6381>
M     S
1     6
2     4
3     5
```
### 扩缩容
```
扩容步骤
1、新增2个redis节点
2、挂载master节点到集群
3、重新分配哈希槽
4、挂载slave节点
docker run -d --name redis-node-7 --net host --privileged=true -v /data/redis/share/redis-node-7:/data/redis-cluster-data redis --cluster-enabled yes --appendonly yes --port 6387 
docker run -d --name redis-node-8 --net host --privileged=true -v /data/redis/share/redis-node-8:/data/redis-cluster-data redis --cluster-enabled yes --appendonly yes --port 6388 
redis-cli --cluster check 127.0.0.1(host):6381
docker exec -it redis-node-7 /bin/bash
redis-cli --cluster add-node ip:6387 ip:6386
redis-cli --cluster reshard ip:6386 ///不是全部重新分配，其他节点移动部分
redis-cli --cluster add-node ip:6388 ip:6387 --cluster-slave --cluster-master-id xxx
缩容步骤
1、卸载slave节点
2、重新分配哈希槽位
3、卸载master节点
4、停止、删除两个新增节点容器
redis-cli --cluster del-node ip:6388 id
redis-cli --cluster reshard ip:6386 // 槽位选择
redis-cli --cluster del-node ip:6387 id
```
