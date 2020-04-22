volatile能够保证可见性和有序性，但不能保证原子性

1. 如何保证可见性的：

   - 缓存一致性协议（如MESI）

     > MESI protocol is the Invalidate-based cache coherence protocol,and is one of the most common protocols which support **write-back caches**.
     >
     > Write-back caches can save a log on bandwidth which is generally wasted on a **write-through caches**
     >
     > 
     >
     > **Writing Policy** : When a system writes data to cache,it must at some point write that data to the backing store as well.The time of this write is controlled by so-called writing policy
     >
     > - Write-through : write is done synchronously both to cache and backing store
     > - Write-back: also known as write-behind.Writing is done only to cache.And the write to backing store is postponed until the modified content is about to be replaced by another cache block
     >
     > The letters in the acronym MESI represent for **4 exclusive states** that a cache line can be marked with(encoded using 2 bits)
     >
     > - M 
     >
     >   M for Modified. The cache line is present only in the current cache,and is *dirty* ,which means it has been modified and is different from the value in the main memory.Before permitting any other read,the cache line must be write to main memory and its state changed to **S**
     >
     > - E
     >
     >   E for Exclusive. The cache line is present only in the current cache,and is *clean*,which means it has the same value with that in main memory. It may be changed to **S** state in response to any read request.Alternatively, it may be changed to **M** when writing to it.
     >
     > - S
     >
     >   S for Shared. The cache line may be stored in other caches of the machine , and is *clean*. This cache line may be discarded (changed to **I** state) at any time.
     >
     > - I
     >
     >   I for Invalid. The cache line is invalid or unused.
     >
     >   关于MESI讲的比较好的一篇文章：
     >
     >   <https://www.cnblogs.com/bjlhx/p/10658938.html>
     >
     >   <https://en.wikipedia.org/wiki/MESI_protocol>
     >
     >   <https://en.wikipedia.org/wiki/Bus_snooping>
     >
     >   MESI之间的状态转换是通过Bus Snooping（总线监听）来实现的
     >
     >   Bus Snooping协议分为两种
     >
     >   - Write-Invalidate
     >   - Write-Update
     >
     >   而MESI是属于Write-Invalidate的
     >
     >   There are two kinds of snooping protocols depending on the way to manage a local copy of a write operation:
     >
     >   ### Write-invalidate
     >
     >   When a processor writes on a shared cache block, all the shared copies in the other caches are [invalidated](https://en.wikipedia.org/wiki/Cache_invalidation) through bus snooping. This method ensures that only one copy of a datum can be exclusively read and written by a processor. All the other copies in other caches are invalidated. This is the most commonly used snooping protocol. [MSI](https://en.wikipedia.org/wiki/MSI_protocol), [MESI](https://en.wikipedia.org/wiki/MESI_protocol), [MOSI](https://en.wikipedia.org/wiki/MOSI_protocol), [MOESI](https://en.wikipedia.org/wiki/MOESI_protocol), and [MESIF](https://en.wikipedia.org/wiki/MESIF_protocol) protocols belong to this category.
     >
     >   ### Write-update
     >
     >   When a processor writes on a shared cache block, all the shared copies of the other caches are updated through bus snooping. This method broadcasts a write data to all caches throughout a bus. It incurs larger bus traffic than write-invalidate protocol. That is why this method is uncommon. [Dragon](https://en.wikipedia.org/wiki/Dragon_protocol) and [firefly](https://en.wikipedia.org/wiki/Firefly_(cache_coherence_protocol)) protocols belong to this category.[[3\]](https://en.wikipedia.org/wiki/Bus_snooping#cite_note-3)

   - 锁总线

2. 如何保证有序性：

   禁止指令重排序  

   => 如何禁止指令重排序？  = > 内存屏障

   JVM规范规定了4种内存屏障，分别是

   - LoadLoad
   - LoadStore
   - StoreStore
   - StoreLoad

   在某些CPU上，对于内存屏障，有lfence，sfence，mfence等汇编原语。但有些CPU没有这种专门的原语，所以HotSpot采用了lock锁总线的方式来实现内存屏障，因为lock指令是任何CPU都拥有的指令。对于volatile修饰的变量，会在该变量读写操作前后加上屏障

   > LoadLoad
   >
   > volatile 读操作
   >
   > LoadStore

   > StoreStore
   >
   > volatile 写操作
   >
   > StoreLoad