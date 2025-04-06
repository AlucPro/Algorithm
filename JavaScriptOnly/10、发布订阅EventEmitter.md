# 发布订阅模式的实现

## 1. 基础实现

### 定义

```javascript
class MyEventEmitter {
  // 事件发射器
}
```

- 实现发布-订阅模式
- 支持事件注册和触发
- 支持多事件类型
- 支持参数传递

### 实现思路

1. 使用 Map 存储事件和对应的处理函数
2. 实现事件的注册（on）和触发（emit）
3. 支持多个事件处理函数
4. 支持参数传递

### 代码实现

```javascript
class MyEventEmitter {
  constructor() {
    this.events = new Map();
  }

  // 注册事件
  on(eventName, fn) {
    if (!this.events.has(eventName)) {
      this.events.set(eventName, []);
    }
    const events = this.events.get(eventName);
    this.events.set(eventName, [...events, fn]);
  }

  // 触发事件
  emit(eventName, ...args) {
    if (!this.events.has(eventName)) return;
    const events = this.events.get(eventName);
    events.forEach((fn) => {
      fn(...args);
    });
  }
}
```

### 使用示例

```javascript
const emitter = new MyEventEmitter();

// 注册事件
emitter.on("message", (data) => {
  console.log(`收到消息: ${data}`);
});

emitter.on("message", (data) => {
  console.log(`再次收到消息: ${data}`);
});

// 触发事件
emitter.emit("message", "Hello World");
// 输出：
// 收到消息: Hello World
// 再次收到消息: Hello World
```

## 2. 完整实现

### 定义

```javascript
class MyEventEmitter {
  // 完整的事件发射器
}
```

- 支持事件移除
- 支持一次性事件
- 支持事件计数
- 支持错误处理

### 实现思路

1. 扩展基础功能
2. 添加事件移除方法
3. 实现一次性事件
4. 添加错误处理机制

### 代码实现

```javascript
class MyEventEmitter {
  constructor() {
    this.events = new Map();
  }

  // 注册事件
  on(eventName, fn) {
    if (!this.events.has(eventName)) {
      this.events.set(eventName, []);
    }
    const events = this.events.get(eventName);
    this.events.set(eventName, [...events, fn]);
  }

  // 注册一次性事件
  once(eventName, fn) {
    const onceFn = (...args) => {
      fn(...args);
      this.off(eventName, onceFn);
    };
    this.on(eventName, onceFn);
  }

  // 移除事件
  off(eventName, fn) {
    if (!this.events.has(eventName)) return;
    const events = this.events.get(eventName);
    const newEvents = events.filter((eventFn) => eventFn !== fn);
    this.events.set(eventName, newEvents);
  }

  // 移除所有事件
  removeAllListeners(eventName) {
    if (eventName) {
      this.events.delete(eventName);
    } else {
      this.events.clear();
    }
  }

  // 触发事件
  emit(eventName, ...args) {
    if (!this.events.has(eventName)) return;
    const events = this.events.get(eventName);
    events.forEach((fn) => {
      try {
        fn(...args);
      } catch (error) {
        console.error(`事件 ${eventName} 处理出错:`, error);
      }
    });
  }

  // 获取事件监听器数量
  listenerCount(eventName) {
    if (!this.events.has(eventName)) return 0;
    return this.events.get(eventName).length;
  }
}
```

### 使用示例

```javascript
const emitter = new MyEventEmitter();

// 注册事件
const handler1 = (data) => {
  console.log(`处理器1收到消息: ${data}`);
};

const handler2 = (data) => {
  console.log(`处理器2收到消息: ${data}`);
};

emitter.on("message", handler1);
emitter.on("message", handler2);

// 注册一次性事件
emitter.once("login", (user) => {
  console.log(`用户 ${user} 登录成功`);
});

// 触发事件
emitter.emit("message", "Hello World");
// 输出：
// 处理器1收到消息: Hello World
// 处理器2收到消息: Hello World

emitter.emit("login", "张三");
// 输出：用户 张三 登录成功

emitter.emit("login", "李四");
// 无输出，因为是一次性事件

// 移除事件
emitter.off("message", handler1);

// 再次触发事件
emitter.emit("message", "Hello Again");
// 输出：
// 处理器2收到消息: Hello Again

// 获取事件监听器数量
console.log(emitter.listenerCount("message")); // 1
```
