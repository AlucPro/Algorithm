# 数组扁平化、柯里化、函数组合的实现

## 1. 数组扁平化（flat）

### 定义

```javascript
function flat(arr) {
  // 返回扁平化后的数组
}
```

- 将嵌套数组转换为单层数组
- 支持任意深度的嵌套
- 保持元素顺序

### 实现思路

1. 递归版本：深度优先遍历
2. 迭代版本：广度优先遍历
3. 处理非数组元素
4. 保持元素顺序

### 代码实现

#### 递归版本

```javascript
function flat(arr) {
  const ret = [];

  arr.forEach((item) => {
    if (!Array.isArray(item)) {
      ret.push(item);
      return;
    }

    const items = flat(item);
    ret.push(...items);
  });

  return ret;
}
```

#### 迭代版本

```javascript
function flat2(arr) {
  const ret = [];
  const arrList = [...arr];

  while (arrList.length) {
    const item = arrList.pop();

    if (Array.isArray(item)) {
      arrList.push(...item);
      continue;
    }

    ret.unshift(item);
  }

  return ret;
}
```

### 使用示例

```javascript
const arr = [1, [2, 3], [[4, 5], 6]];

// 递归版本
console.log(flat(arr)); // [1, 2, 3, 4, 5, 6]

// 迭代版本
console.log(flat2(arr)); // [1, 2, 3, 4, 5, 6]
```

## 2. 柯里化（curry）

### 定义

```javascript
function curry(fn) {
  // 返回柯里化后的函数
}
```

- 将多参数函数转换为单参数函数
- 支持参数收集
- 当参数足够时自动执行

### 实现思路

1. 判断参数数量
2. 收集参数
3. 参数足够时执行原函数
4. 参数不足时返回新函数

### 代码实现

```javascript
function curry(fn, ...args) {
  // 参数足够时执行原函数
  if (args.length >= fn.length) {
    return fn(...args);
  }

  // 参数不足时返回新函数
  return function (...newArgs) {
    return curry(fn, ...args, ...newArgs);
  };
}
```

### 使用示例

```javascript
function multiply(a, b, c) {
  return a * b * c;
}

const curriedMultiply = curry(multiply);

console.log(curriedMultiply(2)(3)(4)); // 24
console.log(curriedMultiply(2, 3)(4)); // 24
console.log(curriedMultiply(2)(3, 4)); // 24
```

## 3. 函数组合（compose）

### 定义

```javascript
function compose(...fns) {
  // 返回组合后的函数
}
```

- 将多个函数组合成一个函数
- 从右到左执行
- 前一个函数的输出作为后一个函数的输入

### 实现思路

1. 接收多个函数作为参数
2. 从右到左依次执行
3. 传递参数和返回值

### 代码实现

```javascript
function compose(...fns) {
  return function (...args) {
    return fns.reduceRight((result, fn) => {
      return fn(result);
    }, args);
  };
}
```

### 使用示例

```javascript
const add = (x) => x + 1;
const multiply = (x) => x * 2;
const square = (x) => x * x;

const composed = compose(square, multiply, add);
console.log(composed(2)); // ((2 + 1) * 2) ^ 2 = 36
```

## 4. 管道函数（pipe）

### 定义

```javascript
function pipe(...fns) {
  // 返回管道函数
}
```

- 将多个函数组合成一个函数
- 从左到右执行
- 前一个函数的输出作为后一个函数的输入

### 实现思路

1. 接收多个函数作为参数
2. 从左到右依次执行
3. 传递参数和返回值

### 代码实现

```javascript
function pipe(...fns) {
  return function (...args) {
    return fns.reduce((result, fn) => {
      return fn(result);
    }, args);
  };
}
```

### 使用示例

```javascript
const add = (x) => x + 1;
const multiply = (x) => x * 2;
const square = (x) => x * x;

const piped = pipe(add, multiply, square);
console.log(piped(2)); // ((2 + 1) * 2) ^ 2 = 36
```

## 5. 高级应用

### 组合柯里化和函数组合

```javascript
// 柯里化的组合函数
const curriedCompose = curry(compose);
const curriedPipe = curry(pipe);

// 使用示例
const add = (x) => x + 1;
const multiply = (x) => x * 2;
const square = (x) => x * x;

const transform = curriedCompose(square, multiply, add);
console.log(transform(2)); // 36

const transform2 = curriedPipe(add, multiply, square);
console.log(transform2(2)); // 36
```

### 带初始值的函数组合

```javascript
function composeWith(initialValue, ...fns) {
  return fns.reduceRight((result, fn) => fn(result), initialValue);
}

function pipeWith(initialValue, ...fns) {
  return fns.reduce((result, fn) => fn(result), initialValue);
}

// 使用示例
const result = composeWith(
  2,
  (x) => x * x,
  (x) => x * 2,
  (x) => x + 1
);
console.log(result); // 36
```
