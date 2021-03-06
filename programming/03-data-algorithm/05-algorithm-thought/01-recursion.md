# 递归

递归是一种应用非常广泛的算法（或者编程技巧）。很多数据结构和算法的编码实现都要用到递归，比如 DFS 深度优先搜索、前中后序二叉树遍历等等。  

## 递归算法

### 递归需要满足的条件

1. 问题可以拆解为几个子问题的解。
2. 问题和分解之后的子问题，除了数据规模不同，求解思路完全一样。（递推公式）
3. 存在递归终止条件。

### 递归编写思路

1. 写出递推公式
2. 找到终止条件

### 走台阶问题

假如这里有 n 个台阶，每次你可以跨 1 个台阶或者 2 个台阶，请问走这 n 个台阶有多少种走法？

思考下 ^_^  

递推公式：

```
f(n) = f(n-1) + f(n-2)
```

终止条件：

```
f(1) = 1; f(2) = 2;
```

转化代码：

```js
function stairSolution(n) {
    if (n === 1) return 1;
    if (n === 2) return 2;
    return stairSolution(n - 1) + stairSolution(n - 2);
}
```

总结：写递归代码的关键就是找到如何将大问题分解为小问题的规律，并且基于此写出递推公式，然后再推敲终止条件，最后将递推公式和终止条件翻译成代码。  

### 常见问题

#### 堆栈溢出

递归调用过多，层级太深，会造成堆栈溢出。

解决方案：限制递归深度，治标不治本。

#### 重复计算

在递归调用时，可能会出现重复计算，问题拆解为多个子问题，子问题重叠，计算重复。

解决方案：可以通过一个表来存储已经求解过的值，防止重复求解。

走台阶问题解法优化：

```js
let stairMap = {};

function stairSolution2(n) {
    if (n === 1) return 1;
    if (n === 2) return 2;

    if (stairMap[n] === undefined) {
        stairMap[n] = stairSolution2(n - 1) + stairSolution2(n - 2);
    }

    return stairMap[n];
}
```

#### 无限递归

如果数据库里存在脏数据，造成递归调用成环，会产生无限递归问题。

解决方案：

1. 限制递归深度
2. 检测环的存在

#### 其他开销

在时间效率上，递归代码里多了很多函数调用，当这些函数调用的数量较大时，就会积聚成一个可观的时间成本。在空间复杂度上，因为递归调用一次就会在内存栈中保存一次现场数据，所以在分析递归代码空间复杂度时，需要额外考虑这部分的开销。

总结：

递归有利有弊，利是递归代码的表达力很强，写起来非常简洁；而弊就是空间复杂度高、有堆栈溢出的风险、存在重复计算、过多的函数调用会耗时较多等问题。  

### 改造为非递归代码

递归代码可以通过迭代循环的方式改造为非递归代码。

走台阶问题改造如下：

```js
function stairSolution(n) {
    if (n === 1) return 1;
    if (n === 2) return 2;

    let result;
    let pre = 2;
    let prepre = 1;

    for (let i = 3; i <= n; i++) {
        result = pre + prepre;
        prepre = pre;
        pre = result;
    }

    return result;
}
```

是不是所有的递归代码都可以改为这种**迭代循环**的非递归写法呢？  

笼统地讲，是的。因为递归本身就是借助栈来实现的，如果我们自己在内存堆上实现栈，手动模拟入栈、出栈过程，这样任何递归代码都可以改写成看上去不是递归代码的样子。  

但是这种思路实际上是将递归改为了“手动”递归，本质并没有变，而且也并没有解决前面讲到的某些问题，徒增了实现的复杂度。  



## 参考链接

[数据结构与算法之美 - 极客时间](https://time.geekbang.org/column/intro/100017301)

