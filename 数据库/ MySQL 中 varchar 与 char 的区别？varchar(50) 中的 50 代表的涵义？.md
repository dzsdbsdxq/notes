1、varchar 与 char 的区别，char 是一种固定长度的类型，varchar 则是一种可变长度的类型。

- 2、varchar(50) 中 50 的涵义最多存放 50 个字符。varchar(50) 和 (200) 存储 hello 所占空间一样，

  但后者在排序时会消耗更多内存，因为 `ORDER BY col` 采用 fixed_length 计算 col 长度(memory引擎也一样)

  。

  > 所以，实际场景下，选择合适的 varchar 长度还是有必要的。