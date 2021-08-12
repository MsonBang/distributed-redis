## Redis主从工作原理

    1.配置大体流程
        复制一份redis.conf文件，修改里面的内容，作为从节点的配置
        
        修改port：port 6380
        
        将pid进程号写入pidfile配置的文件: pidfile /var/run/redis_6380.pid
        
        指定数据存放目录 logfile "6380.log" /usr/loacal/opt/redis@4.0/data/6380
        
        配置主从辅助 replicaof 192.1680.60 6379 //从6379的Redis实例复制
        
        启动从节点 ./redis-server redis-6380.conf
        
        连接从节点 ./redis-ci -p 6380
        
        测试从节点是否能及时同步主节点修改数据
        
    2.步骤   
        （1）如果配置了从节点slave,从节点都会发送一个psync命令给master主节点请求复制数据。
        
        （2）master收到psync命令后，会在后台将数据持久化生成最新的rdb快照文件，持久化期间仍可接收客户端请求，
            并且将这些数据集缓存在内存中。持久化完毕，master将rdb发送给slave。
            
        （3）slave接收到rdb,然后记载到内存中，然后master将之前缓存的命令发送给slave。
        
        （4）当主从节点断开时，slave会自动重连master。
        
        （5）如果master收到了多个slave节点的请求，只会持久化一次并方法送个每个slave。
        
    3.数据部分复制(断点续传)
    
        3.1 什么是部分复制？
        
            当master和slave断开重连之后，从节点都会将主节点所有数据整体复制过来。在Redis2.8之后，支持从master部分数据复制。
            也称为断点续传。
            
        3.2 断点续传
        
            （1）master会在内存中创建一个复制数据用的缓存队列，master和slave都维护了复制数据的下标offset和master进程ID。
            
            （2）当连接断开重连后，slave会请求master进行未完成的复制，从记录的下标开始。
            
            （3）如果master的进程id变了，那么slave就直接进行一次全量数据复制。
            
    4.主从复制风暴
    
        如果有很多从节点，多个从节点同时复制主节点导致主节点压力过大。
        
        可以将从节点继续向下扩展从节点。缓解主从复制风暴。
            
   
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    