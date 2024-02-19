MySQL5.6 下 Innodb 引擎的主要改进：

1. online DDL
2. memcached NoSQL 接口
3. transportable tablespace（ alter table discard/import tablespace）
4. MySQL 正常关闭时，可以 dump 出 buffer pool 的（ space， page_no），重启时 reload，加快预热速度
5. 索引和表的统计信息持久化到 mysql.innodb_table_stats 和 mysql.innodb_index_stats，可提供稳定的执行计划
6. Compressed row format 支持压缩表

MySQL5.7 下 Innodb 引擎的主要改进：

- 1、修改 varchar 字段长度有时可以使用

  > 这里的“有时”，指的是也有些限制。可见 [《MySQL 5.7 online ddl 的一些改进》](https://yq.aliyun.com/articles/581726) 。

- 2、Buffer pool 支持在线改变大小

- 3、Buffer pool 支持导出部分比例

- 4、支持新建 innodb tablespace，并可以在其中创建多张表

- 5、磁盘临时表采用 innodb 存储，并且存储在 innodb temp tablespace 里面，以前是 MyISAM 存储

- 6、透明表空间压缩功能