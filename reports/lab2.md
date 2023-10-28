# 地址空间

## mmap和munmap实现

首先mmap和munmap需要使用到区间查询操作，最常用的便是查询两个区间是否重叠，使用iset包中的IntervalMap可以很方便的执行区间查询操作。

