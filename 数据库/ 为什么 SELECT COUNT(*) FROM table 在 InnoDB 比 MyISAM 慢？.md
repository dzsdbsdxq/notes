对于 `SELECT COUNT(*) FROM table` 语句，在没有 `WHERE` 条件的情况下，InnoDB 比 MyISAM 可能会慢很多，尤其在大表的情况下。因为，InnoDB 是去实时统计结果，会全表扫描；而 MyISAM 内部维持了一个计数器，预存了结果，所以直接返回即可。

详细的原因，胖友可以看看 [《高性能 MySQL 之 Count 统计查询》](https://blog.csdn.net/qq_15037231/article/details/81179383) 博客。