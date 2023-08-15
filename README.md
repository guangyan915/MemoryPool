# MemoryPool

## 三级缓存结构

- thread cache： 线程缓存，用于小于256KB的内存的分配。每个线程独享一个thread cache（线程本地存储），不需要加锁，这也是这个并发线程池高效的地方。
- central cache： 中心缓存，所有线程共享。thread cache按需从central cache中获取对象。central cache在合适时机回收thread cache中的对象，避免一个线程占用太多内存，而其它线程的内存吃紧，达到内存分配在多个线程中更均衡地按需调度的目的。central cache存在竞争，从这里取内存对象时需要加锁，这里用的是桶锁，且只有thread cache没有内存对象时才会找到central cache，所以这里竞争不会很激烈。
- page cache： 页缓存，存储的内存以页为单位存储及分配。当central cache没有内存对象时，从page cache分配出一定数量的page，并切割成定长大小的小块内存，分配给central cache。当一个span的几个跨度页的对象都回收后，page cache会回收central cache满足条件的soan对象，并合并相邻的页，组成更大的页，缓解内存碎片的问题。

