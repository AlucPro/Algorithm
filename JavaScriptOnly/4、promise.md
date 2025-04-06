# Promise 的手写实现

## 1. Promise 基础实现

### 定义

```javascript
new Promise((resolve, reject) => {
  // 异步操作
  resolve(value); // 成功时调用
  reject(reason); // 失败时调用
});
```

### 实现思路

1. 定义 Promise 的三种状态
2. 实现构造函数
3. 实现 then 方法
4. 处理异步和链式调用
5. 实现错误处理

### 代码实现

```javascript
class MyPromise {
  static PENDING = "pending";
  static FULFILLED = "fulfilled";
  static REJECTED = "rejected";

  constructor(executor) {
    this.status = MyPromise.PENDING;
    this.value = null;
    this.reason = null;
    this.onFulfilledCallbacks = [];
    this.onRejectedCallbacks = [];

    try {
      executor(this.resolve.bind(this), this.reject.bind(this));
    } catch (error) {
      this.reject(error);
    }
  }

  resolve(value) {
    if (this.status === MyPromise.PENDING) {
      this.status = MyPromise.FULFILLED;
      this.value = value;
      this.onFulfilledCallbacks.forEach((callback) => callback(value));
    }
  }

  reject(reason) {
    if (this.status === MyPromise.PENDING) {
      this.status = MyPromise.REJECTED;
      this.reason = reason;
      this.onRejectedCallbacks.forEach((callback) => callback(reason));
    }
  }

  then(onFulfilled, onRejected) {
    onFulfilled =
      typeof onFulfilled === "function" ? onFulfilled : (value) => value;
    onRejected =
      typeof onRejected === "function"
        ? onRejected
        : (reason) => {
            throw reason;
          };

    const promise2 = new MyPromise((resolve, reject) => {
      if (this.status === MyPromise.FULFILLED) {
        setTimeout(() => {
          try {
            const x = onFulfilled(this.value);
            this.resolvePromise(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
        });
      } else if (this.status === MyPromise.REJECTED) {
        setTimeout(() => {
          try {
            const x = onRejected(this.reason);
            this.resolvePromise(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
        });
      } else {
        this.onFulfilledCallbacks.push(() => {
          setTimeout(() => {
            try {
              const x = onFulfilled(this.value);
              this.resolvePromise(promise2, x, resolve, reject);
            } catch (error) {
              reject(error);
            }
          });
        });
        this.onRejectedCallbacks.push(() => {
          setTimeout(() => {
            try {
              const x = onRejected(this.reason);
              this.resolvePromise(promise2, x, resolve, reject);
            } catch (error) {
              reject(error);
            }
          });
        });
      }
    });

    return promise2;
  }

  resolvePromise(promise2, x, resolve, reject) {
    if (promise2 === x) {
      return reject(new TypeError("Chaining cycle detected for promise"));
    }

    if (x instanceof MyPromise) {
      x.then(resolve, reject);
    } else {
      resolve(x);
    }
  }
}
```

### 使用示例

```javascript
const promise = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve("success");
  }, 1000);
});

promise.then(
  (value) => console.log(value), // 输出：success
  (reason) => console.error(reason)
);
```

## 2. Promise 静态方法实现

### Promise.resolve

```javascript
static resolve(value) {
  if (value instanceof MyPromise) {
    return value;
  }
  return new MyPromise(resolve => resolve(value));
}
```

### Promise.reject

```javascript
static reject(reason) {
  return new MyPromise((_, reject) => reject(reason));
}
```

### Promise.all

```javascript
static all(promises) {
  return new MyPromise((resolve, reject) => {
    const results = [];
    let count = 0;

    promises.forEach((promise, index) => {
      MyPromise.resolve(promise).then(
        value => {
          results[index] = value;
          count++;
          if (count === promises.length) {
            resolve(results);
          }
        },
        reason => reject(reason)
      );
    });
  });
}
```

### Promise.race

```javascript
static race(promises) {
  return new MyPromise((resolve, reject) => {
    promises.forEach(promise => {
      MyPromise.resolve(promise).then(resolve, reject);
    });
  });
}
```

## 3. Promise 实例方法实现

### catch

```javascript
catch(onRejected) {
  return this.then(null, onRejected);
}
```

### finally

```javascript
finally(callback) {
  return this.then(
    value => MyPromise.resolve(callback()).then(() => value),
    reason => MyPromise.resolve(callback()).then(() => { throw reason })
  );
}
```

## 4. 可取消的 Promise

### 实现思路

1. 添加取消方法
2. 维护取消状态
3. 在适当的时候中断执行

### 代码实现

```javascript
class CancelablePromise extends MyPromise {
  constructor(executor) {
    let _reject;
    super((resolve, reject) => {
      _reject = reject;
      executor(resolve, reject);
    });
    this._reject = _reject;
  }

  cancel(reason = "Promise cancelled") {
    this._reject(new Error(reason));
  }
}
```

### 使用示例

```javascript
const promise = new CancelablePromise((resolve, reject) => {
  setTimeout(() => {
    resolve("success");
  }, 1000);
});

promise.then(
  (value) => console.log(value),
  (reason) => console.error(reason)
);

// 取消 Promise
promise.cancel();
```

## 5. 控制并发请求

### 实现思路

1. 维护请求队列
2. 控制并发数量
3. 按顺序处理请求

### 代码实现

```javascript
class RequestPool {
  constructor(maxConcurrent = 3) {
    this.maxConcurrent = maxConcurrent;
    this.queue = [];
    this.running = 0;
  }

  add(request) {
    return new MyPromise((resolve, reject) => {
      this.queue.push({ request, resolve, reject });
      this.run();
    });
  }

  run() {
    while (this.running < this.maxConcurrent && this.queue.length) {
      const { request, resolve, reject } = this.queue.shift();
      this.running++;

      request()
        .then(resolve)
        .catch(reject)
        .finally(() => {
          this.running--;
          this.run();
        });
    }
  }
}
```

### 使用示例

```javascript
const pool = new RequestPool(2);

// 模拟请求
const createRequest = (id, time) => () =>
  new MyPromise((resolve) =>
    setTimeout(() => {
      console.log(`Request ${id} completed`);
      resolve(id);
    }, time)
  );

// 添加请求
for (let i = 0; i < 5; i++) {
  pool.add(createRequest(i, 1000));
}
```

## 6. Axios 的 Promise 化

### 实现思路

1. 封装请求方法
2. 返回 Promise 对象
3. 处理请求和响应

### 代码实现

```javascript
function promisifyAxios(config) {
  return new MyPromise((resolve, reject) => {
    axios(config)
      .then((response) => resolve(response.data))
      .catch((error) => reject(error));
  });
}
```

### 使用示例

```javascript
// 使用封装的请求方法
promisifyAxios({
  method: "get",
  url: "https://api.example.com/data",
})
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```
