# ES5 和 ES6 继承的实现

## 1. ES6 继承

### 定义

```javascript
class Father {
  constructor(name) {
    this.name = name;
  }
  sayFName() {
    console.log("father is ", this.name);
  }
}

class Son extends Father {
  constructor(isBoy, fname) {
    super(fname); // 调用父类构造函数
    this.isBoy = isBoy;
  }
}
```

### 实现思路

1. 使用 `class` 关键字定义类
2. 使用 `extends` 关键字实现继承
3. 使用 `super` 调用父类构造函数
4. 子类可以访问父类的属性和方法

### 代码实现

```javascript
class Father {
  constructor(name) {
    this.name = name;
  }
  sayFName() {
    console.log("father is ", this.name);
  }
}

class Son extends Father {
  constructor(isBoy, fname) {
    super(fname); // 相当于 Father.constructor.call(this, fname)
    this.isBoy = isBoy;
  }
}
```

### 使用示例

```javascript
const s = new Son(true, "ff");
s.sayFName(); // 输出：father is ff
```

## 2. ES5 继承

### 定义

```javascript
function Father(name) {
  this.name = name;
}

Father.prototype.sayName = function () {
  console.log("father is ", this.name);
};

function Son(name, isBoy) {
  Father.call(this, name); // 继承属性
  this.isBoy = isBoy;
}

Son.prototype = Object.create(Father.prototype); // 继承方法
```

### 实现思路

1. 使用构造函数定义类
2. 使用 `call` 方法在子类构造函数中调用父类构造函数，实现属性继承
3. 使用 `Object.create` 创建父类原型的副本，实现方法继承
4. 修正子类构造函数的 `constructor` 指向

### 代码实现

```javascript
// 父类
function SwimingAnimal(type) {
  this.type = type;
  this.canSwiming = true;
}

SwimingAnimal.prototype.saycanSwimming = function () {
  console.log(this.canSwiming, this.type);
};

// 子类
function Dog(name) {
  // 1. 继承属性
  SwimingAnimal.call(this, "dog");
  this.name = name;
}

// 2. 继承方法
Dog.prototype = Object.create(SwimingAnimal.prototype);

// 3. 修正构造函数指向
Dog.prototype.constructor = Dog;
```

### 使用示例

```javascript
const d = new Dog("aa");
d.saycanSwimming(); // 输出：true dog
```

## 3. 两种继承方式的区别

### ES6 继承

- 使用 `class` 和 `extends` 关键字
- 语法更简洁，更接近面向对象编程
- 内置 `super` 关键字调用父类方法
- 支持静态方法和静态属性
- 支持私有方法和私有属性（使用 # 前缀）

### ES5 继承

- 使用构造函数和原型链
- 需要手动实现属性继承和方法继承
- 需要手动修正构造函数指向
- 实现相对复杂，但更灵活
- 可以更好地理解 JavaScript 的原型继承机制
