## 哈希表
哈希表也叫散列表，通常是基于数组实现的，它提供了快速的插入操作和查找操作，无论哈希表中有多少条数据，插入和查找的时间复杂度都是为O(1)。

哈希表中的数据是没有顺序的，所以不能以一种固定的方式（比如从小到大 ）来遍历其中的元素。通常情况下，哈希表中的 key 是不允许重复的，不能放置相同的 key，用于保存不同的元素。

js的对象Object就是基于哈希表实现的。

## 哈希表函数
哈希表采用的是一种转换思想，其中一个重要的概念是如何将「键」或者「关键字」转换成数组下标？在哈希表中，这个过程由哈希函数来完成，但是并不是每个「键」或者「关键字」都需要通过哈希函数来将其转换成数组下标，有些「键」或者「关键字」可以直接作为数组的下标。

哈希函数的作用就是帮我们把非int的「键」或者「关键字」转化成int，可以用来做数组的下标。

哈希函数不管怎么实现，都应该满足下面三个基本条件：
1. 哈希函数计算得到的散列值是一个非负整数
2. 如果 key1 = key2，那 hashFn(key1) == hashFn(key2)
3. 如果 key1 ≠ key2，那 hashFn(key1) ≠ hashFn(key2)

### 哈希函数的简单实现
```js
// 定义一个数组的长度
hashFn(string, limit = 7) {
  // 自己采用的一个质数（无强制要求，质数即可）
  const PRIME = 31;

  // 1、定义存储 hashCode 的变量
  let hashCode = 0;

  // 2、使用霍纳法则（秦九韶算法），计算 hashCode 的值
  for (let item of string) {
    hashCode = PRIME * hashCode + item.charCodeAt();
  }

  // 3、对 hashCode 取余，并返回
  return hashCode % limit;
}
```

## 哈希冲突
上面第三点看起来非常合理，但是经过哈希函数哈希化过后得到的下标值可能有重复，这种情况称为哈希冲突。

哈希冲突是不可避免的，我们常用解决哈希冲突的方法有两种「开放地址法」和「链地址法」。

### 链地址法
链地址法解决冲突的办法是每个数组单元中存储的不再是单个数据, 而是一个链条。常见的是数组或者链表。当一个数据被添加到同样的下标时，往这个项对应的数组或者链表后面插入。
![链地址法](https://cdn.lishuxue.site/blog/image/数据结构与算法/lian-di-zhi-fa.png)

### 开放地址法
开放地址法的主要工作方式是寻找空白的单元格来添加重复的数据。当下标中已经存在数据时，往后查找空白的单元格，并将内容放入。
![开放地址法](https://cdn.lishuxue.site/blog/image/数据结构与算法/kai-fang-di-zhi-fa.png)

根据探测空白单元格位置方式的不同，可分为三种方法：
* 线性探测
* 二次探测
* 再哈希法

#### 线性探测
当插入位置有数据时，依次往后寻找空白单元格。

但是这种插入方式会产生聚集。在聚集范围内的数据项，都需要一步一步往后移动，并且插入到聚集的后面，因此聚集变的越大，聚集增长的越快。

#### 二次探测
相比于线性探测，二次探测是防止聚集产生的一种尝试，思想是探测相隔较远的单元。但是二次探测只是对线性探测的优化，依然存在聚集的问题。

线性探测, 我们可以看成是步长为1的探测, 比如从下标值x开始, 那么线性测试就是x+1, x+2, x+3依次探测.

二次探测, 对步长做了优化, 比如从下标值x开始, 对x+1², x+2², x+3²等位置依次探测。

#### 再哈希法
在开放地址法中寻找空白单元格的最好的解决方式为再哈希化。当插入位置有数据时，把关键字用另一个哈希函数，再做一次哈希化，用这次哈希化的结果作为该关键字的步长。

## 哈希表的扩容与压缩
### 装填因子
装填因子（loadFactor）表示当前哈希表中已经包含的数据项和整个哈希表长度的比值；
* 装填因子 = 总数据项 / 哈希表长度；
* 开放地址法的装填因子最大为 1，因为只有空白的单元才能放入元素；
* 链地址法的装填因子可以大于 1，因为只要愿意，拉链法可以无限延伸下去；
### 为什么需要扩容？
前面我们在哈希表中使用的是长度为 7 的数组，由于使用的是链地址法，装填因子(loadFactor)可以大于 1，所以这个哈希表可以无限制地插入新数据。

但是，随着数据量的增多，每一个下标对应的数组(链表)就会越来越长，这就会造成哈希表效率的降低。

### 什么情况下需要扩容？
常见的情况是 装填因子 > 0.75 的时候进行扩容。

### 哈希表的容量为啥最好是质数
哈希函数一个简单的实现方法是将关键字通过某种运算得到一个整数，再让这个整数除以哈希表的大小，取其余数，以此作为关键字的存储位置。质数只能被1和他自身整除，因此数据会均匀分布到每一个单元格。

### 如何进行扩容？
* 简单的扩容可以直接扩大两倍。
* 扩容之后所有的数据项都要进行同步修改

两倍扩容或压缩之后，通过循环调用 isPrime 判断得到的容量是否为质数，不是则+1，直到是为止。比如原长度：7，2 倍扩容后长度为 14，14 不是质数，14 + 1 = 15 不是质数，15 + 1 = 16 不是质数，16 + 1 = 17 是质数，停止循环，由此得到质数 17。

## 哈希表常见操作
* put(key, value) 插入或修改操作。
* get(key) 获取哈希表中特定位置的元素。
* remove(key) 删除哈希表中特定位置的元素。
* isEmpty() 如果哈希表中不包含任何元素，返回 trun，如果哈希表长度大于 0 则返回 false。
* size() 返回哈希表包含的元素个数。
* resize(value) 对哈希表进行扩容操作。

## js 实现
我们采用数组实现哈希表，通过链地址法存储数据，每一个数组项里面还是数组。
```js
// 将关键字通过某种运算得到一个整数，再让这个整数除以哈希表的大小，取其余数，以此作为关键字的存储位置。
const hashFn = (string, limit = 7) => {
  // 1、关键字转换后的整数
  let hashCode = 0;

  // 2、转换
  for (let item of string) {
    hashCode = hashCode  + item.charCodeAt();
  }

  // 3、对 hashCode 取余，并返回
  return hashCode % limit;
}

// 判断一个数是否为质数
const isPrime = (number) => {
    if (number <= 1) return false;
    for (let i = 2; i < number; i++) {
        if (number % i === 0) {
            return false;
        }
    }
    return true;
}

// 根据传入的number，获取最邻近的质数
const getPrime = (number) => {
    while (!isPrime(number)) {
      number++;
    }
    return number;
}

class HashTable {
    constructor() {
        this.storage = []; // 哈希表总存储结构
        this.count = 0; // 哈希表元素个数
        this.limit = 7; // 哈希表长度（初始设为质数 7）
        // 装填因子（已有个数 / 总个数）
        this.loadFactor = 0.75;
        this.minLoadFactor = 0.25;
    }

    put(key, value) {
        // 通过哈希函数算出存储的位置
        const index = hashFn(key, this.limit);

        // 获取当前位置上的值
        let data = this.storage[index];

        // 判断当前位置上的值，并加入
        if (data === undefined) {
            data = [];
            this.storage[index] = data;
        } 

        // 判断是否是修改某个值
        for (let i=0; i<data.length; i++) {
            let temp = data[i]; // temp 的格式：[key, value]
            if (temp[0] === key) { // 如果 key 相等，则修改数据
                temp[1] = value;
                return;
            }
        }

        // 如果不是修改，将数据添加到这个链条的最后。链地址法
        data.push([key, value]);
        this.count++;

        // 判断哈希表是否需要扩容，若装填因子 > 0.75，则扩容
        if (this.count / this.limit > this.loadFactor) {
            // 直接两倍扩容，取最近的质数
            this.resize(getPrime(this.limit * 2))
        }
    }

    get(key) {
        const index = hashFn(key, this.limit);
        const data = this.storage[index];
        if (data === undefined) return null;

        for (const temp of data) {
            if (temp[0] === key) {
                return temp[1];
            }
        }
        return null;
    }

    remove(key) {
        const index = hashFn(key, this.limit);
        const data = this.storage[index];
        if (data === undefined) return null;

        for (let i=0; i<data.length; i++) {
            let temp = data[i];
            if (temp[0] === key) {
                data.splice(i, 1);
                this.count--;

                // 根据装填因子的大小，判断是否要进行哈希表压缩
                if (this.limit > 7 && this.count / this.limit < this.minLoadFactor) {
                    // 直接缩小两倍容量，取最近的质数
                    this.resize(getPrime(Math.floor(this.limit / 2)));
                }

                return temp;
            }
        }
    }

    isEmpty() {
        return this.count === 0;
    }

    size() {
        return this.count;
    }

    resize(limit) {
        // 先保存旧的数据
        const oldStorage = this.storage;
        // 重置哈希表所有属性
        this.storage = [];
        this.count = 0;
        this.limit = limit;

        // 遍历旧的数据，重新 put 到 this.storage
        for (const oldData of oldStorage) {
            if (oldData) {
                for (const temp of oldData) {
                    this.put(temp[0], temp[1]);
                }
            }
        }
    }
}
```
