>

### zabbix

#### 监控

为什么监控。有哪些监控，区别，优缺点

详细见 个人github托管界面：

<https://tigerfivegit.github.io/2018/12/14/%E7%9B%91%E6%8E%A7%E4%BD%93%E7%B3%BB/>

<https://tigerfivegit.github.io/2018/12/29/%E5%BC%80%E6%BA%90%E8%BF%90%E7%BB%B4%E7%9B%91%E6%8E%A7%E4%BA%A7%E5%93%81%E7%AF%87/>



#### Zabbix监控什么？

| 分类     | 监控项                                                       |
| -------- | ------------------------------------------------------------ |
| 硬件监控 | 温度、硬件故障等                                             |
| 系统监控 | CPU、内存、硬盘、网卡流量、TCP状态、进程数                   |
| 应用监控 | Nginx、Tomcat、PHP、MySQL、Redis等                           |
| 日志监控 | 系统日志、服务日志、访问日志、错误日志                       |
| 安全监控 | WAF、敏感文件监控                                            |
| API监控  | 可用性、接口请求、响应时间                                   |
| 业务监控 | 例如电商网站每分钟产生多少订单、注册多少用户、多少活跃用户、推广效果如何 |
| 流量监控 | 根据流量获取用户相关信息。例如用户地理位置、某页面访问状况、页面停留时间等 |

#### zabbix常用监控项

```
zabbix自带的常用监控项
agent.ping 检测客户端可达性、返回nothing表示不可达。1表示可达
system.cpu.load --检测cpu负载。返回浮点数
system.cpu.util -- 检测cpu使用率。返回浮点数
vfs.dev.read -- 检测硬盘读取数据，返回是sps.ops.bps浮点类型，需要定义1024倍
vfs.dev.write -- 检测硬盘写入数据。返回是sps.ops.bps浮点类型，需要定义1024倍
net.if.out[br0] --检测网卡流速、流出方向，时间间隔为60S
net-if-in[br0] --检测网卡流速，流入方向（单位：字节） 时间间隔60S
proc.num[]  目前系统中的进程总数，时间间隔60s
proc.num[,,run] 目前正在运行的进程总数，时间间隔60S
###处理器信息
通过zabbix_get 获取负载值
合理的控制用户态、系统态、IO等待时间剋保证进程高效率的运行
系统态运行时间较高说明进程进行系统调用的次数比较多，一般的程序如果系统态运行时间占用过高就需要优化程序，减少系统调用
io等待时间过高则表明硬盘的io性能差，如果是读写文件比较频繁、读写效率要求比较高，可以考虑更换硬盘，或者使用多磁盘做raid的方案
system.cpu.swtiches --cpu的进程上下文切换，单位sps，表示每秒采样次数，api中参数history需指定为3
system.cpu.intr  --cpu中断数量、api中参数history需指定为3
system.cpu.load[percpu,avg1]  --cpu每分钟的负载值，按照核数做平均值(Processor load (1 min average per core))，api中参数history需指定为0
system.cpu.load[percpu,avg5]  --cpu每5分钟的负载值，按照核数做平均值(Processor load (5 min average per core))，api中参数history需指定为0
system.cpu.load[percpu,avg15]  --cpu每5分钟的负载值，按照核数做平均值(Processor load (15 min average per core))，api中参数history需指定为0

zabbix的自定义常用项
####内存相关
vim /usr/local/zabbix/etc/zabbix_agentd.conf.d/catcarm.conf
UserParameter=ram.info[*],/bin/cat  /proc/meminfo  |awk '/^$1:{print $2}'
ram.info[Cached] --检测内存的缓存使用量、返回整数，需要定义1024倍
ram.info[MemFree] --检测内存的空余量，返回整数，需要定义1024倍
ram.info[Buffers] --检测内存的使用量，返回整数，需要定义1024倍

####TCP相关的自定义项
vim /usr/local/zabbix/share/zabbix/alertscripts/tcp_connection.sh
#!/bin/bash
function ESTAB { 
/usr/sbin/ss -ant |awk '{++s[$1]} END {for(k in s) print k,s[k]}' | grep 'ESTAB' | awk '{print $2}'
}
function TIMEWAIT {
/usr/sbin/ss -ant | awk '{++s[$1]} END {for(k in s) print k,s[k]}' | grep 'TIME-WAIT' | awk '{print $2}'
}
function LISTEN {
/usr/sbin/ss -ant | awk '{++s[$1]} END {for(k in s) print k,s[k]}' | grep 'LISTEN' | awk '{print $2}'
}
$1

vim /usr/local/zabbix/etc/zabbix_agentd.conf.d/cattcp.conf
UserParameter=tcp[*],/usr/local/zabbix/share/zabbix/alertscripts/tcp_connection.sh $1

tcp[TIMEWAIT] --检测TCP的驻留数，返回整数
tcp[ESTAB]  --检测tcp的连接数、返回整数
tcp[LISTEN] --检测TCP的监听数，返回整数

####nginx相关的自定义项

vim /etc/nginx/conf.d/default.conf
    location /nginx-status
    {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }

    
vim /usr/local/zabbix/etc/zabbix_agentd.conf.d/nginx.conf
UserParameter=Nginx.active,/usr/bin/curl -s "http://127.0.0.1:80/nginx-status" | awk '/Active/ {print $NF}'
UserParameter=Nginx.read,/usr/bin/curl -s "http://127.0.0.1:80/nginx-status" | grep 'Reading' | cut -d" " -f2
UserParameter=Nginx.wrie,/usr/bin/curl -s "http://127.0.0.1:80/nginx-status" | grep 'Writing' | cut -d" " -f4
UserParameter=Nginx.wait,/usr/bin/curl -s "http://127.0.0.1:80/nginx-status" | grep 'Waiting' | cut -d" " -f6
UserParameter=Nginx.accepted,/usr/bin/curl -s "http://127.0.0.1:80/nginx-status" | awk '/^[ \t]+[0-9]+[ \t]+[0-9]+[ \t]+[0-9]+/ {print $1}'
UserParameter=Nginx.handled,/usr/bin/curl -s "http://127.0.0.1:80/nginx-status" | awk '/^[ \t]+[0-9]+[ \t]+[0-9]+[ \t]+[0-9]+/ {print $2}'
UserParameter=Nginx.requests,/usr/bin/curl -s "http://127.0.0.1:80/nginx-status" | awk '/^[ \t]+[0-9]+[ \t]+[0-9]+[ \t]+[0-9]+/ {print $3}'

PHP.listenqueue --检测PHP队列数，返回整数
PHP.idle --检测PHP空闲进程数，返回整数
PHP.active --检测PHP活动进程数，返回整数
PHP.conn --检测PHP请求数,返回整数
PHP.reached --检测PHP达到限制次数，返回整数
PHP.requets --检测PHP慢请求书，返回整数

####redis相关的自定义项
vim /usr/local/zabbix/etc/zabbix_agentd.conf.d/redis.conf
UserParameter=Redis.Status,/usr/local/redis/bin/redis-cli -h 127.0.0.1 -p 6379 ping |grep -c PONG
UserParameter=Redis_conn[*],/usr/local/redis/bin/redis-cli -h $1 -p $2 info | grep -w "connected_clients" | awk -F':' '{print $2}'
UserParameter=Redis_rss_mem[*],/usr/local/redis/bin/redis-cli -h $1 -p $2 info | grep -w "used_memory_rss" | awk -F':' '{print $2}'
UserParameter=Redis_lua_mem[*],/usr/local/redis/bin/redis-cli -h $1 -p $2 info | grep -w "used_memory_lua" | awk -F':' '{print $2}'
UserParameter=Redis_cpu_sys[*],/usr/local/redis/bin/redis-cli -h $1 -p $2 info | grep -w "used_cpu_sys" | awk -F':' '{print $2}'
UserParameter=Redis_cpu_user[*],/usr/local/redis/bin/redis-cli -h $1 -p $2 info | grep -w "used_cpu_user" | awk -F':' '{print $2}'
UserParameter=Redis_cpu_sys_cline[*],/usr/local/redis/bin/redis-cli -h $1 -p $2 info | grep -w "used_cpu_sys_children" | awk -F':' '{print $2}'
UserParameter=Redis_cpu_user_cline[*],/usr/local/redis/bin/redis-cli -h $1 -p $2 info | grep -w "used_cpu_user_children" | awk -F':' '{print $2}'
UserParameter=Redis_keys_num[*],/usr/local/redis/bin/redis-cli -h $1 -p $2 info | grep -w "$$1" | grep -w "keys" | grep db$3 | awk -F'=' '{print $2}' | awk -F',' '{print $1}'
UserParameter=Redis_loading[*],/usr/local/redis/bin/redis-cli -h $1 -p $2 info | grep loading | awk -F':' '{print $$2}'

Redis.Status --检测Redis运行状态， 返回整数
Redis_conn  --检测Redis成功连接数，返回整数
Redis_rss_mem  --检测Redis系统分配内存，返回整数
Redis_lua_mem  --检测Redis引擎消耗内存，返回整数
Redis_cpu_sys --检测Redis主程序核心CPU消耗率，返回整数
Redis_cpu_user --检测Redis主程序用户CPU消耗率，返回整数
Redis_cpu_sys_cline --检测Redis后台核心CPU消耗率，返回整数
Redis_cpu_user_cline --检测Redis后台用户CPU消耗率，返回整数
Redis_keys_num --检测库键值数，返回整数
Redis_loding --检测Redis持久化文件状态，返回整数



mysql:
version:数据库版本
key_buffer_size:myisam的索引buffer大小
sort_buffer_size:会话的排序空间（每个线程会申请一个）
join_buffer_size:这是为链接操作分配的最小缓存大小，这些连接使用普通索引扫描、范围扫描、或者连接不适用索引
max_connections:最大允许同时连接的数量
max_connect_errors：允许一个主机最多的错误链接次数，如果超过了就会拒绝之后链接（默认100）。可以使用flush hosts命令去解除拒绝
open_files_limits:操作系统允许mysql打开的文件数量，可以通过opened_tables状态确定是否需要增大table_open_cache,如果opened_tables比较大且一直还在增大说明需要增大table_open_cache
max-heap_tables_size:建立的内存表的最大大小（默认16M）这个参数和tmp_table_size一起限制内部临时表的最大值(取这两个参数的小的一个），如果超过限制，则表会变为innodb或myisam引擎，（5.7.5之前是默认是myisam，5.7.6开始是innodb，可以通过internal_tmp_disk_storage_engine参数调整）。
max_allowed_packet:一个包的最大大小
##########GET INNODB INFO
#INNODB variables
innodb_version:
innodb_buffer_pool_instances：将innodb缓冲池分为指定的多个（默认为1）
innodb_buffer_pool_size:innodb缓冲池大小、5.7.5引入了innodb_buffer_pool_chunk_size,
innodb_doublewrite：是否开启doublewrite（默认开启）
innodb_read_io_threads:IO读线程的数量
innodb_write_io_threads:IO写线程的数量
########innodb status
innodb_buffer_pool_pages_total:innodb缓冲池页的数量、大小等于innodb_buffer_pool_size/(16*1024)
innodb_buffer_pool_pages_data:innodb缓冲池中包含数据的页的数量
########## GET MYSQL HITRATE
1、查询缓存命中率
如果Qcache_hits+Com_select<>0则为 Qcache_hits/（Qcache_hits+Com_select），否则为0

2、线程缓存命中率
如果Connections<>0,则为1-Threads_created/Connections，否则为0

3、myisam键缓存命中率
如果Key_read_requests<>0,则为1-Key_reads/Key_read_requests，否则为0

4、myisam键缓存写命中率
如果Key_write_requests<>0,则为1-Key_writes/Key_write_requests，否则为0

5、键块使用率
如果Key_blocks_used+Key_blocks_unused<>0，则Key_blocks_used/（Key_blocks_used+Key_blocks_unused），否则为0

6、创建磁盘存储的临时表比率
如果Created_tmp_disk_tables+Created_tmp_tables<>0,则Created_tmp_disk_tables/（Created_tmp_disk_tables+Created_tmp_tables），否则为0

7、连接使用率
如果max_connections<>0，则threads_connected/max_connections，否则为0

8、打开文件比率
如果open_files_limit<>0，则open_files/open_files_limit，否则为0

9、表缓存使用率
如果table_open_cache<>0，则open_tables/table_open_cache，否则为0

```

#### zabbix其他须掌握技能

自定义监控

自动发现 --> 匹配模板 

自动注册 --> 匹配模板

分段监控

细化报警收件人



### Nginx

#### nginx配置的结构

```
全局配置
events {
    
   
}

log_format   ;
logs         ;

http {                        //只能有一个
upstream name {               //可以多个
    
}    

server {                       //可以多个，在http里边
    listen  ;
    server_name _；
    location 匹配项 {           //可以多个，在server里边
       
    }
    location 匹配项 {
      proxy_pass http://name  ;  
    }    
}
    
    
}

include   ；

以上未nginx配置文档的结构，必知必会

重点复习
https   ssl   443
tcp
status
验证
防盗链
地址重写  rewrite
流量控制  


配置文件过多，使用include，进行拆分

nginx -t
nginx -s reload
nginx -c 文件路径
nginx -v
```



#### nginx负载均衡

upstream 、 proxy_pass    权重、算法、(down、backup、) 、timeout、次数

upstream --> proxy_pass 



带端口的负载均衡



#### nginx反向代理

参考负载均衡

#### nginx 

```
https   ssl   443
tcp
status
验证
防盗链
地址重写  rewrite
流量控制
nginx优势
第三方模块    --add-module=
会话保持  -- hash 、cookie、jvm_route
动静分离
```



### ELK

#### ELK原理 

链接：https://pan.baidu.com/s/1aCviHozFmk7xReBNxTZHlA 提取码：57dj 

#### ELK部署

链接：https://pan.baidu.com/s/1W0MsNBi_tUV2cW930o7q-w 提取码：0mdk 

#### ELK组件

Elasticseach、logstash、kibana、beats(filebeat)、kafka

#### ELK数据流

（servers） -- > （PATH）filebeat （topic） -- > kafka -- > (topic)logstash (type)--> elasticsearch （type）—--> kibana(es的type--index) 

#### kafka数据堆积问题

1、单位时间可以被消耗， 这就是kafka的本职

2、单位时间不能被消耗，增加kafka中topic的partation数量， 增加logstash或者kafka集群数量

3、日志汇集存储上

4、更换选型

#### 业务架构选型，为什么选择kafka

```
###### 1、Kafka

Kafka是LinkedIn开源的分布式发布-订阅消息系统，目前归属于Apache顶级项目。Kafka主要特点是基于Pull的模式来处理消息消费，追求高吞吐量，一开始的目的就是用于日志收集和传输。0.8版本开始支持复制，不支持事务，对消息的重复、丢失、错误没有严格要求，适合产生大量数据的互联网服务的数据收集业务。

###### 2、RabbitMQ

RabbitMQ是使用Erlang语言开发的开源消息队列系统，基于AMQP协议来实现。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。AMQP协议更多用在企业系统内对数据一致性、稳定性和可靠性要求很高的场景，对性能和吞吐量的要求还在其次。

###### 3、RocketMQ

RocketMQ是阿里开源的消息中间件，它是纯Java开发，具有高吞吐量、高可用性、适合大规模分布式系统应用的特点。RocketMQ思路起源于Kafka，但并不是Kafka的一个Copy，它对消息的可靠传输及事务性做了优化，目前在阿里集团被广泛应用于交易、充值、流计算、消息推送、日志流式处理、binglog分发等场景。

RabbitMQ比Kafka可靠，Kafka更适合IO高吞吐的处理，一般应用在大数据日志处理或对实时性（少量延迟），可靠性（少量丢数据）要求稍低的场景使用，比如ELK日志收集。
```



各种web环境的部署

LNMP环境搭建、Nginx+mysql+tomcat 部署、Nginx+uwsgi+python+django、Nginx + uwsgi + python +flask、

http://note.youdao.com/noteshare?id=e2b7a71860fc4acebc140982f54586f0



### 公司人员组成

技术部门 --- > 运维 (网络、DBA、IDC运维、系统运维、应用运维、运维开发、运维架构、)、开发（分组）、测试(项目测试)、产品 ( )        技术服务部、项目交付部、运维

![](https://i.loli.net/2019/06/20/5d0b2846a2c8874224.jpg)

人事 

部门助理  

分控部门

运营部门

财务部门

总经理室



### 公司部门分工

网络、DBA、系统运维、应用运维、运维开发、运维架构





### 平时工作中内容

查看监控、处理工单、日常sql提取、报警的处理、服务搭建、项目上线、学习、









