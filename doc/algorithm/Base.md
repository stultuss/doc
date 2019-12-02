算法思想
=========================

> 记录面试中的一些问题
>

## 分治法

> 在计算机科学中，分治法是一种很重要的算法，核心就是“分而治之“

基本思想：**将一个规模为 N 的问题，分解成 K 个规模较小的子问题。再把子问题分成更小的子问题，直到子问题可以简单求解，原问题的解就是子问题的解的合并**。

根据上述描述，可以总结以下几个特征：

1. 该问题的规模缩小到一定程度就可以很容易的解决
2. 该问题可以分解为若干个规模较小的相同问题（最优子结构）。
3. 利用该问题分解出来的子问题的解，可以合并为该问题的解。
4. 子问题是相同独立的，子问题不包含公共子问题。

注：**分治算法大多采用递归实现**，如果满足第一条和第二条特征，但不满足第三条特征，可以考虑 **贪心算法** 或 **动态规划**。

代码实现快速排序：

 ```javascript
function quickSort(arr) {
  if (arr.length < 2) return arr;
  let left = [], right = [];
  let base = arr[0]; // 取第一个做基准最简单，但有时会很糟糕。
  for (let i = 1; i < arr.length; i++) {
    (arr[i] < base) ? left.push(arr[i]) : right.push(arr[i]);
  }
  return quickSort(left).concat(base, quickSort(right));
}
 ```

## 倍增法

倍增算法又称为最近公共祖先算法。

基本思想：**利用二进制思想，想办法一步一步向上搜索变成以 2^k 的向上跳，所以定义 f\[\]\[\] 数组，使 f \[j\]\[i\] 表示节点 i 的 2^j 倍的祖先。**

算法实现理解：

1. 预处理（DFS）出所有节点的深度和父节点。
   - BFS 防止爆栈，无法处理孩子个数。
   - DFS 可能爆栈，可以处理孩子个数，使用时建议扩栈。
2. 处理每个节点的所有祖先节点。
3. 将所查询的两点 u，v 上升到同一个高度。
   - 找到祖先（以 2^k 的高速向上找）。
   - 未找到祖先，上升高度直到找到祖先。

代码实现后缀排序：

```javascript

```

## 贪心法

> 总的来说，贪心对程序员来说还是很简单的。平常工作中或有意或无意的会用到它。通常都是用递归实现。

总结一个思路：

- 和分治一样，分解成若干个子问题
- 计算局部最优解
- 再把局部最优解合成原来问题的一个解

虽然贪心的简单易用，相对的也会存在一些问题：

- 不能保证最后的解是最优解，**局部最优并不能带来全局最优！**
- 不能用来求最大值，最小值。
- 只能求满足特定条件的解。

贪心算法的基本模版

```
从问题的第一个解出发：
while(向总目标前进一步)
{
	利用可行的决策，求出一个子解
}
由所有子解组合成问题的一个可行解
```

贪心的具体实现，在后面和动态规划一起写。

## 动态规划

在贪心的基础上，有一些问题可能会出错。

例如：计算有一堆面值为 1，5，11 的纸币，要凑出总价值为 15 的纸币，**需要用到尽量少的钞票。**。如果使用贪心算法，最先算出最大面值的 11，然后再通过通过 1 凑出最终的 15，结果最终变成需要一张面值 11 的纸币和四张面值 1 的纸币。而正确答案只需要 3 张面值 5 的纸币即可。

动态规划一共有三条很重要的原则：

1. 决定策略，这需要一定数学和抽象的功底。
2. 将问题**分解**成几个相似的子问题，并且每个问题只解决一次
3. **存储** 所有子问题的解

那么回到上一个题目，首先是决策：

```
通过面值，我们知道利用 11 凑出 15，代价等于 f(4), 即 
取 11 	：cost = f(15 - 11) + 1张11 = (4张1) + 1张11 = 5, 代价是 5 张纸币。那么继续往下假设。
取 5 	：cost = f(15 - 5) + 1张5 	= (2张5) + 1张5 = 3,
取 1		：cost = f(15 - 1) + 1张1 	= (1张11 + 3张1) + 1张1 = 5

通过上面的三个式子，我们得出一个规律: f(n) 只和 f(n-1)，f(n-5)，f(n-11) 有关，
最终可以决定相关的等式为：f(n) = min{f(n-1),f(n-5),f(n-11)} + 1
```

怎么用代码实现：

```js
let n = 15; // 凑的纸币总值
let pars = [11, 5, 1].sort((a, b) => a - b); // 不同面值的纸币

// 初始化
let f = {};
f[0] = {
  cost: 0
};

// 分解子问题：求凑出 1～n 的面值的子问题
for (let i = 1; i <= n; i++) {
  let cost = Infinity;
  
  // 遍历面值的临时记录
  let parsValue = 0;
  let t = {};
  
  // 代入公示 f(n) = min(f(n-1),f(n-5),f(n-11)) + 1
  pars.forEach((par) => {
    if (i - par >= 0) {
      let fn = f[i - par].cost + 1;
      t[fn] = par;
      
      cost = Math.min(cost, fn); // 比较代价
      parsValue = t[cost]; // 判断最小代价属于那个面值产生的。
    }
  });
  
  // 保存历史记录
  f[i] = {
    cost,
    parsValue
  };
}

console.log(f);
console.log(`最少需要${f[n].cost}张纸币, 可以凑出价值为${n}的纸币`);
console.log(`组合方案`);
let t = n;
for (let i = 0; i < f[n].cost; i++) {
  console.log(f[t].parsValue);
  t = t - f[t].parsValue;
}
```

结果展示：

```bash
{
  '0': { cost: 0 },
  '1': { cost: 1, parsValue: 1 },
  '2': { cost: 2, parsValue: 1 },
  '3': { cost: 3, parsValue: 1 },
  '4': { cost: 4, parsValue: 1 },
  '5': { cost: 1, parsValue: 5 },
  '6': { cost: 2, parsValue: 5 },
  '7': { cost: 3, parsValue: 5 },
  '8': { cost: 4, parsValue: 5 },
  '9': { cost: 5, parsValue: 5 },
  '10': { cost: 2, parsValue: 5 },
  '11': { cost: 1, parsValue: 11 },
  '12': { cost: 2, parsValue: 11 },
  '13': { cost: 3, parsValue: 11 },
  '14': { cost: 4, parsValue: 11 },
  '15': { cost: 3, parsValue: 5 }
}
最少需要3张纸币, 可以凑出价值为15的纸币
组合方案
5
5
5
```

## 习题：括号配对

给定几种括号："()"，“[]”, "{}"，有以下两个问题：

1. 验证字符串内的括号是否缺失：

> 思路：最简单的做法就是，记录左括号出现的顺序，当出现右括号时进行配对，配对成功，将左括号弹出队列，最后判定记录的左括号数量是否为 0

```javascript
function verifyString(str) {
  if (!str) {
    return true
  }
  
  // 设置配对
  let pairs = {
    ')': '(',
    ']': '[',
    '}': '{'
  };
  // 设置符号
  let symbols = ['{', '}', '[', ']', '(', ')'];
  // 设置次序栈
  let pos = [];
  
  for (let i = 0; i < str.length; i++) {
    let s = str[i];
    
    // 发现左侧括号，加入次序栈
    if (symbols.indexOf(s) !== -1 && !pairs[s]) {
      pos.push(s);
      continue;
    }
  
    let r = pairs[s];
    if (!r) {
      continue;
    }
  
    // 发现右侧括号，判定有出现过括号
    if (pos.length === 0) {
      return false;
    }
    
    // 发现右侧括号，判定是否和上一个出现的括号一致
    if (pos.pop() !== r) {
      return false;
    }
  }
  
  return pos.length === 0;
}

console.log(verifyString('999999{[9(99]})'));
```

2. 清除掉有无效的括号（最近的不清除）：

> 思路：遍历一遍字符串，找到无效的括号的 key 并记录，然后逐个进行裁剪

```javascript
function clearString(str) {
  if (!str) {
    return true;
  }
  
  // 设置配对
  let pairs = {
    ')': '(',
    ']': '[',
    '}': '{'
  };
  // 设置栈, 记录位置
  let heap = {
    '(': [],
    ')': [],
    '{': [],
    '}': [],
    '[': [],
    ']': []
  };
  // 括号出现顺序
  let pos = [];
  
  for (let i = 0; i < str.length; i++) {
    let s = str[i];
    
    // 发现左侧括号，记录出现位置
    if (heap[s] && !pairs[s]) {
      heap[s].push(i);
      pos.push(s);
      continue;
    }
    
    let r = pairs[s];
    if (!r) {
      continue;
    }
    
    // 发现右侧括号，但左侧括号长度为 0，直接丢到无效的右括号中
    if (heap[r].length === 0) {
      heap[s].push(i);
      continue;
    }
    
    // 发现右侧括号，左侧括号长度大于 0，则抵消一个左侧括号，把最近的括号从数组底部弹出，
    let last = pos.pop();
    if (last === r) {
      heap[r].pop();
    } else {
      heap[s].push(i);
      pos.push(last);
    }
  }
  
  // 合并无效的索引，并根据索引排序
  let aux = [];
  for (let key of Object.keys(heap)) {
    aux = aux.concat(heap[key]);
  }
  aux.sort((a, b) => a - b);
  
  // 根据 heap 清除无效的括号
  let offset = 0;
  aux.forEach((i) => {
    str = str.substring(0, i - offset) + str.substring(i - offset + 1);
    offset++;
  });
  
  return str;
}

console.log(clearString('999(999{[999)]}')); // 999999{[999]}
```

## 习题：背包问题

解题前需要确定几点：

1. 同一物品是否可以拿复数？
2. 同一物品数量是否有上限？
3. 前两者条件都有。。。

作为算法苦手，先确定几种解法：

- 贪心策略：
  - 拿重量最轻的。
  - 拿价值最高的。
  - 拿价值密度最高的。
- ​	动态规划策略：
  - //TODO 分解子问题，决定策略。

## 习题：最长上升子序列

// TODO