## 简介

Redis 是速度非常快的非关系型 NoSQL 内存键值数据库。



## 1.目的&特性

**缓存目的**

高性能：操作缓存就是直接操作内存，所以速度相当快。

高并发：直接操作缓存能够承受的请求是远远大于直接访问数据库的，所以我们可以考虑把数据库中的部分数据转移到缓存中去，这样用户的一部分请求会直接到缓存这里而不用经过数据库。缓存是走内存的，内存天然就支撑高并发。

**缓存特征**

命中率：当某个请求能够通过访问缓存而得到响应时，称为缓存命中。缓存命中率越高，缓存的利用率也就越高。

最大空间：缓存通常位于内存中，内存的空间通常比磁盘空间小的多，缓存的最大空间不可能非常大。当缓存存放的数据量超过最大空间时，就需要淘汰部分数据来存放新到达的数据。

淘汰策略：FIFO 需要经常访问最新的数据，LRU 保证内存中的数据都是经常被访问的热点数据（从而保证缓存命中率），LFU 优先淘汰一段时间内使用次数最少的数据。

键的过期时间：Redis 可以为每个键设置过期时间，当键过期时，会自动删除该键。对于散列表这种容器，只能为整个键设置过期时间（整个散列表），而不能为键里面的单个元素设置过期时间。

命名空间：Redis 没有关系型数据库中的表这一概念来将同种类型的数据存放在一起，而是使用命名空间的方式来实现这一功能。键名的前面部分存储命名空间，后面部分的内容存储 ID，通常使用 : 来进行分隔。



## 2.数据类型&数据结构

#### 数据类型

键的类型只能为字符串，值支持五种数据类型。

字符串、列表、集合、散列表、有序集合

STRING、LIST、SET、HASH、ZSET

STRING 可以实现常规 key-value 缓存应用和计数，LIST 可以实现分页查询和消息队列，SET 可以实现共同好友和去重，HASH 可以实现存储对象，ZSET 可以实现排行榜。

Redis 里存的是字节数组 byte[]，无论字符串、数字、对象、图片、声音、视频、还是文件，只要能变成 byte[]，都可以存到 Redis 里。

#### ZSET 

增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列。

value 值不可以重复，先按 score 排序，再按 value 排序。

使用 ZIPLIST 和 SKIPLIST 两种方式编码。

**ZIPLIST** 

ZIPLIST 所保存的元素数量超过服务器属性 server.zset_max_ziplist_entries 的值（默认值为 128）或者新添加元素的 member 的长度大于服务器属性 server.zset_max_ziplist_value 的值（默认值为 64），ZIPLIST 将会转换为 SKIPLIST。 

ZIPLIST 查找某个给定元素的复杂度为 O(N) 。

**SKIPLIST**

同时使用字典和跳表两个数据结构来保存有序集元素。

使用字典结构，并将 member 作为键，score 作为值，可以在 O(1) 的时间内：检查给定 member 是否存在于有序集（被很多底层函数使用）；取出 member 对应的 score 值（实现 ZSCORE 命令）。

通过使用跳表，可以让有序集支持：在 O(logN) 期望时间、O(N) 最坏时间内根据 score 对 member 进行定位（被很多底层函数使用）；范围性查找和处理操作，这是（高效地）实现 ZRANGE、ZRANK 和 ZINTERSTORE 等命令的关键。

通过同时使用字典和跳跃表，有序集可以高效地实现按成员查找和按顺序查找两种操作。

#### 数据结构 

#### 字典 dict

dictht 是一个散列表结构，使用拉链法保存哈希冲突。

dict 中包含两个哈希表 dictht，这是为了方便进行 rehash 操作。在扩容时，将其中一个 dictht 上的键值对 rehash 到另一个 dictht 上面，完成之后释放空间并交换两个 dictht 的角色。

**渐进式 rehash**

rehash 操作不是一次性完成，而是采用渐进方式，为了避免一次性执行过多的 rehash 操作给服务器带来过大的负担。

通过记录 dict 的 rehashidx 完成，它从 0 开始，然后每执行一次 rehash 都会递增。

在 rehash 期间，每次对字典执行添加、删除、查找或者更新操作时，都会执行一次渐进式 rehash。

采用渐进式 rehash 会导致字典中的数据分散在两个 dictht 上，因此对字典的查找操作也需要到对应的 dictht 去执行。

#### 跳跃表

是有序集合的底层实现之一（还有散列表）。

跳表的本质是同时维护了多个链表，并且链表是分层的。最低层的链表维护了跳表内所有的元素，每上面一层链表都是下面一层的子集。

跳表内的所有链表的元素都是排序的

在查找时，从上层指针开始查找，找到对应的区间之后再到下一层去查找。搜索是跳跃式的。

查找、插入或删除的时间复杂度 O(logN)。

**与红黑树等平衡树相比的优点**

- 对平衡树的插入和删除往往很可能导致平衡树进行一次全局的调整。而对跳表的插入和删除只需要对整个数据结构的局部进行操作即可。
- 在高并发的情况下，你会需要一个全局锁来保证整个平衡树的线程安全。而对于跳表，你只需要部分锁即可。
- 更容易实现。



## 3.使用场景

**计数器**

对 String 进行自增自减运算，从而实现计数器功能。

Redis 这种内存型数据库的读写性能非常高，很适合存储频繁读写的计数量。

**缓存**

将热点数据放到内存中，设置内存的最大使用量以及淘汰策略来保证缓存的命中率。

**查找表**

DNS 记录就很适合使用 Redis 进行存储。查找表和缓存类似，也是利用了 Redis 快速的查找特性。但是查找表的内容不能失效，而缓存的内容可以失效，因为缓存不作为可靠的数据来源。

**消息队列**

List 是一个双向链表，可以通过 lpush 和 rpop 写入和读取消息。

不过最好使用 Kafka、RabbitMQ 等消息中间件。

**会话缓存**

可以使用 Redis 来统一存储多台应用服务器的会话信息。

当应用服务器不再存储用户的会话信息，也就不再具有状态，一个用户可以请求任意一个应用服务器，从而更容易实现高可用性以及可伸缩性。

**分布式锁**

可以使用 Redis 自带的 SETNX 命令实现分布式锁，除此之外，还可以使用官方提供的 RedLock 分布式锁实现。

#### 缓存位置

**浏览器**

当 HTTP 响应允许进行缓存时，浏览器会将 HTML、CSS、JavaScript、图片等静态资源进行缓存。

**ISP**

网络服务提供商 ISP 是网络访问的第一跳，通过将数据缓存在 ISP 中能够大大提高用户的访问速度。

**内容分发网络 CDN**

CDN 是一种互连的网络系统，它利用更靠近用户的服务器从而更快更可靠地将 HTML、CSS、JavaScript、音乐、图片、视频等静态资源分发给用户。

加速用户获取数据，命中 CDN 时不需要访问后端服务器，提高访问速度。

CDN 的关键技术主要有内容存储和分发技术。

**优点**

- 更快地将数据分发给用户。
- 通过部署多台服务器，从而提高系统整体的带宽性能。
- 多台服务器可以看成是一种冗余机制，从而具有高可用性。

**反向代理**

反向代理位于服务器之前，请求与响应都需要经过反向代理。通过将数据缓存在反向代理，在用户请求反向代理时就可以直接使用缓存进行响应。

**本地缓存**

使用 Guava Cache 将数据缓存在服务器本地内存中，服务器代码可以直接读取本地内存中的缓存，速度非常快。

**分布式缓存**

使用 Redis、Memcache 等分布式缓存将数据缓存在分布式缓存系统中。

分布式缓存单独部署，可以根据需求分配硬件资源。而且服务器集群都可以访问分布式缓存，而本地缓存需要在服务器集群之间进行同步，实现难度和性能开销上都非常大。

**数据库缓存**

MySQL 等数据库管理系统具有自己的查询缓存机制来提高查询效率。

**Java 内部的缓存**

Java 为了优化空间，提高字符串、基本数据类型包装类的创建效率，设计了字符串常量池及 Byte、Short、Character、Integer、Long、Boolean 这六种包装类缓冲池。

**CPU 多级缓存**

CPU 为了解决运算速度与主存 IO 速度不匹配的问题，引入了多级缓存结构，同时使用 MESI 等缓存一致性协议来解决多核 CPU 缓存数据一致性的问题。

**多 CPU 的嗅探机制**

每个处理器通过嗅探在总线上传播的数据来检查自己的缓存值是不是过期了，如果处理器发现自己缓存行对应的内存地址呗修改，就会将当前处理器的缓存行设置无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据库读到处理器缓存中。

**多处理器下的缓存一致性协议 MESI**

保证了每个缓存中使用的共享变量的副本是一致的。

如果一个变量在多个 CPU 中都存在缓存（多线程环境中），那么就可能存在缓存不一致的问题。

在早期的 CPU 当中，是通过在总线上加 LOCK# 锁的形式来解决缓存不一致的问题。因为 CPU 和其他部件进行通信都是通过总线来进行的，如果对总线加 LOCK# 锁的话，也就是说阻塞了其他 CPU 对其他部件访问（如内存），从而使得只能有一个 CPU 能使用这个变量的内存。由于在锁住总线期间，其他 CPU 无法访问内存，导致效率低下。

MESI 的核心思想：当 CPU 写数据时，如果发现操作的变量是共享变量，即在其他 CPU 中也存在该变量的副本，会发出信号通知其他 CPU 将该变量的缓存行置为无效状态，因此当其他 CPU 需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。



## 4.Redis 与 Memcached

两者都是非关系型内存键值数据库，主要有以下不同：

- 数据类型：Redis 有五种数据类型，而 Memcached 仅支持字符串类型。
- 持久化：Redis 支持 RDB 快照和 AOF 日志进行持久化，而 Memcached 不支持持久化。
- 分布式：Redis Cluster 实现了分布式的支持。而 Memcached 不支持分布式，只能通过在客户端使用一致性哈希来实现分布式存储，这种方式在存储和查询时都需要先在客户端计算一次数据所在的节点。
- 内存管理机制：Redis 中并不是所有数据都一直存储在内存中，可以将一些很久没用的 value 交换到磁盘。而 Memcached 的数据则会一直在内存中，将内存分割成特定长度的块来存储数据，以完全解决内存碎片的问题，但是这种方式会使得内存的利用率不高。
- 核：Redis 只使用单核，Memcached 可以使用多核。平均每一个核上 Redis 在存储小数据时比 Memcached 性能更高。存储大数据时 Memcached 高。
- 线程：Redis 单线程，Memcached 多线程。Memcached 是多线程非阻塞 IO 复用的网络模型。



## 5.数据淘汰策略

可以设置内存最大使用量，当内存使用量超出时，会施行数据淘汰策略。

Redis 的过期策略：对于设置了过期时间的数据定期删除 + 惰性删除。

**定期删除**

Redis 默认是每隔 100ms 就随机抽取一些设置了过期时间的 key，检查其是否过期，如果过期就删除。可能会导致很多过期 key 到了时间并没有被删除掉。

**惰性删除**

获取 key 的时候，如果此时 key 已经过期，就删除，不会返回任何东西。

**内存淘汰机制**

定期删除漏掉了很多过期 key，然后你也没及时去查，也就没走惰性删除，使大量过期 key 堆积在内存里，导致 Redis 内存块耗尽了，此时会施行内存淘汰机制。

- volatile-lru：从已设置过期时间的数据集中挑选最近最少使用的数据淘汰。
- volatile-ttl：从已设置过期时间的数据集中挑选将要过期的数据淘汰。
- volatile-random：从已设置过期时间的数据集中任意选择数据淘汰。
- allkeys-lru：从所有数据集中挑选最近最少使用的数据淘汰。
- allkeys-random：从所有数据集中任意选择数据进行淘汰。
- noeviction：禁止驱逐数据。

作为内存数据库，出于对性能和内存消耗的考虑，Redis 的淘汰算法实际实现上并非针对所有 key，而是抽样一小部分并且从中选出被淘汰的 key。

使用 Redis 缓存数据时，为了提高缓存命中率，需要保证缓存数据都是热点数据。可以将内存最大使用量设置为热点数据占用的内存量，然后启用 allkeys-lru 淘汰策略。

Redis 4.0 引入了 volatile-lfu 和 allkeys-lfu 淘汰策略，LFU 策略通过统计访问频率，将访问频率最少的键值对淘汰。

**热点 key**

大量请求访问的 key。

发现：

- 凭借业务经验预估。
- 在客户端或代理层进行收集。
- Redis 自带命令 monitor 或 redis-cli 的热点 key 发现功能 –hotkeys。
- 还有自己抓包评估（客户端使用 TCP 协议与服务端进行交互，通信协议采用的是 RESP）。

然后解决：

- 利用二级缓存，在你发现热 key 以后，把热 key 缓存到系统的 JVM 中。针对这种热 key 请求，会直接从 JVM 中取，而不会走到 Redis 层。分散了请求。
- 备份多台 Redis 服务器分摊热 key 请求

- 在项目运行过程中，自动发现热 key，然后程序自动处理：监控热 key，通知系统做处理。



## 6.持久化

Redis 是内存型数据库，为了保证数据在断电后不会丢失，需要将内存中的数据持久化到硬盘上。

#### RDB 快照

创建快照来获得存储在内存里面的数据在某个时间点上的副本，再存放到硬盘上。

默认方法。

可以将快照复制到其它服务器从而创建具有相同数据的服务器副本。还可以将快照留在原地以便重启服务器的时候使用。

如果系统发生故障，将会丢失最后一次创建快照之后的数据。

如果数据量很大，保存快照的时间会很长。

实际操作过程是 fork 一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。

**优点**

恢复速度快。紧凑。可以将数据集还原到不同的版本。
RDB 会生成多个数据文件，每个数据文件都代表了某一个时刻中 Redis 的数据，非常适合做冷备，可以将这种完整的数据文件发送到一些远程的安全存储上去。
RDB 对 Redis 对外提供的读写服务，影响非常小，可以让 Redis 保持高性能，因为 Redis 主进程只需要 fork 一个子进程，让子进程执行磁盘 IO 操作来进行 RDB 持久化即可。

**缺点**

在数据集比较庞大时，fork() 可能会非常耗时。
在 Redis 故障时，丢失数据较多。
RDB 每次在 fork 子进程来执行 RDB 快照数据文件生成的时候，如果数据文件特别大，可能会导致对客户端提供的服务暂停数毫秒，或者甚至数秒。

#### AOF 文件

将写命令添加到 AOF 文件的末尾。

在 Redis 重启的时候，可以通过回放 AOF 日志中的写入指令来重新构建整个数据集。

使用 AOF 持久化需要设置同步选项，从而确保写命令同步到磁盘文件上的时机。这是因为对文件进行写入并不会马上将内容同步到磁盘上，而是先存储到缓冲区，然后由操作系统决定什么时候同步到磁盘。有以下同步选项：

- always：选项会严重减低服务器的性能。
- everysec：选项比较合适，可以保证系统崩溃时只会丢失一秒左右的数据，并且 Redis 每秒执行一次同步对服务器性能几乎没有任何影响。
- no：选项并不能给服务器性能带来多大的提升，而且也会增加系统崩溃时数据丢失的数量。

**AOF 重写**

随着服务器写请求的增多，AOF 文件会越来越大。Redis 提供了一种将 AOF 重写的特性，能够去除 AOF 文件中的冗余写命令。

AOF 重写后，新的 AOF 文件和原有的 AOF 文件所保存的数据库状态一样，但体积更小。该功能是通过读取数据库中的键值对来实现的，程序无须对现有 AOF 文件进行任何读入、分析或者写入操作。

过程：

1. 在执行 BGREWRITEAOF 命令时，Redis 服务器会维护一个 AOF 重写缓冲区，该缓冲区会在子进程创建新 AOF 文件期间，记录服务器执行的所有写命令。

2. 当子进程完成创建新 AOF 文件的工作之后，服务器会将重写缓冲区中的所有内容追加到新 AOF 文件的末尾，使得新旧两个 AOF 文件所保存的数据库状态一致。

3. 最后，服务器用新的 AOF 文件替换旧的 AOF 文件。即使重写过程中发生停机，现有的 AOF 文件也不会丢失。

**优点**

AOF 的默认同步选项为 everysec，兼顾数据和写入性能。Redis 每秒同步一次 AOF 文件，Redis 性能几乎没受到任何影响。而且这样即使出现系统崩溃，用户最多只会丢失一秒之内产生的数据。当硬盘忙于执行写入操作的时候，Redis 还会优雅的放慢自己的速度以便适应硬盘的最大写入速度。
AOF 日志文件以 append-only 模式写入，所以没有任何磁盘寻址的开销，写入性能非常高，而且文件不容易破损，即使文件尾部破损，也很容易修复。
AOF 日志文件即使过大的时候，出现后台重写操作，也不会影响客户端的读写。因为在 rewrite log 的时候，会对其中的指令进行压缩，创建出一份需要恢复数据的最小日志出来。在创建新日志文件的时候，老的日志文件还是照常写入。当新的 merge 后的日志文件 ready 的时候，再交换新老日志文件即可。
AOF 日志文件的命令通过非常可读的方式进行记录，这个特性非常适合做灾难性的误删除的紧急恢复。
append only 对 AOF 文件的写入不需要进行 seek，即使日志因为某些原因而包含了未写入完整的命令，redis-check-aof 工具也可以轻易地修复这种问题。对文件进行分析也很轻松，导出 AOF 文件也非常简单。实时性更好。    

**缺点**

AOF 文件的体积通常要大于 RDB 文件的体积。AOF 的速度可能会慢于 RDB。在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间。
支持的写 QPS 会比 RDB 支持的写 QPS 低，因为 AOF 一般会配置成每秒 fsync 一次日志文件。
恢复时容易出 bug 且速度慢。AOF 为了避免 rewrite 过程导致的 bug，因此每次 rewrite 并不是基于旧的指令日志进行 merge 的，而是基于当时内存中的数据进行指令的重新构建，这样健壮性会好很多。

Redis 4.0 开始支持 RDB 和 AOF 的混合持久化。生产环境其实更多是二者结合使用的。
如果把混合持久化打开，AOF 重写的时候就直接把 RDB 的内容写到 AOF 文件开头。这样做的好处是可以结合 RDB 和 AOF 的优点, 快速加载同时避免丢失过多的数据。当然缺点也是有的，AOF 里面的 RDB 部分是压缩格式不再是 AOF 格式，可读性较差。
如果同时使用 RDB 和 AOF 两种持久化机制，那么在 Redis 重启的时候，会使用 AOF 来重新构建数据，因为 AOF 中的数据更加完整。

可以将这些数据备份到云上，磁盘上的数据丢了从云服务器上恢复。



## 7.事务&事件

#### 事务

一个事务包含了多个命令，服务器在执行事务期间，不会改去执行其它客户端的命令请求。

**流水线**

多个命令被一次性发送给服务器。

可以减少客户端与服务器之间的网络通信次数从而提升性能。

将多个命令请求打包，按顺序地执行。

不要在多个命令中穿插着查询。

**实现方式**

Redis 最简单的事务实现方式是使用 MULTI 和 EXEC 命令将事务操作包围起来。

MULTI 开启事务，EXEC 执行事务，discard 放弃事务。

watch 监视：事务开始执行前 key 被其它命令操作了则会取消事务。

**不支持回滚**

Redis 同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚。

因为Redis 认为，失败都是由命令使用不当造成，这样做是为了保持内部实现简单快速，回滚并不能解决所有问题。

#### 事件

Redis 服务器是一个事件驱动程序。

**文件事件**

服务器通过套接字与客户端或者其它服务器进行通信，文件事件就是对套接字操作的抽象。

Redis 基于 Reactor 模式开发了自己的网络事件处理器，使用 I/O 多路复用程序来同时监听多个套接字，并将到达的事件传送给文件事件分派器，分派器会根据套接字产生的事件类型调用相应的事件处理器。

**时间事件**

服务器有一些操作需要在给定的时间点执行，时间事件是对这类定时操作的抽象。

时间事件又分为定时事件和周期性事件。

Redis 将所有时间事件都放在一个无序链表中，通过遍历整个链表查找出已到达的时间事件，并调用相应的事件处理器。

**事件的调度与执行**

服务器需要不断监听文件事件的套接字才能得到待处理的文件事件，但是不能一直监听，否则时间事件无法在规定的时间内执行，因此监听时间应该根据距离现在最近的时间事件来决定。

事件调度与执行由 aeProcessEvents 函数负责。



## 8.复制

通过使用 slave of host port 命令来让一个服务器成为另一个服务器的从服务器。

一个从服务器只能有一个主服务器，并且不支持主主复制。

拓展读性能。

#### 连接过程

1. 主服务器创建快照文件，发送给从服务器，并在发送期间使用缓冲区记录执行的写命令。快照文件发送完毕之后，开始向从服务器发送存储在缓冲区中的写命令。
2. 从服务器丢弃所有旧数据，载入主服务器发来的快照文件，之后从服务器开始接受主服务器发来的写命令。
3. 主服务器每执行一次写命令，就向从服务器发送相同的写命令。

随着负载不断上升，创建一个中间层来分担主服务器的复制工作，从而形成主从链。

#### 主从架构 master-slave

一主多从，实现读写分离，主节点负责写，并且将数据复制到其它的 slave 节点，从节点负责读。可以很轻松实现水平扩容，支撑读高并发。Redis 实现高并发主要依靠主从架构。
Redis replication -> 主从架构 -> 读写分离 -> 水平扩容支撑读高并发。
Redis 采用异步方式复制数据到 slave 节点，不过 Redis 2.8 开始，slave node 会周期性地确认自己每次复制的数据量。
slave node 在做复制的时候，不会 block master node 的正常工作，也不会 block 对自己的查询操作，它会用旧的数据集来提供服务；但是复制完成的时候，需要删除旧数据集，加载新数据集，这个时候就会暂停对外服务了。
如果采用了主从架构，那么必须开启 master node 的持久化，不建议用 slave node 作为 master node 的数据热备，因为那样的话，如果你关掉 master 的持久化，可能在 master 宕机重启的时候数据是空的，然后可能一经过复制，slave node 的数据也丢了。
需要备份，万一本地的所有文件丢失了，从备份中挑选一份 RDB 去恢复 master，这样才能确保启动的时候，是有数据的，即使采用了后续讲解的高可用机制，slave node 可以自动接管 master node，但也可能 sentinel 还没检测到 master failure，master node 就自动重启了，还是可能导致上面所有的 slave node 数据被清空。

#### 核心原理

当启动一个 slave node 的时候，它会发送一个 PSYNC 命令给 master node。
slave node 初次连接到 master node，那么会触发一次 full resynchronization 全量复制。
此时 master 会启动一个后台线程，开始生成一份 RDB 快照文件，同时还会将从客户端 client 新收到的所有写命令缓存在内存中。
RDB 文件生成完毕后，master 会将这个 RDB 发送给 slave，slave 会先写入本地磁盘，然后再从本地磁盘加载到内存中，接着 master 会将内存中缓存的写命令发送到 slave，slave 也会同步这些数据。
slave node 如果跟 master node 有网络故障，断开了连接，会自动重连，连接之后 master node 仅会复制给 slave 部分缺少的数据。

主从复制的断点续传：
从 Redis 2.8 开始支持，如果主从复制过程中，网络连接断掉了，那么可以接着上次复制的地方，继续复制下去，而不是从头开始复制一份。
master node 会在内存中维护一个 backlog，master 和 slave 都会保存一个 replica offset 还有一个 master run id，offset 就是保存在 backlog 中的。如果 master 和 slave 网络连接断掉了，slave 会让 master 从上次 replica offset 开始继续复制，如果没有找到对应的 offset，那么就会执行一次 resynchronization。
如果根据 host+ip 定位 master node，是不靠谱的，如果 master node 重启或者数据出现了变化，那么 slave node 应该根据不同的 run id 区分。

复制的前期过程：
slave node 启动时，会在自己本地保存 master node 的信息，包括 master node 的 host 和 ip。
slave node 内部有个定时任务，每秒检查是否有新的 master node 要连接和复制，如果发现，就跟 master node 建立 socket 网络连接。
然后 slave node 发送 ping 命令给 master node。如果 master 设置了 requirepass，那么 slave node 必须发送 masterauth 的口令过去进行认证。

无磁盘化复制：master 在内存中直接创建 RDB，然后发送给 slave，不会在自己本地落地磁盘了。只需要在配置文件中开启 repl-diskless-sync yes 即可。
过期 key 处理：slave 不会过期 key，只会等待 master 过期 key。如果 master 过期了一个 key，或者通过 LRU 淘汰了一个 key，那么会模拟一条 del 命令发送给 slave。
heartbeat：主从节点互相都会发送 heartbeat 信息。master 默认每隔 10 秒发送一次 heartbeat，slave node 每隔 1 秒发送一个 heartbeat。保持通信。

正常复制为全量复制，断点续传为增量复制。

复制会带来数据一致性问题。
通过主从架构的切换和哨兵实现高可用（加上持久化）。



## 9.分片

分片是将数据划分为多个部分的方法，可以将数据存储到多台机器里面，这种方法在解决某些问题时可以获得线性级别的性能提升。

拓展写性能。

同 MySQL 分库分表。



## 10.哨兵 Sentinel

可以监听集群中的服务器，并在主服务器进入下线状态时，自动从从服务器中选举出新的主服务器。

功能
集群监控：负责监控 Redis master 和 slave 进程是否正常工作。
消息通知：如果某个 Redis 实例有故障，那么哨兵负责发送消息作为报警通知给管理员。
故障转移：如果 master node 挂掉了，会自动转移到 slave node 上。
配置中心：如果故障转移发生了，通知 client 客户端新的 master 地址。

哨兵用于实现 Redis 集群的高可用，本身也是分布式的，作为一个哨兵集群去运行，互相协同工作。
故障转移时，判断一个 master node 是否宕机了，需要大部分的哨兵都同意才行，涉及到了分布式选举的问题。
即使部分哨兵节点挂掉了，哨兵集群还是能正常工作的。

核心知识：
哨兵至少需要 3 个实例，来保证自己的健壮性。
哨兵 + Redis 主从的部署架构，是不保证数据零丢失的，只能保证 Redis 集群的高可用性。
对于哨兵 + Redis 主从这种复杂的部署架构，尽量在测试环境和生产环境，都进行充足的测试和演练。

哨兵主备切换的数据丢失问题
异步复制导致的数据丢失：master->slave 的复制是异步的，所以可能有部分数据还没复制到 slave，master 就宕机了，此时这部分数据就丢失了。
解决办法：有了 min-slaves-max-lag 这个配置，就可以确保说，一旦 slave 复制数据和 ack 延时太长，就认为可能 master 宕机后损失的数据太多了，那么就拒绝写请求，这样可以把 master 宕机时由于部分数据未同步到 slave 导致的数据丢失降低的可控范围内。

脑裂导致的数据丢失：脑裂就是某个 master 所在机器突然脱离了正常的网络，跟其他 slave 机器不能连接，但是实际上 master 还运行着，此时哨兵可能就会认为 master 宕机了，然后开启选举，将其他 slave 切换成了 master。这个时候，集群里就会有两个 master 。
此时虽然某个 slave 被切换成了 master，但是可能 client 还没来得及切换到新的 master，还继续向旧 master 写数据。因此旧 master 再次恢复的时候，会被作为一个 slave 挂到新的 master 上去，自己的数据会清空，重新从新的 master 复制数据。而新的 master 并没有后来 client 写入的数据，因此，这部分数据也就丢失了。
解决办法：min-slaves-to-write 1 和 min-slaves-max-lag 10，如果一个 master 出现了脑裂，跟其他 slave 丢了连接，那么上面两个配置可以确保说，如果不能继续给指定数量的 slave 发送数据，而且 slave 超过 10 秒没有给自己 ack 消息，那么就直接拒绝客户端的写请求。因此在脑裂场景下，最多就丢失 10 秒的数据。

sdown 是主观宕机，就一个哨兵如果自己觉得一个 master 宕机了，那么就是主观宕机。sdown 达成的条件很简单，如果一个哨兵 ping 一个 master，超过了 is-master-down-after-milliseconds 指定的毫秒数之后，就主观认为 master 宕机了。
odown 是客观宕机，如果 quorum 数量的哨兵都觉得一个 master 宕机了，那么就是客观宕机。如果一个哨兵在指定时间内，收到了 quorum 数量的其它哨兵也认为那个 master 是 sdown 的，那么就认为是 odown 了。
主观宕机和客观宕机有转换机制。
哨兵集群的自动发现机制（哨兵互相之间的发现）：通过 Redis 的 pub/sub 系统实现的，每个哨兵都会往 __sentinel__:hello 这个 channel 里发送一个消息，这时候所有其他哨兵都可以消费到这个消息，并感知到其他的哨兵的存在。每个哨兵还会跟其他哨兵交换对 master 的监控配置，互相进行监控配置的同步。
slave 配置的自动纠正：哨兵会负责自动纠正 slave 的一些配置。
slave->master 选举算法：如果一个 master 被认为 odown 了，而且 majority 数量的哨兵都允许主备切换，那么某个哨兵就会执行主备切换操作。需要选举一个 slave 。
quorum 和 majority：每次一个哨兵要做主备切换，首先需要 quorum 数量的哨兵认为 odown，然后选举出一个哨兵来做切换，这个哨兵还需要得到 majority 哨兵的授权，才能正式执行切换。
configuration epoch：执行切换的那个哨兵，会从要切换到的新 master（salve->master）那里得到一个 configuration epoch，这就是一个 version 号，每次切换的 version 号都必须是唯一的。如果第一个选举出的哨兵切换失败了，那么其他哨兵，会等待 failover-timeout 时间，然后接替继续执行切换。
configuration 传播：哨兵完成切换之后，会在自己本地更新生成最新的 master 配置，然后同步给其他的哨兵，就是通过之前说的 pub/sub 消息机制。其他的哨兵都是根据版本号的大小来更新自己的 master 配置的。



## 11.Redis Cluster

主要是针对海量数据 + 高并发 + 高可用的场景，Redis Cluster 支撑 N 个 Redis master node，每个 master node 都可以挂载多个 slave node，可以横向扩容更多的 master 节点。
自动将数据进行分片，每个 master 上放一部分数据。提供内置的高可用支持，部分 master 不可用时，还是可以继续工作的。
在 Redis Cluster 架构下，每个 Redis 要放开两个端口号：a & a + 10000，a + 10000 端口号是用来进行节点间通信的，就是 cluster bus 的通信，用来进行故障检测、配置更新、故障转移授权。每个节点每隔一段时间都会往另外几个节点发送 ping 消息，同时其它几个节点接收到 ping 之后返回 pong。
cluster bus 用了另外一种二进制的协议，Gossip 协议，用于节点间进行高效的数据交换，占用更少的网络带宽和处理时间。

节点间的内部通信机制：
集群元数据的维护有两种方式：集中式和 Gossip 协议。Redis Cluster 节点间采用 Gossip 协议进行通信。
集中式是将集群元数据（节点信息、故障等等）几种存储在某个节点上。集中式元数据集中存储的一个典型代表，就是大数据领域的 Storm。它是分布式的大数据实时计算引擎，是集中式的元数据存储的结构，底层基于分布式协调的中间件 ZooKeeper 对所有元数据进行存储维护。
Gossip 协议是 Redis 维护集群元数据采用的方式，所有节点都持有一份元数据，不同的节点如果出现了元数据的变更，就不断将元数据发送给其它的节点，让其它节点也进行元数据的变更。
集中式的好处在于，元数据的读取和更新，时效性非常好，一旦元数据出现了变更，就立即更新到集中式的存储中，其它节点读取的时候就可以感知到；不好在于，所有的元数据的更新压力全部集中在一个地方，可能会导致元数据的存储有压力。
Gossip 好处在于，元数据的更新比较分散，不是集中在一个地方，更新请求会陆陆续续打到所有节点上去更新，降低了压力；不好在于，元数据的更新有延时，可能导致集群中的一些操作会有一些滞后。

分布式寻址算法：hash 算法（大量缓存重建）、一致性 hash 算法（自动缓存迁移）+ 虚拟节点（自动负载均衡）、Redis Cluster 的 hash slot 算法。
hash slot 算法：Redis Cluster 中每个 master 都会持有部分 slot。hash slot 让 node 的增加和移除很简单，增加一个 master，就将其他 master 的 hash slot 移动部分过去，减少一个 master，就将它的 hash slot 移动到其他 master 上去。移动 hash slot 的成本是非常低的。任何一台机器宕机，另外两个节点，不影响的。因为 key 找的是 hash slot，不是机器。

Redis Cluster 的高可用：
主备切换，和哨兵类似。直接集成了 replication 和 sentinel 的功能。
判断节点宕机：主观宕机 pfail，客观宕机 fail，sdown，odown。在 cluster-node-timeout 内，某个节点一直没有返回 pong，那么就被认为 pfail。如果一个节点认为某个节点 pfail 了，那么会在 gossip ping 消息中，ping 给其他节点，如果超过半数的节点都认为 pfail 了，那么就会变成 fail。
从节点过滤：检查每个 slave node 与 master node 断开连接的时间，如果超过了 cluster-node-timeout * cluster-slave-validity-factor，那么就没有资格切换成 master。
从节点选举：每个从节点，都根据自己对 master 复制数据的 offset，来设置一个选举时间，offset 越大（复制数据越多）的从节点，选举时间越靠前，优先进行选举。所有的 master node 开始 slave 选举投票，给要进行选举的 slave 进行投票，如果大部分 master node（N/2+1）都投票给了某个从节点，那么选举通过，那个从节点可以切换成 master。从节点执行主备切换，从节点切换为主节点。

客户端只要和集群中的一个节点建立链接后，就可以获取到整个集群的所有节点信息。此外还会获取所有哈希槽和节点的对应关系信息，这些信息数据都会在客户端缓存起来。客户端需要实现一个和集群端一样的哈希函数。
集群已经发生了变化，客户端的缓存还没来得及更新，会出现拿到一个 key 向对应的节点发请求，其实这个 key 已经不在那个节点上了。此时这个节点会告诉客户端 key 已经不在我这里了，同时附上 key 现在所在的节点信息，让客户端再去请求一次。节点只处理自己拥有的 key，对于不拥有的 key 将返回重定向错误，即 -MOVED key 127.0.0.1:6381，客户端重新向这个新节点发送请求。
一组相关的 key 映射到同一个节点上，一次能获取多个值。集群不支持多 key 命令，而是使用哈希标签，通过仅使用 key 中的位于 { 和 } 间的字符串参与计算哈希值。这样可以保证哈希值相同，落到相同的节点上，但是 key 又是不同的，不会互相覆盖。
Redis 集群的每个节点里只有一个线程负责接受和执行所有客户端发送的请求。
客户端和集群的交互过程是串行化阻塞式的，即客户端发送了一个命令后必须等到响应回来后才能发第二个命令。Redis 提供了一种管道技术，可以让客户端一次发送多个命令，期间不需要等待服务器端的响应，等所有的命令都发完了，再依次接收这些命令的全部响应。这就极大地节省了许多时间，提升了效率。只能在客户端模拟实现，就是使用多个连接往多个节点同时发送命令，然后等待所有的节点都返回了响应，再把它们按照发送命令的顺序整理好，返回给用户代码。    

​     

## 12.缓存问题

缓存穿透和缓存雪崩，是缓存最大的两个问题。

#### 缓存穿透

指的是对某个一定不存在的数据进行请求，该请求将会穿透缓存到达数据库。

**解决方案**

- 最基本的就是首先做好参数校验，一些不合法的参数请求直接抛出异常信息返回给客户端。

- 对这些不存在的数据缓存一个空数据。这种方式可以解决请求的 key 变化不频繁的情况，如果大量不同请求 key 需要将过期时间设置为很短。
- 对这类请求进行过滤。使用布隆过滤器判重，一定不存在的数据会被 BitSet 拦截掉，从而避免了对底层存储系统的查询压力。
- 缓存预热，使在缓存上查不到的数据变少。

#### 缓存雪崩

指的是由于数据没有被加载到缓存中，或者缓存数据在同一时间大面积失效（过期），又或者缓存服务器宕机，导致大量的请求都到达数据库。

在有缓存的系统中，系统非常依赖于缓存，缓存分担了很大一部分的数据请求。当发生缓存雪崩时，数据库无法处理这么大的请求，导致数据库崩溃。

**解决方案**

- 为了防止缓存在同一时间大面积过期导致的缓存雪崩，可以通过观察用户行为，合理设置缓存过期时间来实现。

- 为了防止缓存服务器宕机出现的缓存雪崩，可以使用分布式缓存，分布式缓存中每一个节点只缓存部分的数据，当某个节点宕机时可以保证其它节点的缓存仍然可用。
- 也可以进行缓存预热，避免在系统刚启动不久由于还未将大量数据进行缓存而导致缓存雪崩。
- 基于 Redis or ZooKeeper 实现互斥锁，等待第一个请求构建完缓存之后，再释放锁，进而其它请求才能通过该 key 访问数据。
- 利用定时线程在缓存过期前主动的重新构建缓存或者延后缓存的过期时间，以保证所有的请求能一直访问到对应的缓存。
- 写请求访问量大，可以采用集群分片方案，用分片分摊写流量。

事前：Redis 高可用，主从和哨兵，Redis Cluster，避免全盘崩溃。选择合适的内存淘汰策略。

事中：本地 Ehcache 缓存 + Hystrix 限流和降级，避免 MySQL 被打死。

事后：Redis 持久化，一旦重启，自动从磁盘上加载数据，快速恢复缓存数据。

#### 缓存无底洞

为了满足业务要求添加了大量缓存节点，但是性能不但没有好转反而下降了的现象。

缓存系统通常采用 hash 函数将 key 映射到对应的缓存节点，随着缓存节点数目的增加，键值分布到更多的节点上，导致客户端一次批量操作会涉及多次网络操作，这意味着批量操作的耗时会随着节点数目的增加而不断增大。此外，网络连接数变多，对节点的性能也有一定影响。

**解决方案**

- 优化批量数据操作命令。

- 减少网络通信次数。
- 降低接入成本，使用长连接/连接池，NIO 等。

#### *缓存一致性

要求数据库 MySQL 更新的同时缓存 Redis 也能够实时更新。

缓存里的数据要经常查询，最好不易改变，且数据的正确性对最终结果影响不大。

**解决方案**

- 在数据更新的同时立即去更新缓存。

- 在读缓存之前先判断缓存是否是最新的，如果不是最新的先进行更新。

要保证缓存一致性需要付出很大的代价，缓存数据最好是那些对一致性要求不高的数据，允许缓存数据存在一些脏数据。

**删除缓存代替更新缓存**

很多时候，在复杂点的缓存场景，缓存不单单是数据库中直接取出来的值。另外更新缓存的代价有时候是很高的。删除缓存，而不是更新缓存，就是一个 lazy 计算的思想，让它到需要被使用的时候再重新计算。

**导致数据不一致的场景**

- 更新数据库成功，删除缓存失败，数据库中是新数据，缓存中是旧数据，数据不一致；

- 删除缓存成功，更新数据库失败，数据库中是旧数据，缓存中是空的。读的时候缓存没有，所以去读了数据库中的旧数据，然后更新到缓存中。数据一致。

**高并发场景**

删除了缓存，然后要去修改数据库，此时还没修改。一个请求过来，去读缓存，发现缓存空了，去查询数据库，查到了修改前的旧数据，放到了缓存中。随后数据变更的程序完成了数据库的修改。数据不一致。

**解决方案**

- 更新数据的时候，根据数据的唯一标识，将操作路由之后，发送到一个 JVM 内部队列中。一个队列对应一个工作线程，每个工作线程串行拿到对应的操作，然后一条一条的执行。优化点，如果发现队列中已经有一个更新缓存的请求了，那么就不用再放个更新请求操作进去了，直接等待前面的更新操作请求完成即可。

- 读请求和写请求串行化，串到一个内存队列里去，这样就可以保证一定不会出现不一致的情况。串行化之后，就会导致系统的吞吐量会大幅度的降低。或者加锁。有问题，读请求长时阻塞，读请求并发量过高，要求多个实例的请求路由到相同的服务实例上，热点商品的路由问题导致请求的倾斜。

**Cache Aside Pattern**

1. 读的时候，先读缓存，缓存没有的话，就读数据库，然后取出数据后放入缓存，同时返回响应。

2. 更新的时候，先更新数据库，然后再删除缓存。



## 13.一致性哈希 DHT

#### 数据分布

**哈希分布**

哈希分布就是将数据计算哈希值之后，按照哈希值分配到不同的节点上。例如有 N 个节点，数据的主键为 key，则将该数据分配的节点序号为：hash(key)%N。

**问题**

当节点数量变化时，也就是 N 值变化，那么几乎所有的数据都需要重新分布，将导致大量的数据迁移。

对于 Redis 来说：一旦某一个 master 节点宕机，所有请求过来，都会基于最新的剩余 master 节点数去取模，尝试去取数据。这会导致大部分的请求过来，全部无法拿到有效的缓存，导致大量的流量涌入数据库。

**顺序分布**

将数据划分为多个连续的部分，按数据的 ID 或者时间分布到不同节点上。

**顺序分布相比于哈希分布的主要优点**

能保持数据原有的顺序。

并且能够准确控制每台服务器存储的数据量，从而使得存储空间的利用率最大。

#### 一致性哈希

是一种哈希分布方式，其目的是为了克服传统哈希分布在服务器节点数量变化时大量数据迁移的问题。

将哈希空间 [0, 2^(n-1)] 看成一个哈希环，每个服务器节点都配置到哈希环上。每个数据对象通过哈希取模得到哈希值之后，存放到哈希环中顺时针方向第一个大于等于该哈希值的节点上。

一致性哈希在增加或者删除节点时只会影响到哈希环中相邻的节点。

删除节点：原节点到前一个节点（q 顺时针方向）之间的数据会被重定位到后一个节点。

增加节点：新节点到前一个节点之间的数据会被重定位到新节点。

**虚拟节点**

一致性哈希存在数据分布不均匀的问题，节点存储的数据量有可能会存在很大的不同。导致造成缓存热点问题。

数据不均匀主要是因为节点在哈希环上分布的不均匀，这种情况在节点数量很少的情况下尤其明显。

解决方式是通过增加虚拟节点，然后将虚拟节点映射到真实节点上。虚拟节点的数量比真实节点来得多，那么虚拟节点在哈希环上分布的均匀性就会比原来的真实节点好，从而使得数据分布也更加均匀。



## 14.Redis 单线程为什么还能这么快？

#### 单线程

Redis 用单个 CPU 绑定一块内存的数据，然后针对这块内存的数据进行多次读写的时候，都是在一个 CPU 上完成的，所以它是单线程的。

Redis 内部使用文件事件处理器 file event handler 是单线程的，所以 Redis 才叫做单线程的模型。

**file event handler**

包含多个 socket，I/O 多路复用程序，文件事件分派器，事件处理器。

多个 socket 可能会并发产生不同的操作，每个操作对应不同的文件事件，但是非阻塞 I/O 多路复用机制同时监听多个 socket，会将 socket 产生的事件放入队列中排队，事件分派器每次从队列中取出一个事件，把该事件交给对应的事件处理器进行处理。

**非阻塞多路 I/O 复用的优点**

- 让单个线程高效地处理多个 socket 连接请求，尽量减少网络 I/O 的时间消耗。

- 只在调用 select、poll、epoll 这些调用的时候才会阻塞，收发客户消息是不会阻塞的，整个进程或者线程就被充分利用起来，提高 CPU 利用率。

#### 为什么单线程还能这么快？

- 核心是基于非阻塞的 I/O 多路复用机制，用很少的线程也能快速处理大量任务。
- 采用单线程，避免了不必要的上下文切换和竞争问题而消耗 CPU，不用去考虑多线程同步和各种锁的问题，如加锁释放锁和死锁导致的性能消耗。单线程天然支持原子操作。

- 完全基于内存，绝大部分请求是纯粹的内存操作，非常快速。数据存在内存中，类似于 HashMap，查找和操作的时间复杂度都是 O(1)。

- 因为 Redis 是基于内存的操作，CPU 不是 Redis 的瓶颈，Redis 的瓶颈主要为网络带宽和机器内存大小。

- 单线程容易实现。

- 数据结构简单，对数据操作也简单，Redis 中的数据结构是专门进行设计的。

- 底层模型，Redis 构建了自己的 VM 机制，调用一般的系统函数会浪费一定的时间去移动和请求。

- C 语言实现，一般来说，C 语言实现的程序距离操作系统更近，执行速度相对会更快。

- 客户端与服务端通信使用 Redis 序列化协议 RESP 进行通信，RESP 的设计初衷是实现简单、快速解析、可阅读，RESP 是可以序列化不同的数据类型的二进制安全协议。

**单线程效率高**

不是说高性能服务器一定是多线程实现的，多线程一定比单线程效率高。

Redis 核心就是数据全都在内存里，单线程去操作就是效率最高的。

因为多线程的本质就是 CPU 模拟出来多个线程的情况，这种模拟出来的情况就有一个代价，就是上下文的切换，对于一个内存的系统来说，它没有上下文的切换就是效率最高的。

内存是一个 IOPS 非常高的系统，因为我想申请一块内存就申请一块内存，销毁一块内存我就销毁一块内存，内存的申请和销毁是很容易的。而且内存是可以动态的申请大小的。

对于下层存储如磁盘、网络、SSD 等慢速设备，应该用多线程。

为何单核 CPU 绑定一块内存效率最高：不能任由操作系统负载均衡，可以手动分配 CPU 核，而不会过多地占用 CPU。默认情况下单线程在进行系统调用的时候会随机使用 CPU 内核，为了优化 Redis，我们可以使用工具为单线程绑定固定的 CPU 内核，减少不必要的性能损耗。

Redis 作为单进程模型的程序，为了充分利用多核 CPU，常常在一台 server 上会启动多个实例。而为了减少切换的开销，有必要为每个实例指定其所运行的 CPU。Linux 上 taskset 可以将某个进程绑定到一个特定的 CPU。 



## 17.客户端与 Redis 的一次通信过程

首先，Redis 服务端进程初始化的时候，会将 server socket 的 AE_READABLE 事件与连接应答处理器关联。

客户端 socket01 向 Redis 进程的 server socket 请求建立连接，此时 server socket 会产生一个 AE_READABLE 事件，IO 多路复用程序监听到 server socket 产生的事件后，将该 socket 压入队列中。
文件事件分派器从队列中获取 socket，交给连接应答处理器。连接应答处理器会创建一个能与客户端通信的 socket01，并将该 socket01 的 AE_READABLE 事件与命令请求处理器关联。

假设此时客户端发送了一个 set key value 请求，此时 Redis 中的 socket01 会产生 AE_READABLE 事件，IO 多路复用程序将 socket01 压入队列，此时事件分派器从队列中获取到 socket01 产生的 AE_READABLE 事件，由于前面 socket01 的 AE_READABLE 事件已经与命令请求处理器关联，因此事件分派器将事件交给命令请求处理器来处理。
命令请求处理器读取 socket01 的 key value 并在自己内存中完成 key value 的设置。操作完成后，它会将 socket01 的 AE_WRITABLE 事件与命令回复处理器关联。

如果此时客户端准备好接收返回结果了，那么 Redis 中的 socket01 会产生一个 AE_WRITABLE 事件，同样压入队列中，事件分派器找到相关联的命令回复处理器，由命令回复处理器对 socket01 输入本次操作的一个结果，比如 ok，之后解除 socket01 的 AE_WRITABLE 事件与命令回复处理器的关联。

这样便完成了一次通信。



## 18.Redis 的并发竞争问题

多个系统同时对一个 key 进行操作，但是最后执行的顺序和我们期望的顺序不同，这样也就导致了结果的不同！
多客户端同时并发写一个 key，可能本来应该先到的数据后到了，导致数据版本错了；或者是多客户端同时获取一个 key，修改值之后再写回去，只要顺序错了，数据就错了。
Redis 自带的 CAS 类的乐观锁。
分布式锁 ZooKeeper 和 Redis。
某个时刻，多个系统实例都去更新某个 key。可以基于 ZooKeeper 实现分布式锁。每个系统通过 ZooKeeper 获取分布式锁，确保同一时间，只能有一个系统实例在操作某个 key，别人都不允许读和写。
你要写入缓存的数据，都是从 MySQL 里查出来的，都得写入 MySQL 中，写入 MySQL 中的时候必须保存一个时间戳，从 MySQL 查出来的时候，时间戳也查出来。
每次要写之前，先判断一下当前这个 value 的时间戳是否比缓存里的 value 的时间戳要新。如果是的话，那么可以写，否则，就不能用旧的数据覆盖新的数据。



## 19.为什么要用 Redis 而不用 map/guava 做缓存?

缓存分为本地缓存和分布式缓存。
Java 自带的 map 或者 guava 实现的是本地缓存，最主要的特点是轻量以及快速，生命周期随着 JVM 的销毁而结束，并且在多实例的情况下，每个实例都需要各自保存一份缓存，缓存不具有一致性。
使用 Redis 或 Memcached 之类的称为分布式缓存，在多实例的情况下，各实例共用一份缓存数据，缓存具有一致性。缺点是需要保持 Redis 或 Memcached 服务的高可用，整个程序架构上较为复杂。