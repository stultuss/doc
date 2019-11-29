排序算法
=========================

> 记录面试中的一些问题
>

https://github.com/MisterBooo/LeetCodeAnimation

在这一篇中，写的 Node 代码尽量不使用内置的函数，以便未来如果用其他语言实现，需要相应的修改。

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
        arr[j] = arr[j] ^ arr[j - 1];
        arr[j - 1] = arr[j] ^ arr[j - 1];
        arr[j] = arr[j] ^ arr[j - 1];
      }
    }
  }
  return arr;
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
    
    // 交换
    if (arr[minIndex] < arr[i]) {
      arr[i] = arr[i] ^ arr[minIndex];
      arr[minIndex] = arr[i] ^ arr[minIndex];
      arr[i] = arr[i] ^ arr[minIndex];
    }
  }
  return arr;
}
```

