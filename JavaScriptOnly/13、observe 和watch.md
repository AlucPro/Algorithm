# 观察和监听模式的实现

## 1. Proxy 基础用法

### 定义

```javascript
const proxy = new Proxy(target, handler);
```

- 创建对象的代理
- 拦截对象操作
- 自定义对象行为
- 实现数据监听

### 实现思路

1. 定义目标对象
2. 创建处理器对象
3. 设置拦截方法
4. 创建代理对象

### 代码实现

```javascript
const target = {};
const handler = {
  // 拦截属性读取
  get(target, propKey, receiver) {
    console.log(`Getting ${propKey}`);
    return target[propKey];
  },

  // 拦截属性设置
  set(target, prop, value) {
    if (prop === "age") {
      if (!Number.isInteger(value)) {
        throw new TypeError("The age is not an integer");
      }
      if (value > 200) {
        throw new RangeError("The age seems invalid");
      }
    }
    target[prop] = value;
    return true;
  },

  // 拦截属性删除
  deleteProperty(target, property) {
    console.log(`Deleting ${property}`);
    return delete target[property];
  },
};

const proxy = new Proxy(target, handler);
```

### 使用示例

```javascript
proxy.name = "Alice";
console.log(proxy.name); // Getting name, Alice

proxy.age = 25;
console.log(proxy.age); // Getting age, 25

delete proxy.name; // Deleting name
```

## 2. 观察者模式实现

### 定义

```javascript
function ob(obj) {
  // 创建可观察对象
}

function watch(fn) {
  // 添加观察函数
}
```

- 实现数据监听
- 支持多属性观察
- 自动触发更新
- 支持属性删除

### 实现思路

1. 使用 Map 存储观察函数
2. 使用 Proxy 拦截操作
3. 实现依赖收集
4. 实现更新触发

### 代码实现

```javascript
let InWatchFn = null;

function ob(obj) {
  const watchFns = new Map(); // Map<string, Set<Function>>

  const handler = {
    // 拦截属性读取
    get(target, prop) {
      if (InWatchFn) {
        if (!watchFns.has(prop)) {
          watchFns.set(prop, new Set());
        }
        watchFns.get(prop).add(InWatchFn);
      }
      return target[prop];
    },

    // 拦截属性设置
    set(target, prop, value) {
      const oldValue = target[prop];
      target[prop] = value;

      if (oldValue !== value && watchFns.has(prop)) {
        [...watchFns.get(prop)].forEach((fn) => fn());
      }

      return true;
    },

    // 拦截属性删除
    deleteProperty(target, prop) {
      if (watchFns.has(prop)) {
        watchFns.delete(prop);
      }
      return delete target[prop];
    },
  };

  return new Proxy(obj, handler);
}

function watch(fn) {
  InWatchFn = fn;
  fn();
  InWatchFn = null;
}
```

### 使用示例

```javascript
const data = ob({
  count: 0,
  foo: "test",
});

// 观察 count 属性
watch(() => {
  console.log("count changed:", data.count);
});

// 观察 foo 属性
watch(() => {
  console.log("foo changed:", data.foo);
});

// 修改属性
data.count += 1;
// 输出: count changed: 1

data.foo = "test2";
// 输出: foo changed: test2

// 删除属性
delete data.count;
// 无输出，因为观察函数已被移除
```

## 3. 完整实现

### 定义

```javascript
class Observer {
  // 完整的观察者类
}
```

- 支持深度观察
- 支持批量更新
- 支持异步更新
- 支持取消观察

### 实现思路

1. 扩展基础功能
2. 添加深度观察
3. 实现批量更新
4. 添加异步支持

### 代码实现

```javascript
class Observer {
  constructor(data) {
    this.data = data;
    this.watchers = new Map();
    this.proxy = this.createProxy(data);
  }

  createProxy(data) {
    const watchers = this.watchers;

    return new Proxy(data, {
      get(target, prop) {
        if (Observer.currentWatcher) {
          if (!watchers.has(prop)) {
            watchers.set(prop, new Set());
          }
          watchers.get(prop).add(Observer.currentWatcher);
        }
        return target[prop];
      },

      set(target, prop, value) {
        const oldValue = target[prop];
        target[prop] = value;

        if (oldValue !== value && watchers.has(prop)) {
          Observer.batchUpdate(() => {
            [...watchers.get(prop)].forEach((watcher) => watcher());
          });
        }

        return true;
      },

      deleteProperty(target, prop) {
        if (watchers.has(prop)) {
          watchers.delete(prop);
        }
        return delete target[prop];
      },
    });
  }

  watch(fn) {
    Observer.currentWatcher = fn;
    fn();
    Observer.currentWatcher = null;
  }

  unwatch(fn) {
    for (const [prop, watchers] of this.watchers) {
      watchers.delete(fn);
    }
  }

  static currentWatcher = null;

  static batchUpdate(callback) {
    setTimeout(callback, 0);
  }
}
```

### 使用示例

```javascript
const observer = new Observer({
  user: {
    name: "Alice",
    age: 25,
  },
  items: ["apple", "banana"],
});

// 观察 user.name
observer.watch(() => {
  console.log("user.name changed:", observer.proxy.user.name);
});

// 观察 items
observer.watch(() => {
  console.log("items changed:", observer.proxy.items);
});

// 修改属性
observer.proxy.user.name = "Bob";
// 输出: user.name changed: Bob

observer.proxy.items.push("orange");
// 输出: items changed: ['apple', 'banana', 'orange']

// 取消观察
observer.unwatch(() => {
  console.log("items changed:", observer.proxy.items);
});
```
