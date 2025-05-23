#### map底层原理

map是hash表，由hmap和bmap组合而成

1. hamp（哈希表头部）

```go
type hmap struct {
    count     int            // 当前元素总数
    B         uint8          // bucket 数量为 2^B
    buckets   unsafe.Pointer // 指向一片连续的 bucket 数组
    oldbuckets unsafe.Pointer // 扩容时指向旧的 buckets，支持渐进搬迁
    nevacuate  uintptr        // 扩容搬迁进度指针
    hash0      uint32         // 哈希种子，防止 DOS 攻击
    // … 还有一些标记和计数器 …
}
```

- bucket：底层实际存储元素的bucket（也就是bmap）数组，长度总是2的幂

2. bmap（桶）

```go
type bmap struct {
    tophash [bucketCnt]uint8      // 每个 cell 的高 8 位哈希，用于快速判等 & 区分空/删除状态
    keys     [bucketCnt]keyType   // 连续存放的 key
    values   [bucketCnt]valueType // 紧跟着的 value
    overflow *bmap                 // 如果本 bucket 满了，溢出到下一个 bucket
}
```

- 每个bmap中的bucketCnt恒为8

- tophash： 存储每个key哈希值的高8位；值< minTopHash表示空或已删除的状态
- keys/values：将所有的key集中存放，再集中存放所有value，可以减少结构体内填充（padding）带来的浪费
- overflow：链表式处理冲突，当同一bucket超过8条记录时，分配新的溢出bucket

3. 插入，查找和删除

- 哈希及定位bucket

  ```go
  hash := hashKey(key, h.hash0)
  idx  := hash & (uintptr(1)<<h.B - 1) // 取低 B 位，得到 bucket 下标
  ```

- 在bucket链表中查找
  - 扫描tophash找到可能匹配的cell，再对比完整key
  - 若命中，更新或读取值->结束
  - 否则若有overflow，继续在下一个bucket查找；否则到空槽插入
- 删除
  - 找到对应cell，将tophash[i]标记为deleted，置key/value为零值
  - 不会自动缩容，bucket数量和已分配内存保持不变(为后续插入复用)

4. 动态扩容
   - 当load factor（元素数/bucket总容量）超过阈值（约6.5）,触发扩容；
     - 新建一片大小为2x的buckets
     - 保持旧buckets，在每次map操作时渐进搬迁部分就bucket到新buckets（通过nevacuate指针记录进度）
     - 搬迁完毕后，丢弃旧buckets，将新buckets替换为主储存
5. 随机化与安全
   - GO在运行时为每个map分配一个随机种子hash0,每次哈希计算都会混入这个种子
     - 防止攻击者预测哈希值分布，避免Hash Dos
     - 使得同一程序多次运行的map遍历顺序不同

#### 小结：

1. map是哈希表，有hamp+bmap组合而成
2. 每个bucket固定承载8条，使用链表处理溢出。
3. 删除不收缩桶数组；扩容通过渐进搬迁做到零停顿。
4. 哈希种子随机化保证安全