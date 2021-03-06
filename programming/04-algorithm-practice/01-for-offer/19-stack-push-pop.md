# 栈的压入、弹出序列

## 题目

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）

## 题解

```js
function IsPopOrder(pushV, popV)
{
    let stack = [];
    const len = pushV.length;
    
    for(let i = 0; i < len; i++) {
        const res = operateStack(popV[i]);
        if (!res) return false;
    }
    
    return true;
    
    function operateStack(num) {
        if (num === pushV[0]) {
            pushV.shift();
            return true;
        } else if (stack.length && num === stack[stack.length - 1]) {
            stack.pop();
            return true;
        } else if (pushV.length) {
            stack.push(pushV.shift());
            return operateStack(num);
        }
        return false;
    }
}
```



## 参考链接

[牛客网](https://www.nowcoder.com/practice/d77d11405cc7470d82554cb392585106?tpId=13&&tqId=11174&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

