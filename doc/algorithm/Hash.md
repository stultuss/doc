哈希算法
=========================

> 读到了一篇 Jason Davies 写的 Bloom Filter 的[科普](https://www.jasondavies.com/bloomfilter/)，他使用了 FNV-1a 的哈希算法编写了一个非常快速的 Bloom Filter 库，并在结尾中提示：**Unfortunately I can't use the 64-bit trick in the linked post as JavaScript only supports bitwise operations on 32 bits.**  在这个基础上才让我产生了重新复习一遍哈希算法的想法，了解其运作机制以及执行效率。

## FNV 算法

FNV 取自 Glenn Fowler 和 Phong Vo 向 IEEE POSIX P1003.2委员会发送评论者意见的想法，在随后的投票中：Landon Curt Noll 对他们的算法进行了改进。该算法被设计为快速，同时保持较低的冲突率。

FNV 散列的高度分散下使其非常适合散列几乎相同的字符串，例如：URL，主机名，文件名等。

### 版本

- FNV
- FNV-1
- FNV-1a

### 公式

FNV-1a 算法公式：

```
hash = offset_basis
for each octet_of_data to be hashed
  hash = hash xor octet_of_data
	hash = hash * FNV_prime
return hash
```

**其与FNV-1算法的区别就是异或运算与乘法运算顺序调整了，即是说FNV-1算法是先进行乘法运算再进行异或运算**

1. offset_basis 参数依赖于hash位数的大小
   - 32 bit offset_basis = 2166136261
   - 64 bit offset_basis = 14695981039346656037
   - 128 bit offset_basis = 144066263297769815596495629667062367629
2. offset_basis 计算脚本

```
hash_bits = insert_the_hash_size_in_bits_here;
FNV_prime = insert_the_FNV_prime_here;
offset_basis = 0;
offset_str = “chongo /\…/\”;
hash_mod = 2hash_bits;
str_len = strlen(offset_str);
for (i=1; i <= str_len; ++i) {
  offset_basis = (offset_basis * FNV_prime) % hash_mod;
  offset_basis = xor(offset_basis, ord(substr(offset_str,i,1)));
}
//输出
print hash_bits, “bit offset_basis =”, offset_basis;
```

2. FNV_prime参数依赖于hash位数的大小
   - 32 bit FNV_prime = power(2, 24) + power(2, 8) + 0x93 = 16777619
   - 64 bit FNV_prime = power(2, 40)  +  power(2, 8)  + 0xb3 = 1099511628211
   - 128 bit FNV_prime = power(2, 88)  +  power(2, 8)  + 0x3b = 309485009821345068724781371
3. octet_of_data 8位元数据或者单个字节数据

## Murmur 算法

由 Austion Appleyby 创建于 2008 年，该算法被设计为高性能，低碰撞率。现已被应用到 Hadoop、libstdc++、nginx、libmemcached 等开源系统中，2011 年 Appleby 被 Google 雇佣，随用 Google 推出其变种 CityHash 算法。官网网站：https://sites.google.com/site/murmurhash/ 

