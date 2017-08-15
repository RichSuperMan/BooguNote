# Mysql_优化

------

## 前言

### 1. 为什么别人问你MySQL优化的知识 总是没有底气。

因为你只是回答一些大而化之的调优原则,比如:

 - 建立合理索引(什么样的索引合理？
 - 分表分库(用什么策略分表分库？)
 - 主从分离(用什么中间件？)
 - 并没有从细化到定量的层面去分析。
 - 如qps提高了%N？
 - 有没有减少文件排序？
 - 语句的扫描行数减少了多少？
 - 没有大量的数据供测试,一般在学习环境中,只是手工添加几百上万条数据,数据量小,看不出语句之间的明确区别.

### 2. 如何提高MySQL的性能?

 - 需要优化，则说明效率不够理想。
 - 因此我们首先要做的，不是优化，而是---诊断。
 - 治病的前提，是诊病，找出瓶颈所在。CPU、内存、IO、峰值、单条语句？

## Mysql数据库的优化技术

### 优化细节

 1. 表的设计合理化（符合三范式）
 2. 添加适当索引(index)[四种常用：普通索引、主键索引、唯一索引、全文索引]
 3. 分表技术（水平分割、垂直分割）
 4. 读写分离
 5. 存储过程
 6. 对mysql配置进行优化
 7. mysql服务器硬件升级
 8. 定时的去清楚不需要的数据，定时进行碎片整理（MyISAM）

### 什么样的表才是符合3NF(范式)
表的范式，是首先符合1NF,才能满足2NF,进一步满足3NF

 - 1NF:即表的列具有原子性，不可再分割
 - 2NF:即表中的记录是唯一的，就满足2NF,通常我们设计一个主键来实现
 - 3NF:即表中不要有冗余数据，就是说，表的信息，如果被推导出来，就不应该单独的设计一个字段来存放。
 - 反3NF:但是，没有冗余的数据库未必是最好的数据库，有时为了提高运行效率，就必须降低范式标准，适当保留冗余数据

### 服务状态查询

 - 查看当前数据库的状态，常用的有：
    - 查看系统状态：SHOW STATUS;
    - 查看刚刚执行 SQL 是否有警告信息：SHOW WARNINGS;
    - 查看刚刚执行 SQL 是否有错误信息：SHOW ERRORS;
    - 查看已经连接的所有线程状况：SHOW PROCESSLIST;
    - 查看当前连接数量：SHOW STATUS LIKE 'max_used_connections';
    - 查看变量，在 my.cnf 中配置的变量会在这里显示：SHOW VARIABLES;
    - 查看当前MySQL 中已经记录了多少条慢查询，前提是配置文件中开启慢查询记录了.
        - SHOW STATUS LIKE '%slow_queries%';
    - 查询当前MySQL中查询、更新、删除执行多少条了，可以通过这个来判断系统是侧重于读还是侧重于写，如果是写要考虑使用读写分离。
        - SHOW STATUS LIKE '%Com_select%';
        - SHOW STATUS LIKE '%Com_update%';
        - SHOW STATUS LIKE '%Com_delete%';
    - 显示MySQL服务启动运行了多少时间，如果MySQL服务重启，该时间重新计算，单位秒
        - SHOW STATUS LIKE 'uptime';
    - 显示查询缓存的状态情况
        - SHOW STATUS LIKE 'qcache%';
        - PS.下面的解释，我目前不肯定是对，还要再找下资料：
            - http://dba.stackexchange.com/questions/33811/qcache-free-memory-not-full-yet-i-get-alot-of-qcache-lowmem-prunes
            - https://dev.mysql.com/doc/refman/5.7/en/query-cache-status-and-maintenance.html
            - https://dev.mysql.com/doc/refman/5.7/en/server-status-variables.html
            - http://www.111cn.net/database/110/c0c88da67b9e0c6c8fabfbcd6c733523.htm
        - Qcache_free_blocks，缓存中相邻内存块的个数。数目大说明可能有碎片。如果数目比较大，可以执行： flush query cache; 对缓存中的碎片进行整理，从而得到一个空闲块。
        - Qcache_free_memory，缓存中的空闲内存大小。如果Qcache_free_blocks 比较大，说明碎片严重。如果            Qcache_free_memory 很小，说明缓存不够用了。
        - Qcache_hits，每次查询在缓存中命中时就增大该值。
        - Qcache_inserts，每次查询，如果没有从缓存中找到数据，这里会增大该值
        - Qcache_lowmem_prunes，因内存不足删除缓存次数，缓存出现内存不足并且必须要进行清理, 以便为更多查询提供空间的次数。返个数字最好长时间来看；如果返个数字在不断增长，就表示可能碎片非常严重，或者缓存内存很少。
        - Qcache_not_cached # 没有进行缓存的查询的数量，通常是这些查询未被缓存或其类型不允许被缓存
        - Qcache_queries_in_cache # 在当前缓存的查询（和响应）的数量。
        - Qcache_total_blocks #缓存中块的数量
### 如何定位慢查询？


