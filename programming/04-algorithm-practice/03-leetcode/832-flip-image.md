# 翻转图像

## 题目

给定一个二进制矩阵 A，我们想先水平翻转图像，然后反转图像并返回结果。

水平翻转图片就是将图片的每一行都进行翻转，即逆序。例如，水平翻转 [1, 1, 0] 的结果是 [0, 1, 1]。

反转图片的意思是图片中的 0 全部被 1 替换， 1 全部被 0 替换。例如，反转 [0, 1, 1] 的结果是 [1, 0, 0]。

示例 1：

```
输入：[[1,1,0],[1,0,1],[0,0,0]]
输出：[[1,0,0],[0,1,0],[1,1,1]]
解释：首先翻转每一行: [[0,1,1],[1,0,1],[0,0,0]]；
     然后反转图片: [[1,0,0],[0,1,0],[1,1,1]]
```


示例 2：

```
输入：[[1,1,0,0],[1,0,0,1],[0,1,1,1],[1,0,1,0]]
输出：[[1,1,0,0],[0,1,1,0],[0,0,0,1],[1,0,1,0]]
解释：首先翻转每一行: [[0,0,1,1],[1,0,0,1],[1,1,1,0],[0,1,0,1]]；
     然后反转图片: [[1,1,0,0],[0,1,1,0],[0,0,0,1],[1,0,1,0]]
```


提示：

- 1 <= A.length = A[0].length <= 20
- 0 <= A[i][j] <= 1



## 题解

### 题解一

遍历矩阵数组，进行翻转和反转的操作。

```js
/**
 * @param {number[][]} A
 * @return {number[][]}
 */
var flipAndInvertImage = function(A) {
    const n = A.length;

    for(let i = 0; i < n; i++) {
        const mid = Math.floor(n / 2);
        // 翻转+反转
        for(let j = 0; j < mid; j++) {
            const tmp = A[i][j];
            A[i][j] = A[i][n - j - 1] ? 0 : 1;
            A[i][n - j - 1] = tmp ? 0 : 1;
        }
        if (n % 2 === 1) {
            A[i][mid] = A[i][mid] ? 0 : 1;
        }
    }

    return A;
};
```

复杂度：

- 时间复杂度：O(n^2)
- 空间复杂度：O(1)

#### 改进

优化为异或运算，添加右指针

```js
var flipAndInvertImage = function(A) {
    const n = A.length;

    for(let i = 0; i < n; i++) {
        let left = 0;
        let right = n - 1;

        while(left < right) {
            const tmp = A[i][left];
            A[i][left] = A[i][right] ^ 1;
            A[i][right] = tmp ^ 1;
            left++;
            right--;
        }

        if (left === right) {
            A[i][left] ^= 1;
        }
    }

    return A;
};
```

#### 改进！

有个小细节，当两个翻转的数不相等时，保留原状即可，不用翻转+反转操作了！

例：[1, 0, ... 1, 0] -> [0, 1, ... 0, 1] -> [1, 0, ... 1, 0]

```js
var flipAndInvertImage = function(A) {
    const n = A.length;

    for(let i = 0; i < n; i++) {
        let left = 0;
        let right = n - 1;

        while(left < right) {
            if(A[i][left] === A[i][right]) {
                A[i][left] ^= 1;
                A[i][right] ^= 1;
            }
            left++;
            right--;
        }

        if (left === right) {
            A[i][left] ^= 1;
        }
    }

    return A;
};
```



## 参考链接

[832. 翻转图像](https://leetcode-cn.com/problems/flipping-an-image/)

