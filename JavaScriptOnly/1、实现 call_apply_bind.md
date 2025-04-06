# 手写实现 call、apply、bind 方法

## 1. call 方法

### 定义

```javascript
foo.call(this, ...args);
```

- 立即执行函数
- 参数逐个传入

### 实现思路

1. 将函数作为传入对象的属性
2. 使用 Symbol 创建唯一键名，避免属性冲突
3. 执行函数并保存返回值
4. 删除临时属性
5. 返回函数执行结果

### 代码实现

```javascript
Function.prototype._call = function (context, ...args) {
  // 1. 处理 context
  const ctx = context || window;

  // 2. 创建唯一键名
  const key = Symbol("k");

  // 3. 将函数作为 context 的属性
  ctx[key] = this;

  // 4. 执行函数并保存结果
  const ret = ctx[key](...args);

  // 5. 删除临时属性
  delete ctx[key];

  // 6. 返回结果
  return ret;
};
```

### 使用示例

```javascript
function foo(a, b) {
  console.log(this.name);
  console.log(a + b);
}

// call 使用示例
foo._call({ name: "jxm" }, 1, 2);
// 输出：
// jxm
// 3
```

## 2. apply 方法

### 定义

```javascript
foo.apply(this, args);
```

- 立即执行函数
- 参数以数组形式传入

### 实现思路

1. 将函数作为传入对象的属性
2. 使用 Symbol 创建唯一键名，避免属性冲突
3. 执行函数并保存返回值
4. 删除临时属性
5. 返回函数执行结果

### 代码实现

```javascript
Function.prototype._apply = function (context, args) {
  // 1. 处理 context
  const ctx = context || window;

  // 2. 创建唯一键名
  const key = Symbol("k");

  // 3. 将函数作为 context 的属性
  ctx[key] = this;

  // 4. 执行函数并保存结果
  const ret = ctx[key](...args);

  // 5. 删除临时属性
  delete ctx[key];

  // 6. 返回结果
  return ret;
};
```

### 使用示例

```javascript
function foo(a, b) {
  console.log(this.name);
  console.log(a + b);
}

// apply 使用示例
foo._apply({ name: "jxm" }, [1, 2]);
// 输出：
// jxm
// 3
```

## 3. bind 方法

### 定义

```javascript
foo.bind(this, ...args)(...args);
```

- 返回一个新函数
- 可以分两次传入参数
- 支持 new 操作符

### 实现思路

1. 返回一个新函数
2. 处理参数合并
3. 处理 new 操作符的情况
   - 当作为构造函数调用时，this 指向实例
   - 否则使用传入的 context

### 代码实现

```javascript
Function.prototype._bind = function (context, ...args) {
  // 1. 保存原函数
  const fn = this;

  // 2. 处理 context
  const ctx = context || window;

  // 3. 返回新函数
  return function newFn(...args2) {
    // 4. 判断是否作为构造函数调用
    if (this instanceof newFn) {
      // 5. 作为构造函数调用时，this 指向实例
      return new fn(...args, ...args2);
    }

    // 6. 普通调用时，使用传入的 context
    return fn.apply(ctx, [...args, ...args2]);
  };
};
```

### 使用示例

```javascript
function foo(a, b) {
  console.log(this.name);
  console.log(a + b);
}

// bind 使用示例
const boundFoo = foo._bind({ name: "jxm" }, 1);
boundFoo(2);
// 输出：
// jxm
// 3
```
