
Node.js Profile
=========================

[TOC]

---

## Debug 工具

Node Inspector 是 Node.js 应用程序的调试器接口，通过启动 Node 进程使用 `--inspect` 参数，就可以使用 Chrome DevTools 工具来进行 debugging。

```bash
node --inspect test.js
```

## Profile 工具

通过启动 Node 进程使用`--prof` 参数，就会在当前目录生成一个文件名类似 `isolate-000002EFCA9485C0-v8.log` 的日志文件

```bash
node --prof test.js
```

由于该日志文件内容实际是没有可读性的，我们可以通过 Node 的日志解析工具生成可读内容

```bash
node --prof-process isolate-000002EFCA9485C0-v8.log > processed.txt
```

最后我们可以得到这样一份文档

```bash
Statistical profiling result from isolate-000002EFCA9485C0-v8.log, (9552 ticks, 3 unaccounted, 0 excluded).

 [Shared libraries]:
   ticks  total  nonlib   name
   9264   97.0%          C:\WINDOWS\SYSTEM32\ntdll.dll
    250    2.6%          E:\data1\app\node\node.exe
      7    0.1%          C:\WINDOWS\System32\KERNEL32.DLL
      1    0.0%          C:\WINDOWS\System32\RPCRT4.dll
      1    0.0%          C:\WINDOWS\System32\KERNELBASE.dll

 [JavaScript]:
   ticks  total  nonlib   name
      2    0.0%    6.9%  Stub: StringAddStub
      2    0.0%    6.9%  Function: ~_makeLong path.js:685:32
      2    0.0%    6.9%  Function: ~NativeModule.compile bootstrap_node.js:584:44
      2    0.0%    6.9%  Builtin: MapLookupHashIndex
      2    0.0%    6.9%  Builtin: LoadIC_Noninlined
      1    0.0%    3.4%  Function: ~realpathSync fs.js:1571:40
      1    0.0%    3.4%  Function: ~isAbsolute path.js:469:34
      1    0.0%    3.4%  Function: ~dirname path.js:724:28
      1    0.0%    3.4%  Function: ~assertEncoding internal/fs.js:19:24
      1    0.0%    3.4%  Function: ~_addListener events.js:233:22
      1    0.0%    3.4%  Function: ~Module._findPath module.js:176:28
      1    0.0%    3.4%  Function: ~InnerArrayJoin native array.js:275:24
      1    0.0%    3.4%  Function: ~DoJoin native array.js:95:16
      1    0.0%    3.4%  Builtin: StringPrototypeCharCodeAt
      1    0.0%    3.4%  Builtin: ObjectHasOwnProperty
      1    0.0%    3.4%  Builtin: LoadIC_Uninitialized
      1    0.0%    3.4%  Builtin: LoadICProtoArray
      1    0.0%    3.4%  Builtin: KeyedStoreIC_Megamorphic_Strict
      1    0.0%    3.4%  Builtin: InterpreterEntryTrampoline
      1    0.0%    3.4%  Builtin: FastNewObject
      1    0.0%    3.4%  Builtin: CallFunction_ReceiverIsAny

 [C++]:
   ticks  total  nonlib   name

 [Summary]:
   ticks  total  nonlib   name
     26    0.3%   89.7%  JavaScript
      0    0.0%    0.0%  C++
     31    0.3%  106.9%  GC
   9523   99.7%          Shared libraries
      3    0.0%          Unaccounted

 [C++ entry points]:
   ticks    cpp   total   name

 [Bottom up (heavy) profile]:
  Note: percentage shows a share of a particular caller in the total
  amount of its parent calls.
  Callers occupying less than 1.0% are not shown.

   ticks parent  name
   9264   97.0%  C:\WINDOWS\SYSTEM32\ntdll.dll

    250    2.6%  E:\data1\app\node\node.exe
    213   85.2%    E:\data1\app\node\node.exe
     62   29.1%      Function: ~createScript vm.js:79:22
     62  100.0%        Function: ~runInThisContext vm.js:138:26
     62  100.0%          Function: ~Module._compile module.js:600:37
     62  100.0%            Function: ~Module._extensions..js module.js:652:37
     21    9.9%      Function: ~runInThisContext bootstrap_node.js:495:28
     21  100.0%        Function: ~NativeModule.compile bootstrap_node.js:584:44
     21  100.0%          Function: ~NativeModule.require bootstrap_node.js:516:34
      3   14.3%            Function: ~NativeModule.compile bootstrap_node.js:584:44
      3   14.3%            Function: ~<anonymous> module.js:1:11
      2    9.5%            Function: ~startup bootstrap_node.js:12:19
      2    9.5%            Function: ~Module._load module.js:442:24
      2    9.5%            Function: ~<anonymous> stream.js:1:11
      2    9.5%            Function: ~<anonymous> fs.js:1:11
      1    4.8%            Function: ~setupNextTick internal/process/next_tick.js:49:23
      1    4.8%            Function: ~setupGlobalVariables bootstrap_node.js:251:32
      1    4.8%            Function: ~setupGlobalTimeouts bootstrap_node.js:296:31
      1    4.8%            Function: ~setupGlobalConsole bootstrap_node.js:306:30
      1    4.8%            Function: ~<anonymous> util.js:1:11
      1    4.8%            Function: ~<anonymous> internal/child_process.js:1:11
      1    4.8%            Function: ~<anonymous> http.js:1:11
     11    5.2%      Function: ~realpathSync fs.js:1571:40
     11  100.0%        Function: ~toRealPath module.js:157:20
     10   90.9%          Function: ~tryFile module.js:149:17
      7   70.0%            Function: ~tryExtensions module.js:164:23
      3   30.0%            Function: ~tryPackage module.js:129:20
      1    9.1%          Function: ~Module._findPath module.js:176:28
      1  100.0%            Function: ~Module._resolveFilename module.js:508:35
     10    4.7%      Function: ~stat module.js:50:14
      7   70.0%        Function: ~Module._findPath module.js:176:28
      7  100.0%          Function: ~Module._resolveFilename module.js:508:35
      7  100.0%            Function: ~Module._load module.js:442:24
      3   30.0%        Function: ~tryFile module.js:149:17
      3  100.0%          Function: ~tryExtensions module.js:164:23
      3  100.0%            Function: ~Module._findPath module.js:176:28
      8    3.8%      Function: ~fs.openSync fs.js:642:23
      8  100.0%        Function: ~fs.readFileSync fs.js:548:27
      8  100.0%          Function: ~Module._extensions..js module.js:652:37
      8  100.0%            Function: ~Module.load module.js:547:33
      8    3.8%      Function: ~createWriteReq net.js:789:24
      8  100.0%        Function: ~Socket._writeGeneric net.js:708:42
      8  100.0%          Function: ~Socket._write net.js:785:35
      8  100.0%            Function: ~doWrite _stream_writable.js:379:17
      7    3.3%      Function: ~NativeModule.compile bootstrap_node.js:584:44
      7  100.0%        Function: ~NativeModule.require bootstrap_node.js:516:34
      7  100.0%          Function: ~NativeModule.compile bootstrap_node.js:584:44
      7  100.0%            Function: ~NativeModule.require bootstrap_node.js:516:34
      6    2.8%      Function: ~readPackage module.js:107:21
      6  100.0%        Function: ~tryPackage module.js:129:20
      6  100.0%          Function: ~Module._findPath module.js:176:28
      6  100.0%            Function: ~Module._resolveFilename module.js:508:35
      5    2.3%      Function: ~fs.readSync fs.js:670:23
      5  100.0%        Function: ~tryReadSync fs.js:536:21
      5  100.0%          Function: ~fs.readFileSync fs.js:548:27
      4   80.0%            Function: ~Module._extensions..js module.js:652:37
      1   20.0%            Function: ~<anonymous> E:\data1\www\matrix\matrixes-simple\node_modules\grpc\index.js:1:11
      3    1.4%      Function: ~ProtoBuf E:\data1\www\matrix\matrixes-simple\node_modules\protobufjs\dist\protobuf.js:22:10
      3  100.0%        Function: ~<anonymous> E:\data1\www\matrix\matrixes-simple\node_modules\protobufjs\dist\protobuf.js:1:11
      3  100.0%          Function: ~Module._compile module.js:600:37
      3  100.0%            Function: ~Module._extensions..js module.js:652:37

```

我们可以看到上面的日志中有几个部分：

- Shared libraries：Node进程使用到的系统级动态链接库部分的时间消耗
- JavaScript，C++，Summary：
  - JavaScript ：在 JavaScript 代码部分的时间消耗，其中 name 列代表 JS 函数名，例如
    - Function：普通的 JS 函数
    - LazyCompile：懒编译的 JS 函数
    - RegExp：正则表达式函数
    - Builtin： Node 运行时的内建 JS 函数
    - Stub：Node 运行时的内建 C 函数
  - C++ ：在 C++ 代码里的时间消耗
  - **Summary **：所有的分类的时间消耗总量都放在一起，最重要的一部分，其中 GC 代表内存回收的时间消耗，Unaccounted 代表 profile 工具无法确定部分的时间消耗。


- C++ entry points：Node bindings 层，从 JS 代码跨界到 C++ 代码运行时，其中消耗的时间。这个概念在上文中已经有过描述。
- Bottom up (heavy) profile：真正的性能问题暴露部分，每个段落代表一个调用栈。
