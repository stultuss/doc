
Node.js v8 引擎
=========================
## V8 内存回收

什么是 V8 Log，以及什么是 V8 GC， 下面有几篇 Alinode 团队写的解读，写的非常详细，链接如下：

- [解读 V8 GC Log（一）: Node.js 应用背景与 GC 基础知识](https://yq.aliyun.com/articles/592878)
- [解读 V8 GC Log（二）: 堆内外内存的划分与 GC 算法](https://yq.aliyun.com/articles/592880?spm=a2c4e.11153940.0.0.4c32dcdeWfBogU)

主要需要记忆两块内容：

- 弱分代假设：
  - 新生代 
  - 老生代

- 回收算法
  - 新生代：Scavenge
    - 一个内存空间，分为两半，使用中的叫 from space，未使用的脚 to space
    - 当内存空间不足时触发启动，调换两个 space，然后做宽度扫描，将所有存活的对象复制到 to space，并清除 from space。
  - 晋升
    - 当启动 Scavenge 时，发现存活的对象已经存活过一次了，就会被晋升到老生代，而不会再复制到 to space 中了。
  - 老生代：Mark-Sweep／Mark-Compact
    - 扫描全部对象，并进行标记（三色 marking）
    - 白色表示可回收，黑色表示不可回收，灰色表示尚未扫描
- 堆外内存
  - buffer

## V8 内存监控

### 内存用量

获取内存堆栈情况

```javascript
// {
//   total_heap_size: 7708672, // 已申请的堆内存
//   total_heap_size_executable: 3670016, // 字节码、优化后的代码等可执行的内容占用的内存量
//   total_physical_size: 7708672, // 不明白，占用的物理内存？
//   total_available_size: 1492158752, // 剩余的堆内存 heap_size_limit - used_heap_size
//   used_heap_size: 4853224, // 已使用的堆内存
//   heap_size_limit: 1501560832, // 最大可用堆内存上限
//   malloced_memory: 8192, // 当前 malloc 申请的堆内存
//   peak_malloced_memory: 1242904, // 通过 malloc 申请的堆内存峰值（申请后还回去了）
//   does_zap_garbage: 0 // 是个0/1式布尔值，表示是否设置了--zap_code_space选项。若为真，那么V8引擎会用一个位模式来覆盖堆中的垃圾。
// }
console.log(require('v8').getHeapStatistics());
```

获取按 space 分类的不同空间的内存使用情况

```javascript
// [
// 新生代空间
//   { space_name: 'new_space',
//     space_size: 2097152,
//     space_used_size: 618920,
//     space_available_size: 412248,
//     physical_space_size: 2097152 },
// 老生代空间
//   { space_name: 'old_space',
//     space_size: 2969600,
//     space_used_size: 2747624,
//     space_available_size: 88,
//     physical_space_size: 2969600 },
// 代码空间
//   { space_name: 'code_space',
//     space_size: 2097152,
//     space_used_size: 1205024,
//     space_available_size: 0,
//     physical_space_size: 2097152 },
// Map 空间
//   { space_name: 'map_space',
//     space_size: 544768,
//     space_used_size: 281776,
//     space_available_size: 0,
//     physical_space_size: 544768 },
// LargeObject 空间
//   { space_name: 'large_object_space',
//     space_size: 0,
//     space_used_size: 0,
//     space_available_size: 1491746304,
//     physical_space_size: 0 }
// ]
console.log(require('v8').getHeapSpaceStatistics());
```

### V8 GC监控

我们可以通过以下命令，获取到 V8 GC Log 的 option 列表

```bash
node --v8-options | grep gc
```

其中，我们会常用如下几个：

- --expose-gc：在 Runtime 中，通过 global 触发 GC

```javascript
if (global.gc) {
    global.gc();
}
```

- --trace-gc：打印每次GC的简要描述。
- --trace-gc_nvp：打印每次GC的详细日志，按键值对的方式。
- --trace-gc_verbose：打印每次GC的详细日志，以及每个空间的变化。

关于 GC 日志的格式，以及每一行的含义，可以看 [Node.JS Profile 1.2 V8 GC详解](https://xenojoshua.com/2018/01/node-v8-gc/)

## 案例

关于内存溢以及排查流程的经典案例：[Node.js 调试 GC 以及内存暴涨的分析](https://blog.devopszen.com/node-js_gc)