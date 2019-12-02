算法例题
=========================

> 记录面试中的一些问题
>

### 习题：八皇后问题（回溯法）

> 在**8**×**8**格的国际象棋上摆放**八**个**皇后**，使其不能互相攻击，即任意两个**皇后**都不能处于同一行、同一列或同一斜线上，问有多少种摆法。

```javascript
function NQueen(pos, n) {
  // 满足数量，打印
  if (n === pos.length) {
    console.log(pos);
    return;
  }
  
  // 遍历每一列，为 post[n] 查找可摆放的列(i)
  for (let i = 0; i < 8; i++) {
    // 尝试将 post[n] 摆放到 i 的位置
    pos[n] = i;
  
    // 开始确定 0 ～ n 列，n 表示已经确定皇后位置的列数（j < n, j++ 就是往回回溯已经确定位置的）
    let j = 0;
    while (j < n || n === 0) {
      // 当只确定第1个皇后的位置，开始遍历查找下一个皇后可摆放的位置。
      if (n === 0) {
        NQueen(pos, n + 1);
        break;
      }
      // 已确定皇后的列（post[j]）和当前列（i）相等（重合），判定同列失败退出
      if (pos[j] === i) {
        break;
      }
      // 已确定皇后的列 (post[j]) 与当前列（i）相差为 1 的情况下列数相减不可为 1（n - j），判定为斜角失败
      if (Math.abs(pos[j] - i) === n - j) {
        break
      }
      // 正常通过遍历，继续查找下一个皇后可摆放的位置。
      if (j === n - 1) {
        NQueen(pos, n + 1);
      }
      
      j++;
    }
  }
}

NQueen([1, 1, 1, 1, 1, 1, 1, 1], 0);
```

## 习题：括号配对（堆栈）

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

## 习题：背包问题（动态规划）

解题前需要确定几点：

1. 同一物品是否可以拿复数？
2. 同一物品数量是否有上限？
3. 前两者条件都有



## 习题：最长上升子序列（动态规划）

// TODO