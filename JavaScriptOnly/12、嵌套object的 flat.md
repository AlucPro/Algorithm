# 嵌套对象的扁平化实现

## 1. 基础实现

### 定义

```javascript
function flat(data) {
  // 将嵌套对象转换为扁平化对象
}
```

- 支持对象和数组的嵌套
- 使用点号（.）和方括号（[]）表示路径
- 保持原始值不变
- 递归处理嵌套结构

### 实现思路

1. 使用递归方式处理嵌套结构
2. 构建属性路径
3. 处理数组和对象
4. 合并结果

### 代码实现

```javascript
function flat(data) {
  const doFlat = (obj, prefix) => {
    let result = {};

    if (Array.isArray(obj)) {
      obj.forEach((item, index) => {
        const path = `${prefix}[${index}]`;
        const flattened = doFlat(item, path);
        result = { ...result, ...flattened };
      });
    } else if (obj !== null && typeof obj === "object") {
      for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
          const path = `${prefix}${prefix ? "." : ""}${key}`;
          const flattened = doFlat(obj[key], path);
          result = { ...result, ...flattened };
        }
      }
    } else {
      if (prefix) {
        result[prefix] = obj;
      }
    }

    return result;
  };

  return doFlat(data, "");
}
```

### 使用示例

```javascript
const data = {
  a: "name",
  b: { c: "haha" },
  d: [1, 2],
};

const flatData = flat(data);
console.log(flatData);
// 输出: { a: 'name', 'b.c': 'haha', 'd[0]': 1, 'd[1]': 2 }
```

## 2. 完整实现

### 定义

```javascript
class ObjectFlattener {
  // 完整的对象扁平化工具
}
```

- 支持自定义分隔符
- 支持数组索引格式
- 支持空值处理
- 支持类型转换

### 实现思路

1. 扩展基础功能
2. 添加配置选项
3. 优化路径生成
4. 添加类型处理

### 代码实现

```javascript
class ObjectFlattener {
  constructor(options = {}) {
    this.options = {
      separator: ".",
      arrayIndex: true,
      skipNull: false,
      ...options,
    };
  }

  // 扁平化处理
  flatten(data) {
    return this._flatten(data, "");
  }

  // 内部递归方法
  _flatten(obj, prefix) {
    let result = {};

    if (Array.isArray(obj)) {
      obj.forEach((item, index) => {
        const path = this.options.arrayIndex
          ? `${prefix}[${index}]`
          : `${prefix}${this.options.separator}${index}`;
        const flattened = this._flatten(item, path);
        result = { ...result, ...flattened };
      });
    } else if (obj !== null && typeof obj === "object") {
      for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
          const path = `${prefix}${prefix ? this.options.separator : ""}${key}`;
          const flattened = this._flatten(obj[key], path);
          result = { ...result, ...flattened };
        }
      }
    } else {
      if (prefix && (!this.options.skipNull || obj !== null)) {
        result[prefix] = obj;
      }
    }

    return result;
  }
}
```

### 使用示例

```javascript
const flattener = new ObjectFlattener({
  separator: "_",
  arrayIndex: false,
  skipNull: true,
});

const data = {
  user: {
    name: "Alice",
    age: null,
    address: {
      city: "Wonderland",
      zip: 12345,
    },
  },
  items: ["apple", "banana"],
  settings: {
    theme: "dark",
    notifications: true,
  },
};

const flatData = flattener.flatten(data);
console.log(flatData);
// 输出: {
//   'user_name': 'Alice',
//   'user_address_city': 'Wonderland',
//   'user_address_zip': 12345,
//   'items_0': 'apple',
//   'items_1': 'banana',
//   'settings_theme': 'dark',
//   'settings_notifications': true
// }
```
