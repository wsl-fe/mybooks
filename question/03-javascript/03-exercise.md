# 习题

## EventLoop

### 1. 请输出下面代码打印情况

#### 1) promise, setTimeout

```javascript
setTimeout(function(){
    console.log(1)
},0);

new Promise(function(resolve){
    console.log(2)
    
    for( var i=0 ; i<10000 ; i++ ){
        i==9999 && resolve()
    }
    
    console.log(3)
}).then(function(){
    console.log(4)
});
console.log(5);

//2 3 5 4 1
```

打印：2 3 5 4 1

执行过程大概如下：

- 由于整个 script 也属于一个  macrotask, 所以整个script被推进 macrotask queue，由于会先执行 macrotask  queue 中的第一个任务，再加上promise 构造函数因为是同步的，所以会先打印出2和3
- 然后继续同步执行末尾的 console.log(5) 打印出5
- 此时 setTimeout 被推进到  macrotask queue中， promise.then 回调被推进到 microtask queue 中
- 由于在第一步中已经执行完了第一个 macrotask ,  所以接下来会顺序执行所有的 microtask, 也就是 promise.then 的回调函数，从而打印出4
- microtask 队列中的任务已经执行完毕，继续执行剩下的  macrotask 队列中的任务，也就是 setTimeout, 所以打印出 1

#### 2) promise, setTimeout

```js
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');
```

打印：

```
script start
script end
promise1
promise2
setTimeout
```

过程：

- 先执行同步代码，输出：`script start` -> `script end`。
- 然后调用微任务，输出 `promise1`，将 `then(promise2)` 放入微任务。
- 再次调用微任务，将 `promise2` 输出。
- 最后调用宏任务 `setTimeout`，输出 `setTimeout`。

#### 3) promise, setTimeout

```js
setTimeout(function() {
  console.log(4);
}, 0);

const promise = new Promise((resolve) => {
  console.log(1);
  for (var i = 0; i < 10000; i++) {
    i == 9999 && resolve();
  }
  console.log(2);
}).then(function() {
  console.log(5);
});

console.log(3);
```

打印：

```
1 2 3 5 4
```

分析：

`script` 下：

- 同步任务：`console.log(1)`、`console.log(2)`、`console.log(3)`。
- 微任务：`Promise.then()`（等到 9999 再添加进来）
- 宏任务 `setTimeout`

所以先走同步任务，注意当我们 `new Promsie()` 的时候，内部的代码会执行的，跟同步任务一样的，而 `.then()` 在 `resolve()` 的情况下才会添加到微任务。

因此先输出 `1 -> 2 -> 3`。

然后推出微任务 `Promise.then()`，所以输出 5。

最后推出宏任务 `setTimeout`，输出 4。

#### 4) promise, setTimeout

```js
setTimeout(function () {
  console.log('timeout1');
}, 1000);

console.log('start');

Promise.resolve().then(function () {
  console.log('promise1');
  Promise.resolve().then(function () {
    console.log('promise2');
  });
  setTimeout(function () {
    Promise.resolve().then(function () {
      console.log('promise3');
    });
    console.log('timeout2')
  }, 0);
});

console.log('done');
```

打印：

```
start
done
promise1
promise2
timeout2
promise3
timeout1
```

#### 5) promise, setTimeout

```js
console.log("script start");

setTimeout(function() {
  console.log("setTimeout---0");
}, 0);

setTimeout(function() {
  console.log("setTimeout---200");
  setTimeout(function() {
    console.log("inner-setTimeout---0");
  });
  Promise.resolve().then(function() {
    console.log("promise5");
  });
}, 200);

Promise.resolve()
.then(function() {
  console.log("promise1");
})
.then(function() {
  console.log("promise2");
});

Promise.resolve().then(function() {
  console.log("promise3");
});

console.log("script end");
```

打印：

```
script start
script end
promise1
promise3
promise2
setTimeout---0
setTimeout---200
promise5
inner-setTimeout---0
```

#### 6) promise, setTimeout

```js
console.log(1);

setTimeout(() => {
  console.log(2);

  new Promise((resolve) => {
    console.log(3);
  }).then(() => {
    console.log(4);
  });
}, 200);

new Promise((resolve) => {
  console.log(5);
  resolve();
}).then(() => {
  console.log(6);
});

setTimeout(() => {
  console.log(7);
}, 0);

setTimeout(() => {
  console.log(8);

  new Promise(function (resolve) {
    console.log(9);
    resolve();
  }).then(() => {
    console.log(10);
  });
}, 100);

new Promise(function (resolve) {
  console.log(11);
  resolve();
}).then(() => {
  console.log(12);
});

console.log(13);
```

打印：

```
1 5 11 13 6 12 7 8 9 10 2 3
```

#### 7) for, setTimeout

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 0);
}
```

打印：

```
3 3 3
```

分析：

1. 走同步任务，`var i` 遍历走完，`i = 3`（`var` 变量污染）。
2. `for()` 遍历的同时，将 3 个 `setTimeout` 塞进了宏任务中。
3. `script` 这个宏任务执行完毕。
4. 依次执行 3 个 `setTimeout`，因为此时 `i` 为 `3`，所以会依次输出 3 个 3。


### 2. 请输出下面代码打印情况

#### 1) promise

```js
const promise1 = new Promise((resolve, reject) => {
  console.log('promise1');
  resolve('resolve1');
})

const promise2 = promise1.then((res) => {
  console.log(res);
});

console.log('1', promise1);
console.log('2', promise2);
```

打印：

```
promise1
1 Promise {<fullfilled>: "resolve1"}
2 Promise {<pending>}
resolve1
```

分析：

1. 碰到 new Promise，输出 promise1
2. 碰到 resolve，改变 Promise 状态，并保存结果
3. 碰到 promise1.then，放进微任务队列
4. promise2 是一个新的状态为 pending 的 Promise
5. 输出 1 和 promise1，当前 promise1 的状态`[[PromiseState]]`为 fullfilled，结果`[[PromiseResult]]`为 resolve1
6. 输出 2 和 promise2，当前 promise2 的状态为 peding
7. 宏任务走完，执行微任务，输出 resolve1

#### 2) promise, setTimeout

```js
console.log('start');

setTimeout(() => {
  console.log('time');
});

Promise.resolve().then(() => {
  console.log('resolve');
});

console.log('end');
```

打印：

```
start
end
resolve
time
```

分析：

  1. 首先执行 script
  2. 输出 'start'
  3. 碰到 setTimeout，丢进宏任务队列
  4. 碰到 Promise，然后 Promise 变成 resolve() 状态后，执行了 .then()，所以丢进微任务队列
  5. 输出 'end'
  6. 遍历本次的微任务队列，输出步骤 4 的内容，即 'resolve'
  7. 步骤 6 走完，执行下一个宏任务队列，输出 'time'

#### 3) promise, setTimeout

```js
const promise = new Promise((resolve, reject) => {
  console.log(1);
  
  setTimeout(() => {
    console.log('timerStart');
    resolve('success');
    console.log('timerEnd');
  }, 0);

  console.log(2);
});

promise.then((res) => {
  console.log(res);
});

console.log(4);
```

打印：

```
1
2
4
timerStart
timerEnd
success
```

分析：

1. 首先执行 script 这个宏任务
2. 碰到 new Promise，输出 1
3. 碰到 setTimeout，放进宏任务队列
4. 输出 2
5. 碰到 .then()，但是没有钥匙（resolve），跳过
6. 输出 4
7. 当前没有微任务，执行下一个宏任务 setTimeout
8. 输出 'timerStart'
9. Promise 碰到 resolve，改变状态，表明 .then() 可以放进微任务了
10. 输出 'timerEnd'
11. 执行宏任务 setTimeout 下的微任务，即 Promise.then()
12. 输出 'success'

#### 4) promise, setTimeout

```js
Promise.resolve().then(() => {
  console.log('promise1');
  const timer2 = setTimeout(() => {
    console.log('timer2');
  }, 0);
});

const timer1 = setTimeout(() => {
  console.log('timer1');
  Promise.resolve().then(() => {
    console.log('promise2');
  });
}, 0);

console.log('start');
```

打印：

```
start
promise1
timer1
promise2
timer2
```

分析：

1. 首先执行 script 这个宏任务
3. 碰到 Promise.then()，将其推进微任务队列，注意不会执行里面内容
4. 碰到 timer1，将其推进宏任务队列
5. 输出 'start'
6. 查看 script 中的微任务队列，发现 Promise.then()，将其推出执行
7. 输出 'promise1'
8. 碰到 timer2，将其推进宏任务队列
9. script 没有剩余的微任务，所以继续遍历宏任务
10. 发现队列 [timer1, timer2]，根据队列先进先出原则，推出 timer1
11. 输出 'timer1'，发现微任务 Promise.then()，将其推进 timer1 的微任务队列
12. 输出 'promise2'
13. 继续执行宏任务队列，出队 timer2，输出 'timer2'

#### 5) promise, setTimeout

```js
const promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('success');
  }, 0);
});

const promise2 = promise1.then(() => {
  throw new Error('error!');
});

console.log('promise1-1', promise1);
console.log('promise2-1', promise2);

setTimeout(() => {
  console.log('promise1-2', promise1);
  console.log('promise2-2', promise2);
}, 0);
```

打印：

```
promise1-1 Promise { <pending> }
promise2-1 Promise { <pending> }
promise1-2 Promise { <fullfilled>: 'success' }
promise2-2 Promise { <rejected>: Error: error! }
```

> 注意：在 Node v10.16.0 上运行结果不太相同

分析：

1. 首先执行 script 这个宏任务
3. 碰到 promise1 这边，执行 new Promise 里面内容，将带 resolve 的 setTimeout 推入宏任务队列
4. 碰到 promise2，因为还没有进入 resolve 状态，所以这里不理会
5. 连续两行输出，因为 promise1 和 promise2 都尚未处理，所以是 pending 状态
6. 碰到第二个 setTimeout，将其推入宏任务队列
7. 查看宏任务队列，推出第一个 setTimeout，将 Promise 状态改为 resolve
8. 执行 promise2，改变 promise2 的状态为 reject
9. 第一个 setTimeout 执行完毕，执行第二个 setTimeout
10. 输出步骤 8 和 步骤 9 中的 Promise 状态

#### 6) promise链

```js
const promise = new Promise((resolve, reject) => {
  resolve('success1');
  reject('error');
  resolve('success2');
});

promise.then((res) => {
  console.log('then1: ', res);
}).then((res) => {
  console.log('then2: ', res);
}).catch((error) => {
  console.log('catch: ', error);
});
```

打印：

```
then1: success1
then2: undefined
```

分析：

1. 执行了 resolve('success1') 后，改变了状态为 resolve，不再理会 new Promise 后面的
2. 将第 1 个 .then() 添加到微任务
3. 执行第 1 个 .then()，将第 2 个 .then() 推进微任务

#### 7) promise链

```js
const promise = new Promise((resolve, reject) => {
  reject('error');
  resolve('success2');
});

promise.then((res) => {
  console.log('then1: ', res);
}).then((res) => {
  console.log('then2: ' ,res);
}).catch((error) => {
  console.log('catch: ', error);
}).then((res) => {
  console.log('then3: ', res);
})

/**
  执行顺序和分析：
  顺序：
    * 'catch:  error'
    * 'then3:  undefined'
  分析：
    1. 碰到 new Promise()，将 reject('error') 执行，改变 Promise 的状态
    2. 碰到 .catch()，将其推进微任务
    3. 执行 .catch() 里面内容，输出 'catch: error'，然后 return Promise {<pending>}
    4. 执行下一个微任务 .then()，输出 then3: undefined
*/
```

#### 8) promise链

```js
Promise
.reject(1)
.then((res) => {
  console.log(res);
  return 2;
}).catch((err) => {
  console.log(err);
  return 3;
}).then((res) => {
  console.log(res);
});

/**
  执行顺序和分析：
  顺序：
    * 1
    * 3
  分析：
    1. reject(1) 会走 .catch，所以先输出 1
    2. return 3 会被包装成 resolve(3)
    3. 所以继续走第 2 个 .then，输出 3
*/
```

#### 9) promise.then 多个

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('timer');
    resolve('success');
  }, 0);
});

const start = Date.now();

promise.then((res) => {
  console.log(res, Date.now() - start);
});

promise.then((res) => {
  console.log(res, Date.now() - start);
});

/**
  执行顺序和分析：
  顺序：
    * 'timer'
    * 'success 4'
    * 'success 4'
  注释：也有 3/4 或者 4/5 的情况
  分析：
    1. new Promise 将 setTimeout 添加进宏任务
    2. 执行完宏任务 script，然后就执行 setTimeout
    3. 输出 'timer'
    4. 标记 Promise 状态为 resolve
    5. 将第一个 .then() 放进微任务
    6. 将第二个 .then() 放进微任务
    7. 因为步骤 5 和步骤 6 的时候，这两者都是相同 resolve 值，所以都是 'success'
    8. 输出 success 4
    9. 输出 success 4
    10. 如果执行比较慢，那么这两个输出的值会不一致。例如 3、4
*/
```

#### 10) return promise

```js
const promise = Promise.resolve().then(() => {
  return promise;
});

promise.catch((err) => {
  console.log(err);
});

/**
  执行顺序和分析：
  顺序：
    * TypeError: Chaining cycle detected for promise #<Promise>
  分析：
    不能返回 promise 本身，会造成死循环
*/
```

#### 11) promise 链

```js
Promise
.resolve(1)
.then(2)
.then(Promise.resolve(3))
.then(console.log);

/**
  执行顺序和分析：
  顺序：
    * 1
  分析：
    1. .then 和 .catch 的参数希望是函数，传入非函数会发生值透传
    2. 值透传导致第 1 个 then 和第 2 个 then 传入的都不是函数，导致它传到最后的 1 个 then 里面
*/
```

#### 12) promise链

```js
Promise
.reject('err')
.then((res) => {
  console.log('success: ', res);
}, (err) => {
  console.log('error: ', err);
}).catch((err) => {
  console.log('catch: ', err);
})

/**
  执行顺序和分析：
  顺序：
    * 'error:  err'
  分析：
    reject('err') 会进入 Promise.then 的第二个参数，所以输出 'error: err'
*/
```

如果本题中的 `.then()` 中的第 2 个参数去掉了，那么就会进入 `.catch()` 函数中。

#### 13) promise链

```js
romise
.resolve()
.then((res) => {
  throw new Error('error!');
}, (err) => {
  console.log('error: ', err);
}).catch((err) => {
  console.log('catch: ', err);
})

/**
  执行顺序和分析：
  顺序：
    * catch:  Error: error!
  分析：
    因为是 .resolve()，所以会执行 .then 第 1 个参数，然后 return 的值到 .catch 中
    而不是返回第 2 个参数上
*/
```

#### 14) .finally

```js
Promise
.resolve('1')
.then((res) => {
  console.log(res);
}).finally(() => {
  console.log('finally1');
});

Promise
.resolve('2')
.finally(() => {
  console.log('finally2');
  return '这里是 finally2';
}).then((res) => {
  console.log('finally2 后面的 then 函数', res);
})
```

打印：

```
1
finally2
finally1
finally2 后面的 then 函数 2
```

#### 15) .finally

```js
Promise
.resolve('1')
.finally(() => {
  console.log('finally1');
  return new Error('我是 finally1 中抛出的异常');
}).then((res) => {
  console.log('finally 后面的 then 函数: ', res);
}).catch((err) => {
  console.log('捕获错误: ', err);
})

/**
  执行顺序和分析：
  顺序：
    * 'finally1'
    * finally 后面的 then 函数: 1
*/
```

#### 16) Promise.all, Promise.race

```js
const one = new Promise((resolve) => {
  setTimeout(() => {
    console.log('one');
    resolve('one resolve');
  }, 1000);
}) 
const two = new Promise((resolve) => {
  setTimeout(() => {
    console.log('two');
    resolve('two resolve');
  }, 3000);
}) 
const three = new Promise((resolve) => {
  setTimeout(() => {
    console.log('three');
    resolve('three resolve');
  }, 2000);
}) 

Promise.all([one, two, three]).then((res) => {
  console.log(res);
});

Promise.race([one, two, three]).then((res) => {
  console.log(res); // 'one'
});

/*
 先输出 one
 再输出：one resolve
 最后依序输出：
 * three
 * two
 * ["one resolve", "two resolve", "three resolve"]
*/
```

#### 17) Promise.all

```js
function runAsync(x) {
  const p = new Promise((resolved, reject) => {
    if (x % 2 === 0) {
      return setTimeout(() => {
        console.log(x);
        resolved(x);
      }, 2000);
    }
    return setTimeout(() => {
      console.log(x);
      resolved(x);  
    }, 1000);
  });
  return p;
}

Promise.all([
  runAsync(1),
  runAsync(2),
  runAsync(3)
]).then((res) => {
  console.log(res);
})

/**
  执行顺序和分析：
  顺序：
    * 1
    * 3
    * 2
    * [1, 2, 3]
  分析：
    1. Promise.all 将 3 个 runAsync 按顺序添加进方法中
    2. 在 script 这个宏任务中，依次添加 3 个 setTimeout
    3. 根据时间对宏任务队列中的 setTimeout 进行重新排序
    4. 1、2、3 对应的秒数为 1s、2s、1s，所以排序为 1 -> 3 -> 2
    5. 等待一秒后，分别输出 1、3
    6. 等待两秒后，输出 2
    7. 执行 .then()，依照 .all() 中数组的排序输出对应的数组结果（怎么进来怎么出去）
  适用场景：
    需要预先加载多种图片、静态文件等，可以通过 Promise.all() 进行处理
*/
```

#### 18) Promise.all

```js
function runAsync (x) {
  const p = new Promise((res, rej) => {
    if (x === 3) {
      return setTimeout(() => {
        rej(x, console.log(x));
      }, 500);
    }
    return setTimeout(() => {
     res(x, console.log(x)); 
    }, 1000);
  });
  return p;
}

function runReject (x) {
  const p = new Promise((res, rej) => {
    return setTimeout(() => {
      rej(x, console.log(x));
    }, 1000 * x);
  });
  return p;
}

Promise.all([
  runAsync(1),
  runReject(4),
  runAsync(3),
  runReject(2),
]).then((res) => {
  console.log('then: ', res);
}, (err) => {
  console.log('err: ', err);
}).catch((err) => {
  console.log('catch: ', err);
})

/**
  执行顺序和分析：
  顺序：
    * 3
    * err: 3
    * 1
    * 2
    * 4
  分析：
    1. 首先，我们当 .all() 是一个队列，先进先出
    2. 此时宏任务依次添加 setTimeout(1)、setTimeout(4)、setTimeout(3)、setTimeout(2)
    3. OK，我们在前面说过，相同 setTimeout 会被排序，所以顺序变为 3 -> 1 -> 2 -> 4
    4. 这时候的 setTimeout 对应的时间为 500ms、1s、2s、4s
    5. 然后，需要记住一点新特性：.catch 只能捕获 .all 里面最先的那个异常，并且只执行一次
    6. 所以，先执行 3 的时候，会依次输出 3 -> err: 3
    7. 后面的 2 和 4 的异常不再抛出，依次输出 1 -> 2 -> 4
*/
```

#### 19) Promise.race

```js
function runAsync(x) {
  const p = new Promise((resolved, reject) => {
    if (x % 2 === 0) {
      return setTimeout(() => {
        console.log(x);
        resolved(x);
      }, 2000);
    }
    return setTimeout(() => {
      console.log(x);
      resolved(x);  
    }, 1000);
  });
  return p;
}

Promise.race([
  runAsync(2),
  runAsync(1),
  runAsync(3)
]).then((res) => {
  console.log('res: ', res);
})

/**
  执行顺序和分析：
  顺序：
    * 1
    * 'res: 1'
    * 3
    * 2
  注意：
    Node v10.16.0 的答案略有不同
  分析：
    1. Promise.race() 将 3 个 runAsync 按顺序添加进方法中
    2. 在 script 这个宏任务中，依次添加 3 个 setTimeout 宏任务：2 -> 1 -> 3
    3. 根据时间对宏任务队列中的 setTimeout 进行重新排序
    4. 1、2、3 对应的秒数为 1s、2s、1s，所以排序为 1 -> 3 -> 2
    5. 等待一秒后，输出 1
    6. 此时 .race() 迫不及待得想告诉你结果，跟着输出 res: 1
    7. 紧接着输出 3
    8. 等待两秒后，输出 2
  适用场景：
    用 race 给某个异步请求设置超时时间，并且在超时后执行相应的操作
*/
```

#### 20) async

```js
async function async1() {
  console.log(1);
  await async2();
  console.log(2);
}

async function async2() {
  console.log(3);
}

async1();

console.log(4);

/**
  执行顺序和分析：
  顺序：
    * 1
    * 3
    * 4
    * 2
  分析：
    1. 首先，我们执行 script 这个宏任务
    2. 碰到 async1()，执行里面代码，输出 1
    3. 碰到 await async2()，阻塞了，所以需要先执行 async2()
    4. 执行 async2()，输出 3
    5. 碰到 console.log(4)，输出 4
    6. 阻塞部分走完了，script 这个宏任务也走完了，接着走 async1() 后面的
    7. 输出 2
*/
```

#### 21) async, setTimeout, promise

```js
async function async1() {
  console.log('async1 start');
  setTimeout(() => {
    console.log('timer1 start');
  }, 0);
  Promise.resolve().then((res) => {
    console.log('promise1');
  })
  await async2();
  setTimeout(() => {
    console.log('timer1 end');
  }, 0);
  console.log('async1 end');
}

async function async2() {
  setTimeout(() => {
    console.log('timer2');
  }, 0);
  Promise.resolve().then((res) => {
    console.log('promise2');
  })
  console.log('async2');
}

async1();

console.log('start');

/**
  执行顺序和分析：
  顺序：
    * 'async1 start'
    * 'async2'
    * 'start'
    * 'promise1'
    * 'promise2'
    * 'async1 end'
    * 'timer1 start'
    * 'timer2'
    * 'timer1 end'
  分析：
    1. 首先，我们理顺一个事实：在 await 后面的，会等当前宏任务里面所有微任务执行完毕，方且执行
    2. 碰到 async1()，开始执行里面内容
    3. 输出 'async1 start'
    4. 将 'timer1 start' 丢进宏任务队列，标记为宏 1
    5. 将 'promise1' 丢进微任务队列，标记为微 1
    6. 碰到 await async2()，先执行 async2，阻塞下面的代码，标记后面代码为马后炮 1
    7. 执行 async2，碰到 'timer2'，将其丢进宏任务队列，标记为宏 2
    8. 碰到 'promise2'，将其丢进微任务队列，标记为微 2
    9. 输出 'async2'
    10. async2 走完，继续往下走，输出 start
    11. 当前有 3 个部分我们没走，分别是微 1、微 2 和马后炮 1
    12. 【死记】，碰到不走 11 这种情况，我们需要记住先执行当前微任务，再马后炮
    13. 执行微任务，输出 'promise1'、'promise2'
    14. 执行马后炮，将 'timer1 end' 丢进宏任务队列，即为宏 3
    15. 输出 'async1 end'
    16. 依次执行宏 1、宏 2 和 宏 3，输出 'timer1 start' -> 'timer2' -> 'timer1 end'
  灵魂提升：
    如果 'timer1 start' -> 'timer2' -> 'timer1 end' 对应的时间分别为 500ms、1000ms、500ms，请问输出啥？
*/
```

#### 22) async

```js
async function fn() {
  return 123;
}

fn().then((res) => {
  console.log(res);
})

/**
  执行顺序和分析：
  顺序：
    * 123
  分析：
    正常情况下， async 中的 await 命令是一个 Promise 对象，返回该对象的结果
    但如果不是 Promise 对象的话，就会直接返回对应的值，相当于 Promise.resolve();
*/
```

#### 23) async, promise

```js
async function async1() {
  console.log('async1 start');
  await new Promise((resolve) => {
    console.log('promise1');
  })
  console.log('async1 success');
  return 'async1 end';
}

console.log('script start');

async1().then((res) => {
  console.log('res: ', res);
})

console.log('script end');

/**
  执行顺序和分析：
  顺序：
    * 'script start'
    * 'async1 start'
    * 'promise1'
    * 'script end'
  分析：
    1. 特殊题
    2. 在 await 后面的 Promise 是没有返回值的，所以 await 会一直等待
    3. 这样子的话，async1 success 这些后面的内容都不会执行了
  思考：
    如果在 'promise1' 后面添加一行 resolve('123'); 结果会怎样？
*/
```

#### 24) async, promise, setTimeout

```js
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}

async function async2() {
  console.log('async2');
}

console.log('script start');

setTimeout(() => {
  console.log('settimeout');
}, 0);

async1();

new Promise((resolve) => {
  console.log('promise1');
  resolve();
}).then((res) => {
  console.log('promise2');
})

console.log('script end');
```

打印：

```
'script start'
'async1 start'
'async2'
'promise1'
'script end'
'async1 end'
'promise2'
'settimeout'
```

#### 25) async, promise

```js
async function testSomething() {
  console.log('test something');
  return 'test something';
}

async function testAsync() {
  console.log('test async');
  return Promise.resolve('hello test async');
}

async function test() {
  console.log('test start');

  const v1 = await testSomething();
  console.log('v1: ', v1);

  const v2 = await testAsync();
  console.log('v2: ', v2);

  console.log(v1, v2);
}

test();

const promise = new Promise((resolve) => {
  console.log('promise start');
  resolve('promise');
})

promise.then((val) => {
  console.log(val);
})

console.log('test end');

/**
  执行顺序和分析：
  顺序：
    * 'test start'
    * 'test something'
    * 'promise start'
    * 'test end'
    * 'v1: test something'
    * 'test async'
    * 'promise'
    * 'v2: hello test async'
    * 'test something' 'hello test async'
*/
```

#### 26) async, promise

```js
async function async1() {
  await async2();
  console.log('async1');
  return 'async1 success';
}

async function async2() {
  return new Promise((resolve, reject) => {
    console.log('async2');
    reject('error');
  })
}

async1().then((res) => {
  console.log('res: ', res);
})

/**
  执行顺序和分析：
  顺序：
    * 'async2'
    * Promise {<rejected>: "error"}
  分析：
    如果在 async 函数中抛出了错误，则终止错误结果，不会继续向下执行。throw new Error 也是如此。
*/
```

#### 27) promise, setTimeout

```js
const first = () => (new Promise((resolve1, reject1) => {
  console.log(3);
  
  const p = new Promise((resolve2, reject2) => {
    console.log(7);
    
    setTimeout(() => {
      console.log(5);
      resolve1(6);
      console.log(p);
    }, 0);

    resolve2(1);
  });

  resolve1(2);

  p.then((res1) => {
    console.log('res1: ', res1);
  });
}));

first().then((res2) => {
  console.log('res2: ', res2);
});

console.log(4);

/**
  执行顺序：
    * 3
    * 7
    * 4
    * res: 1
    * res: 2
    * 5
    * Promise {<fullfilled>: 1}
*/
```

#### 28) async, promise, setTimeout

```js
const async1 = async() => {
  console.log('async1');
  
  setTimeout(() => {
    console.log('timer1');
  }, 2000);

  await new Promise((resolve) => {
    console.log('promise1');
  })

  console.log('async1 end');
  return 'async1 success';
};

console.log('script start');

async1().then((res1) => {
  console.log('res1: ', res1);
})

console.log('script end');

Promise
.resolve(1)
.then(2)
.then(Promise.resolve(3))
.catch(4)
.then((res2) => {
  console.log('res2: ', res2);
})

setTimeout(() => {
  console.log('timer2');
}, 1000);

/**
  执行顺序：
    * 'script start'
    * 'async1'
    * 'promise1'
    * 'script end'
    * 'res2: 1'
    * 'timer2'
    * 'timer1'
*/
```

#### 29) promise, .finally

```js
const p1 = new Promise((resolve) => {
  setTimeout(() => {
    resolve('resolve3');
    console.log('timer1');
  }, 0);
  resolve('resolve1');
  resolve('resolve2');
}).then((res) => {
  console.log(res);
  setTimeout(() => {
    console.log(p1);
  }, 1000);
}).finally((res) => {
  console.log('finally: ', res);
})

/**
  执行顺序：
    * 'resolve1'
    * 'finally: undefined'
    * 'timer1'
    * 'Promise {<fullfilled>: undefined}'
*/
```

