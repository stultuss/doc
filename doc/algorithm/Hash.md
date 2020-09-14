哈希算法
=========================

> 读到了一篇 Jason Davies 写的 Bloom Filter 的[科普](https://www.jasondavies.com/bloomfilter/)，他使用了 FNV-1a 的哈希算法编写了一个非常快速的 Bloom Filter 库，并在结尾中提示：**Unfortunately I can't use the 64-bit trick in the linked post as JavaScript only supports bitwise operations on 32 bits.**  在这个基础上才让我产生了重新复习一遍哈希算法的想法，了解其运作机制以及执行效率。

