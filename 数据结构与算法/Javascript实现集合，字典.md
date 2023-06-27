## 集合

ES6 提供了新的数据结构 Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。

### 操作

- constructor：构造函数，可接受一个数组。
- add(value)：向集合添加一个新的项。
- delete(value)：从集合移除一个项。
- has(value)：判断某个元素是否在集合中。
- clear()：清除集合中的所有项。
- size()：返回集合所包含元素的个数。
- keys()：返回键名的遍历器。
- values()：返回键值的遍历器。

### js 实现

集合比较常见的实现方式是哈希表，这里使用 JavaScript 的 Object 进行封装。js 中的对象是基于哈希表结构的。

```js
class MySet {
  constructor(data) {
    this.items = {};
    if (data && Array.isArray(data)) {
      data.forEach((v) => this.add(v));
    }
  }

  add(value) {
    // 判断是否存在
    if (this.has(value)) {
      return false;
    }
    this.items[value] = value;
    return true;
  }

  delete(value) {
    if (!this.has(value)) {
      return false;
    }
    delete this.items[value];
    return true;
  }

  has(value) {
    return this.items.hasOwnProperty(value);
  }

  clear() {
    this.items = {};
  }

  size() {
    return this.keys().length;
  }

  keys() {
    return Object.keys(this.items);
  }

  values() {
    return Object.values(this.items);
  }
}
```

## 字典

字典存储的是键值对，主要特点是一一对应。字典中 key 是不能重复且无序的，而 value 可以重复。

js 中的 Object 对象本质上就是一种字典结构，但是只能用字符串当作键，所以又推出了 Map 类型，各种类型的值（包括对象）都可以当作键。

有的语言叫 Dictionary ，有的语言叫 Map 。

### 操作

- set(key, value)：设置键名 key 对应的键值为 value，然后返回整个 Map 结构。
- get(key)：读取 key 对应的键值，找不到 key，返回 undefined
- has(key)：判断某个键是否存在。
- delete(key)：删除一项。
- clear()：清除集合中的所有项。
- size()：返回成员个数。
- keys()：返回键名的遍历器。
- values()：返回键值的遍历器。

### js 实现

```js
class MyMap {
  constructor() {
    this.items = {};
  }

  set(key, value) {
    this.items[key] = value;
  }

  get(key) {
    return this.has(key) ? this.items[key] : undefined;
  }

  has(key) {
    return this.items.hasOwnProperty(key);
  }

  delete(key) {
    if (!this.has(key)) return false;
    delete this.items[key];
    return true;
  }

  clear() {
    this.items = {};
  }

  size() {
    return this.keys().length;
  }

  keys() {
    return Object.keys(this.items);
  }

  values() {
    return Object.values(this.items);
  }
}
```
