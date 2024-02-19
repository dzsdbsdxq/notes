> 题目来源：腾讯
>
> 频次：高频

## 答案：树枝

这道题需要从两个维度来回答

1. map的实现原理

   1. go map是基于hash table（哈希表）来实现的，冲突的解决采用拉链法

2. map的底层结构

   1. hmap（哈希表）：每个hmap内含有多个bmap（buckets（桶）、lodbuckets（旧桶）、overflow（溢出桶））可以这样理解，每个哈希表都是由多个桶组成的

      ~~~ go
      type hmap struct {
          count     int    //元素的个数
          flags     uint8  //状态标志
          B         uint8  //可以最多容纳 6.5 * 2 ^ B 个元素，6.5为装载因子
          noverflow uint16 //溢出的个数
          hash0     uint32 //哈希种子
      
          buckets    unsafe.Pointer //指向一个桶数组
          oldbuckets unsafe.Pointer //指向一个旧桶数组，用于扩容
          nevacuate  uintptr        //搬迁进度，小于nevacuate的已经搬迁
          overflow *[2]*[]*bmap     //指向溢出桶的指针
      }
      ~~~

      - buckets：一个指针，指向一个bmap数组、存储多个桶。
      - oldbuckets： 是一个指针，指向一个bmap数组，存储多个旧桶，用于扩容。 
      - overflow：overflow是一个指针，指向一个元素个数为2的数组，数组的类型是一个指针，指向一个slice，slice的元素是桶(bmap)的地址，这些桶都是溢出桶。为什么有两个？因为Go map在哈希冲突过多时，会发生扩容操作。[0]表示当前使用的溢出桶集合，[1]是在发生扩容时，保存了旧的溢出桶集合。overflow存在的意义在于防止溢出桶被gc。

   2. bmap（哈希桶）： bmap是一个隶属于hmap的结构体，一个桶（bmap）可以存储8个键值对。如果有第9个键值对被分配到该桶，那就需要再创建一个桶，通过overflow指针将两个桶连接起来。在hmap中，多个bmap桶通过overflow指针相连，组成一个链表。 

      ~~~ go
      type bmap struct {
          //元素hash值的高8位代表它在桶中的位置，如果tophash[0] < minTopHash，表示这个桶的搬迁状态
          tophash [bucketCnt]uint8
          //接下来是8个key、8个value，但是我们不能直接看到；为了优化对齐，go采用了key放在一起，value放在一起的存储方式，
          keys     [8]keytype   //key单独存储
      	values   [8]valuetype //value单独存储
      	pad      uintptr
      	overflow uintptr	  //指向溢出桶的指针
      }
      ~~~

