redis

主要数据结构：

链表是integers列表建的底层实现

发布与订阅，慢查询，监视器等功能也用到了链表



redis的数据库使用字典来作为底层实现的

字典是hash键的底层实现之一



跳跃表是有序集合键的底层实现之一



整数集合是集合键的底层实现之一



压缩列表是列表键和hash键的底层实现之一



对象：

Redis用这些数据结构创建了一个对象系统，包括：字符串对象，列表对象，哈希对象，集合对象和有序集合对象。

可以针对不同的使用场景，给对象设置多种不同的数据结构实现。

redis底层有8种数据结构，支撑这5种对象类型。每种类型的对象都至少使用了两种不同的编码。



9、数据库

12、事件

文件事件：redis服务器对套接字socket的操作，构成，类型，处理器

时间时间：redis服务器的定时操作

IO多路复用