# Chapter 5 进程及进程管理

## 编程作业

1. sys_spawn实现

    比较简单，使用TaskControlBlock.new创建新的tcb，分配pid，然后建立父进程和子进程tcb之间的关系即可。

2. stride调度算法

    使用优先队列实现就绪队列，在tcb加入就绪队列时更新stride。

## 问答作业

1. p2的stride可能溢出，导致其stride比p1小。

2. stride每次的增量pass小于BigStride / 2，每次调度又会取最小的stride，可以看出STRIDE_MAX和STRIDE_MIN之差不会超过BigStride / 2。

3. Stride的cmp函数的实现见src/manager.rs 13-63。
