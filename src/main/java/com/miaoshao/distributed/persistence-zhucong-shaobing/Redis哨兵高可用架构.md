## Redis哨兵架构

    1.架构简介(sentinel)
    
        sentinel哨兵是特殊的Redis服务，不提供读写服务，主要用来监控Redis实例节点。
        
    2.主从节点通过sentinel连接访问
    
        （1）哨兵架构下，client端第一次从哨兵找出Redis的主节点，后续就直接访问Redis的逐渐点，不会每次都通过sentinel代理访问主节点。
        
        （2）当Redis主节点发生变化后，哨兵第一时间会感觉到，并且将新的节点信息通知给client端。（其实就是client端实现了订阅功能，
            订阅sentinel节点变动消息）
            
    3.Redis哨兵架构搭建步骤
    
        （1）在Redis配置文件目录找到 sentinel.conf，一般是和redis.conf在一个目录下。
            拷贝一份 cp sentinel.conf sentinel-2379.conf
            
        （2）修改一下配置值：
            port 26379
            daemonize yes
            pidfile "/var/run/redis-sentinel-26379.pid"
            logfile "26379.log"
            dir "/usr/local/opt/redis@4.0/data" 或者 "/usr/local/var/db/redis"
            sentinel monitor mymaster 192.168.30.201 6379 2  #mymaster名字随便取，客户端访问时会用到，这个2是个数字，表示
                当有多少个sentinel任务master节点失效时，(sentinel/2+1）master才算正式失效
            
        （3）启动哨兵实例
            ./redis-sentinel sentinel-26379.conf
         
         (4)查看sentinel的info信息
            ./redis-cli -p 26379
            info
         
         (5)按照以上步骤，继续再搭建26380，26381两个哨兵。注意数字都要修改好。然后启动。最后哨兵集群就有三个
         
     4.哨兵集群启动后工作流程
        
        （1）sentinel集群启动完毕后，哨兵集群的数据信息会写入到sentinel配置文件中（追加在文件最后面）
            例如 sentinel-26379.conf,如下所示：
            sentinel known-replica mymaster 192.168.30.201 6380   #从节点
            sentinel known-replica mymaster 192.168.30.201 6381   #从节点
            sentinel known-replica mymaster 192.168.30.201 26380  #感知到其他哨兵节点1
            sentinel known-replica mymaster 192.168.30.201 26381  #感知到其他哨兵节点2
            
        （2）如果当前Redis主节点挂了，哨兵会从新选举出新的Redis主节点。同事也会修改sentinel配置文件中的集群信息。比如6379住节点
            挂了，那么假设选举出的新的主节点为6380的，那么sentinel配置文件中会修改为下面所示：
            sentinel known-replica mymaster 192.168.30.201 6379   #从节点
            sentinel known-replica mymaster 192.168.30.201 6381   #从节点
            sentinel known-replica mymaster 192.168.30.201 26380  #感知到其他哨兵节点1
            sentinel known-replica mymaster 192.168.30.201 26381  #感知到其他哨兵节点2
            很明显看到，6379和6381成了从节点，6380成为了主节点。
            这时候你会发现，配置文件中的有个地方也自动变了，那就是 sentinel monitor mymaster 192.168.30.201 6380
            
        （3）JAVA中实战测试，具体代码在本项目中JedisSentinelTest()方法可使用。
         
            
        
    