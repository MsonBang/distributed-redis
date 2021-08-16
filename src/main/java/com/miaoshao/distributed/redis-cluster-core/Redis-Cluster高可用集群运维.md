## Redis集群方案比较

    1.Redis哨兵集群
        （1）在Redis3.0之前的版本实现集群都是借助哨兵sentinel工具来监控master节点的状态。如果master节点异常，则会做主从切换，将
            某一台slave作为master
        （2）哨兵的配置略微复杂，并且性能一般。在主从切换瞬间存在"访问瞬断"的情况。
        （3）哨兵模式只有一根主节点对提供服务，没法支持很高的并发，单个节点内存不能设置过大，否则持久化文件过大，影响数据恢复和主从同步效率。
   
    2.Redis高可用集群
        Redis集群是一个由【多个主从节点组成的分布式服务器群】，具有复制、高可用和分片特性。
        Redis集群不需要哨兵，也能完成节点移除和故障转移的功能。
        将每个节点设置为集群模式，这种集群模式没有中心节点，可水平扩展，性能和高可用性都比哨兵模式要好。配置非常简单。
        
## Redis高可用集群搭建
    
    1.Redis集群至少要三个master节点，我们就搭建三个master节点，并且给每个master再搭建一个slave节点。总共6个节点。可以使用三台机器
        每台机器一主一从，步骤如下：
        
        1.1 第一步：在第一台机器/usr/local下创建文件夹 redis-cluster，然后再其下面分别创建2个文件夹
            mkdir =p /usr/local/etc/redis-cluster
            mkdir 8001 8004
            
        2.2 第二步：复制一份redis.conf放入8001，8004，然后修改如下配置：
            (1) daemonize yes
            (2) port 8001
            (3) pidfile /var/run/redis_8001.pid #把pid进程号写入pidfile配置的文件
            (4) dir /usr/local/etc/redis-cluster/8001/
            (5) cluster-enabled yes #开启集群模式
            (6) cluster-config-file nodes-8001.conf
            (7) cluster-node-timeout 10000
            (8) #bind 127.0.0.1 注释掉即可
            (9) protected-mode no #关闭保护模式
            (10) appendonly yes
            (11) requirepass golaxy
            (12) masterauth golaxy
            
        2.3 第三步：按照以上方式，分别完成配置6个节点配置文件：8001-8004  8002-8005  8003-8006  
        
   
        2.4 第四步：分别启动6个Redis实例，然后检查是否全部启动成功
            ./redis-server  /usr/local/etc/8001/redis.conf &   #启动Redis
            ps -ef|grep redis #查看是否启动成功
            
        2.5 第五步：用Redis-cli创建整个Redis集群。实现一下命令，需要6个节点，也就是三个机器之间Redis可以互相访问。
            关闭防火墙：
                systemctl stop firewalld   临时关闭
                systemctl disable firewalld 禁止开机启动
            启动集群命令：
                ./redis-cli -a golaxy --cluster create --cluster-replicas 1 
                192.168.30.201:8001 192.168.30.201:8002 192.168.30.201:8003 192.168.30.201:8004
                192.168.30.201:8005 192.168.30.201:8006
                
        2.6 第六步：验证集群
            (1) 连接任意一个集群 ./redis-cli -c -h -p (-a表示访问服务器密码 -c表示集群模式 -h和-p表示指定ip和端口)
                eg: ./redis-cli -a miaoshao111 -c -h 192.168.30.201 -p 8001
            (2) cluster info 查看集群信息
                cluster nodes 查看集群节点列表
            (3) 进行数据操作验证
            (4) 关闭集群需要逐个进行关闭，使用命令：
                ./redis-cli -a miaoshao111 -c -h 192.168.30.201 -p 8001 shutdown
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
             
    
        
