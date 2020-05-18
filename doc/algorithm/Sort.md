排序算法
=========================

> 在这一篇中，写的 Node 代码尽量不使用内置的函数，以便未来如果用其他语言实现，需要相应的修改。
>

## 冒泡排序

快速排序是冒泡排序的改进版，冒泡排序是最简单，也是最符合直觉的算法。原理如下：

- 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
- 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。
- 针对所有的元素重复以上的步骤，除了最后一个。
- 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

```js
function BubbleSort(arr) {
  let n = arr.length;
  // 遍历数组，每当 i + 1，都会有一个最大的值被交换到最后面
  for (let i = 0; i < n - 1; i++) {
    // 遍历数组，每次都从比较 0, 1 开始，，并且每完成一次遍历，都有一个最大值被移动到最后，所以长度 - i
    for (let j = 1; j < n - i; j++) {
      // 左边 > 右边，交换数值
      if (arr[j - 1] > arr[j]) {
         swap(arr, i, j);
      }
    }
  }
  return arr;
}

function swap(arr, i, j) {
  [arr[i], arr[j]] = [arr[j], arr[i]];
}
```

对于长度为 n 的数组，冒泡排序需要经过 n (n - 1) / 2 次比较，最坏的情况下，即数组本身是倒序的情况下，需要经过 n (n - 1) / 2 次交换，是一个稳定的算法，所以其平均复杂度为O(n²)。空间复杂度为 O(1)。

## 选择排序

选择排序也是一种简单的排序算法，他的思想和冒泡排序几乎是相同的，但操作相反。原理如下：

- 从待排序序列中，找到关键字最小的元素；起始假定第一个元素为最小

- 如果最小元素不是待排序序列的第一个元素，将其和第一个元素互换；

- 从余下的 N - 1 个元素中，找出关键字最小的元素，重复1，2步，直到排序结束。

```javascript
function SelectSort(arr) {
  let n = arr.length;
  // 遍历数组，每当 i + 1，都会标记一个最小值，并在最后结束时交换到 i 的位置
  for (let i = 0; i < n; i++) {
    let minIndex = i;
    for (let j = i + 1; j < n; j++) {
      // 左边 < 右边，标记
      if (arr[j] < arr[minIndex]) {
        minIndex = j;
      }
    }
    swap(arr, i, minIndex);
  }
  return arr;
}

function swap(arr, i, j) {
  [arr[i], arr[j]] = [arr[j], arr[i]];
}
```

选择排序也是需要 n (n - 1) / 2 次比较完成排序，但他不是一个稳定的算法，因为如果出现类似 54516 的数组，那么经过第一次交换后，第一个 5 和 第三个 5 的位置顺序被破坏了，所以不是一个稳定的算法。 其平均复杂度为O(n²)。空间复杂度为 O(1)。

## 插入排序

插入排序也是一种简单的算法，他的原理和选择排序相近，但更加的优化，他不需要反复遍历整个数组，而是每次只从已经排序过的区间内，进行插入操作（类似打扑克时的理牌）。原理如下：

- 从第一个元素开始，该元素可以认为已经被排序
- 取出下一个元素，在已经排序的元素序列中从后向前扫描
- 如果该元素（已排序）大于新元素，将该元素移到下一位置
- 重复步骤 3，直到找到已排序的元素小于或者等于新元素的位置
- 将新元素插入到该位置后
- 重复步骤 2~5

```javascript
function InsertionSort(arr) {
  let n = arr.length;
  // 默认 i = 0是已经排序的，所以从 i = 1 开始遍历
  for (let i = 1; i < n; i++) {
    // 从 j = i 开始，每次都与已经排序过的进行比较并交换，直到自身比前值大，退出
    for (let j = i; j > 0; j--) {
      if (arr[j - 1] <= arr[j]) {
        break;
      }
    swap(arr, i, j);
    }
  }
  return arr;
}

function swap(arr, i, j) {
  [arr[i], arr[j]] = [arr[j], arr[i]];
}
```

这个算法比冒泡更优秀的点是：他始终只与已经排序过的序列进行比较，并且可以提前终止遍历，插入排序不会改变原有元素间的顺序，所以是稳定的，其平均复杂度为O(n²)。空间复杂度为 O(1)。

> 插入排序对于有序数组，效率更高。

## 归并排序

归并算法通过将长度为 n 数组拆成 n  个由一个元素组成的数组，然后分别与邻近的数组两两比较并排序变成一个由两个元素组成的数组，以此类推，最终将所有数组排序。它用的是分治的思想。原理如下：

- 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
- 设定两个指针，最初位置分别为两个已经排序序列的起始位置
- 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
- 重复步骤3直到某一指针到达序列尾
- 将另一序列剩下的所有元素直接复制到合并序列尾

```javascript
function MergeSort(arr) {
  let n = arr.length;
  if (n < 2) return arr;
  
  // 第一步，将数组拆分为左右各一半的数组，
  let mid = Math.floor(n / 2);
  
  // 第二步，递归调用 MergeSort 将左右数组继续往下拆，拆到最底层，再一层一层往上合并
  let left = MergeSort(arr.slice(0, mid));
  let right = MergeSort(arr.slice(mid, n));
  
  // 第三步，将最终的数组合并
  return merge(left, right);
  
  // 合并方法
  function merge(left, right) {
    // 遍历左右数组，根据大小逐个出栈，放入新的数组，直到左右数组有任意一个为空
    let result = [];
    while (left.length > 0 && right.length > 0) {
      if (left[0] <= right[0]) {
        result.push(left.shift());
      } else {
        result.push(right.shift());
      }
    }
    
    // 将剩下的元素全部出栈，放入新的数组
    while (left.length) {
      result.push(left.shift());
    }
    
    while (right.length) {
      result.push(right.shift());
    }
    
    return result;
  }
}
```

归并排序无论在什么情况下，他的时间复杂度都是O(n log n)。空间复杂度为 O(n)，是一个稳定的排序算法。

## 快速排序

这是开发人员用的最多的排序算法，性能比归并更好，采用的是分治的思想，快速排序有几个变种，单路，双路，三路等。他们的区别在于选取几个“基准”来对数组进行排序。原理：

- 首选从数组中选取一个数字，作为基准
- 遍历数组，将比基准大的数字丢到右边，比基准小的数字丢到左边（类似树的形状）
- 最后将左右两个数组，再递归上述流程
- 最终获得的值就是排序过的数组。

> 关于快速排序，基准的选择很重要
>
> - 如果是一个随机数组，那么选择第一个或最后一个，都没有问题。但如果是一个有序待排数组，那么就会很糟糕，它的树形就会变成一边重，一边轻。
> - 选择基准有几个方案：
>   - 完全随机，隐患：正好随机到接近最大或最小值。 （O1）
>   - 推荐取三个数的中值，举例：取出首，中，尾三个值进行比较，排除掉最大值和最小值，就可以得到中值了 （O1）

```javascript
function QuickSort(arr) {
  let n = arr.length;
  if (n < 2) return arr;
  
  // 比较三值，进行交换，升序排列
  let low = 0, mid = Math.floor(n / 2), high = n;
  if (arr[low] > arr[mid]) swap(arr, low, mid);
  if (arr[mid] > arr[high]) swap(arr, mid, high);
  
  // 对数组进行分区
  let base = arr.splice(mid, 1)[0];
  let left = [], right = [];
  for (let i = 0; i < arr.length; i++) {
    (arr[i] < base) ? left.push(arr[i]) : right.push(arr[i]);
  }
  return QuickSort(left).concat(base, QuickSort(right));
}

function swap(arr, i, j) {
  [arr[i], arr[j]] = [arr[j], arr[i]];
}
```

快速排序是一个不稳定的排序算法，当基准值选的很差的时候，最坏情况下复杂度为 O(n²)，其平均复杂度为nlogn)。空间复杂度为 O(logn)。

## 堆排序

堆排序是指利用堆这种数据结构所设计的一种排序算法。堆积是一个近似完全二叉树的结构，并同时满足堆的性质：即子结点的键值或索引总是小于（或者大于）它的父节点。原理：

- 将初始二叉树转化为大顶堆，此时根节点为最大值，将其与最后一个节点交换。
- 除开最后一个节点，将其他节点组成一个新堆并转化为大顶堆，此时根节点为次最大值，将其与最后一个节点交换
- 重复步骤2，直到堆中元素个数为1，排序完成

通过以上原理，可以将堆排序的过程分解成两步：建堆和排序

1. 将数组 **原地** （不借助其他数组）建成一个大顶堆。
2. 从下往上，将最后一个节点与根节点交换，每次堆化完成后，根节点总是最大的节点。

```javascript
function HeapSort(arr) {
  let n = arr.length;
  if (n < 2) return arr;
  
  // 初始化大顶堆
  // 因为大顶堆是一个完全二叉树，所以从 n/2 到 1 都属于子树节点，n / 2+ 1 到 n 都属于叶子节点，不需要堆化
  for (let i = Math.floor(n / 2); i >= 0; i--) {
    heapify(arr, n, i);
  }
  
  // 首尾交换，并重新堆化，每次堆化，堆顶总是最大值
  let len = n;
  for (let i = n - 1; i > 0; i--) {
    swap(arr, 0, i);
    len--;
    heapify(arr, len, 0);
  }
  return arr;
  
  // 堆化
  function heapify(arr, len, i) {
    // 当前子树节点 i 的左右子节点的位置总是 2 * i + 1 和 2 * i + 2;
    let largest = i, left = 2 * i + 1, right = 2 * i + 2;
    if (left < len && arr[left] > arr[largest]) largest = left;
    if (right < len && arr[right] > arr[largest]) largest = right;
    if (largest !== i) {
      swap(arr, i, largest);
      heapify(arr, largest);
    }
  }
}

function swap(arr, i, j) {
  [arr[i], arr[j]] = [arr[j], arr[i]];
}
```

堆排序也是不稳定的排序算法，并且不论什么情况下它的时间复杂度都为O(nlogn)，空间复杂度为O(1).

### 扩展问题：堆排序比快速排序更快吗？

堆排序的时间复杂度为O(nlogn)，而快速排序平均也是O(nlogn)，最坏情况下是O(n²)，是不是说明堆排序比快速排序更优秀，效率更高？

```
数据规模    快速排序      堆排序
1000万       0.71        3.51
5000万       3.75       26.54  
1亿          7.72       61.35
```

从排序测试的统计结果来看，堆排序并没有比快速排序更优秀，原因有以下几点：

1. 堆排序访问数据的方式没有快速排序友好。快速排序是局部顺序访问，而堆排序是跳着访问再进行堆化，几乎不访问相邻元素，对 CPU 缓存不友好。
2. 数学上的时间复杂度并不代表实际运行上的情况

**标准库 Sort 就是使用的是优化后的快速排序：先快速排序，当递归深度达到一个阈值就改为堆排，然后对最后的几个进行插入排序**

## 计数排序

计数排序非常的符合直觉，简单概括就是先遍历一边，找到最小值 n 和最大值 m，然后构建一个长度为 m - n 的空间，再次遍历记录每个数出现的位置及次数，最后根据这个空间取出排序的结果。

> 正常的计数排序不能对负数进行排序，下面的进行了优化.

```javascript
function CounterSort(arr) {
  let n = arr.length;
  if (n < 2) return arr;
  
  // 找到最大最小值，
  let pCounter = new Array(Math.max(...arr) + 1);
  let nCounter = new Array(Math.abs(Math.min(...arr)));
  for (let v of arr) {
    if (v < 0) {
      v = Math.abs(v);
      (!nCounter[v]) ? nCounter[v] = 1 : nCounter[v]++
    } else {
      (!pCounter[v]) ? pCounter[v] = 1 : pCounter[v]++
    }
  }
  let res = [];
  if (nCounter.length > 0) {
    for (let i = nCounter.length - 1; i >= 0; i--) {
      while (nCounter[i] > 0) {
        res.push(-i);
        nCounter[i]--;
      }
    }
  }
  if (pCounter.length > 0) {
    for (let i = 0; i <= pCounter.length - 1; i++) {
      while (pCounter[i] > 0) {
        res.push(i);
        pCounter[i]--;
      }
    }
  }
  return res;
}
```

计数排序的问题有很多，原版是不支持负数排序的，并且无法对浮点数排序，类似 [4000, 5000] 的数组，会有内存的浪费，等等等等，但都可以通过算法上进行弥补，最后的执行效率并不如快速排序等常用排序算法更有效率。