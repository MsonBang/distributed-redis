##Redis数据存储结构
    1.字符串String
        单值缓存  set key value / get key
        多个键值对缓存 mset key value key value key value / mget key key key
        对象缓存  set user:json格式数据 / mset user1:name zhangsan user1:age 18
                mget user1:name  user1：age
        分布式锁  setnx product:10001 true 返回1代表取锁成功，返回0代表获取锁失败
        计数器   incr article:readcount:文章ID / get article:readcount:文章ID
        web集群session共享  Spring session + redis实现session共享
        分布式系统全局系列号 incrby orderid 1000 //Redis批量生成序列号提升性能

*********************************************************************************

    2.哈希hash
        对象缓存 hmset user userid:name zhangsan userid:age 18
                hmset user 001:name zhangsan 001:age 18
                hmget user 001:name 001:age
        电商购物车 hset cart:1001 10088 1 购物车1001用户的10088添加一个商品
                 hincrby cart:1001 10088 1 购物车1001用户10088商品数量+1
                 hlen cart:1001 1001用户商品总数
                 hdel cart:1001 10088 删除1001用户购物车
                 hgetall cart：1001 获取1001用户购物车所有商品
    Hash和String优缺点
    优点：相比String操作消耗内存与CPU更小，更节省空间
    缺点：Redis集群架构下不适合大规模使用
   
********************************************************************************* 
    
    3.列表list
        常用命令：
            lpush key value[vlaye,....]  向列表左侧插入value
            rpush key valye[value,....]  向列表右侧插入去拉也
            lpop key 获取左侧value
            rpop key 获取右侧value
            Stack栈 = lpush + lpop 实现了栈的后进先出
            Queue队列 = lpush + rpop 实现了队列的先进先出
            Blocking MQ阻塞队列 = lpush + brpop 实现了阻塞队列，如果队列没有东西，一直在获取，类似mq的消费者
            lrange start stop 0 4  获取列表最新的0-4五个元素
        应用场景1：微博消息和微信公众号消息
            微博主tom发了一个微博， lpush msg:miaoshao 1008 给我的消息列表添加一个1008的消息ID
            微博住jery发了一个微博,lpush msg:miaoshao 1009 个我的消息列表左侧添加一个1009的消息ID
            微博和微信公众号消息流 lrange msg:miaoshao 0 2 获取从0索引开始，取2个消息。          
              
*********************************************************************************
    
    4.集合set
        常用命令：
            sadd key mumber[member...]
            smembers key 查看key对应的所有values
            srandmember key 3 随机获取3个元素
            sismember key value 查看是否存在于集合中
            scard key 获取集合的个数
            
            sinter set1 set2 set3 求三个集合交集
            sunion set1 set2 set3 三个集合并集，去重加到一起
            sdiff set1 set2 set3 差集：第一个集合减去后面两个集合的并集。
        应用场景1：抽奖小程序
            sadd key userid 点击参与抽奖加入集合
            smembers key 查看所有参与抽奖的用户
            srandmember key 3 随机取3个元素
            spop key 3 抽出来3个，同时集合删除这三个元素。
        应用场景2：微信微博点赞，收藏，标签
            sadd like:消息001 用户001 / sadd like:消息002 用户002
            srem like:消息001 用户001 删除这条点赞记录
            sismember like:消息001 用户001 查看是否在这条集合里面。适用于查看自己是否点赞过。
            smembers like:消息001 获取所有用户列表
            scard liek:消息001 获取这条消息点赞的用户数。
        应用场景3：集合操作实现微博微信-关注模型
            共同关注求交集 sinter set1 set2
            我关注的人也关注他 sismmber set1 用户001
            我可能认识的人 sdiff set1 set 相当于set-set2，剩余的可能就是认识的
        应用场景4：进行集合的帅选，条件过滤，使用到了sinter交集的set命令
            sadd brand:huawei P40
            sadd brand:xiaomi mi-10
            sadd brand:iphone phone12
            sadd os:android P40 mi-10
            sadd cpu:brand:intel P40 mi-10
            sadd ram:8G P40 mi-10 iphone12
            
            现在想查询安卓系统,cpu为inter并且内存为8G的：
            sinter os:android cpu:barand:inter ram:8G
            
*********************************************************************************
    
    5.有序集合zset（课程到1：43，8.11继续开始）
        zadd key score member 分值
        zrem key member 删除key
        ##排序,按照分值从大到小排序
        zincrby hotnews:20190101 1 中国金牌榜
        zincrby hotnews:20190101 2 美国金牌榜
        zrange hotnews:20190101 0 9 withscores
        
*********************************************************************************

##Redis的单线程和高性能
    1.Redis是单线程吗？
        （1）redis的单线程主要是指【redis的网络IO】和【键值对读写】是一个线程完成的。
        这就是Redis对外提供键值存储服务的主要流程。ll
        （2）但是Redis的其他功能是额外的线程执行的。例如持久化、集群数据同步等。
        
    2.Redis单线程为什么速度效率很快？
        （1）因为Redis的数据都在内存中，所有的运算都是内存级别的运算，单线程避免了多个线程切换性能损耗。
        注意：对于耗时的指令，就减少使用。比如keys命令。如果key键很大量，会导致redis卡顿。
        
    3.既然是单线程，那么如何处理大量的并发客户端连接？
         Redis的多路复用：
            Redis利用epoll来实现IO多路复用，将连接信息和事件放到队列中。
            队列-----》文件事件分排器-----》事件处理器
            
##其他高级命令
    查看Redis最大连接数：
        config get maxclients
        
    keys：全量遍历键，列出所有key
        keys *    keys+后面可以使用正则表达式查询特定的key值
        
    scan: 渐进式遍历 scan cursor[游标值] key的正则表达式 扫描数量
        scan 0 miaoshao* 3
        scan 10 miaoshao* 3
        
    info:查看Redis运行信息，分为9个模块。
        server
        clients
        memory
        persistence
        stats
        replication
        CPU
        cluster
        keySpace
        
        