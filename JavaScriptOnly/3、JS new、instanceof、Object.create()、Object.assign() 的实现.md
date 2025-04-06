# JS 核心方法的手写实现

## 1. new 操作符的实现

### 定义

```javascript
const obj = new Constructor(arg1, arg2);
```

### 实现思路

1. 创建一个空对象
2. 将对象的原型指向构造函数的原型
3. 将构造函数的 this 指向新对象
4. 执行构造函数并获取返回值
5. 判断返回值是否为对象，是则返回该对象，否则返回新创建的对象

### 代码实现

```javascript
function myNew(fn, ...args) {
  // 1. 创建一个空对象，并且该对象的 prototype 指向构造函数的 prototype
  const obj = Object.create(fn.prototype);

  // 2. 执行构造函数，将 this 指向新对象，并传递参数
  const ret = fn.call(obj, ...args);

  // 3. 如果构造函数返回的是对象，则返回该对象；否则返回新创建的对象
  return ret instanceof Object ? ret : obj;
}
```

### 使用示例

```javascript
function Person(name) {
  this.name = name;
}

const p = myNew(Person, "John");
console.log(p.name); // 输出：John
console.log(p instanceof Person); // 输出：true
```

## 2. instanceof 的实现

### 定义

```javascript
obj instanceof Constructor;
```

- 检查对象的原型链上是否存在构造函数的原型
- 返回布尔值

### 实现思路

1. 获取对象的原型
2. 获取构造函数的原型
3. 循环检查对象的原型链
4. 如果找到构造函数的原型，返回 true
5. 如果到达原型链末端（null），返回 false

### 代码实现

```javascript
function myInstanceof(ins, fn) {
  // 1. 获取对象的原型
  let insP = ins.__proto__;

  // 2. 获取构造函数的原型
  const fnP = fn.prototype;

  // 3. 循环检查原型链
  while (insP) {
    if (insP === fnP) return true;
    insP = insP.__proto__;
  }

  return false;
}
```

### 使用示例

```javascript
function Person(name) {
  this.name = name;
}

const p = new Person("John");
console.log(myInstanceof(p, Person)); // 输出：true
console.log(myInstanceof(p, Object)); // 输出：true
console.log(myInstanceof(p, Array)); // 输出：false
```

## 3. Object.create() 的实现

### 定义

```javascript
Object.create(proto, propertiesObject);
```

- 创建一个新对象
- 使用现有对象作为新对象的原型
- 可选地添加属性描述符

### 实现思路

1. 创建一个空函数
2. 将函数的原型指向传入的对象
3. 返回该函数的实例
4. 可选地添加属性描述符

### 代码实现

```javascript
function myCreate(obj, properties) {
  // 1. 创建一个空函数
  function F() {}

  // 2. 将函数的原型指向传入的对象
  F.prototype = obj;

  // 3. 可选地添加属性描述符
  if (properties) {
    Object.defineProperties(obj, properties);
  }

  // 4. 返回该函数的实例
  return new F();
}
```

### 使用示例

```javascript
const proto = { x: 10 };
const obj = myCreate(proto);
console.log(obj.x); // 输出：10
console.log(obj.__proto__ === proto); // 输出：true
```

## 4. Object.assign() 的实现

### 定义

```javascript
Object.assign(target, ...sources);
```

- 将所有可枚举属性从一个或多个源对象复制到目标对象
- 返回目标对象

### 实现思路

1. 遍历所有源对象
2. 遍历源对象的所有可枚举属性
3. 将属性复制到目标对象
4. 返回目标对象

### 代码实现

```javascript
function assign(target, ...source) {
  // 1. 遍历所有源对象
  for (let obj of source) {
    // 2. 遍历源对象的所有可枚举属性
    for (let key in obj) {
      // 3. 确保是对象自身的属性
      if (obj.hasOwnProperty(key)) {
        target[key] = obj[key];
      }
    }
  }

  // 4. 返回目标对象
  return target;
}
```

### 使用示例

```javascript
const target = { a: 1 };
const source = { b: 2, c: 3 };
const result = assign(target, source);
console.log(result); // 输出：{ a: 1, b: 2, c: 3 }
console.log(target === result); // 输出：true
```
