## 栈
栈（stack）又名堆栈，它是一种运算受限的线性表。限定仅在表尾进行插入和删除操作。后进先出(LIFO = Last In First Out)。

### 操作
* push(element): 添加一个新元素到栈顶位置。
* pop()：移除栈顶的元素，同时返回被移除的元素。
* peek()：返回栈顶的元素，不对栈做任何修改。
* isEmpty()：判断栈是否是空。
* size()：返回栈里的元素个数。
* clear()：移除栈里的所有元素。

### js实现
```js
class Stack {
    constructor() {
        this._elements = [];
    }

    push(element) {
        this._elements.push(element);
    }

    pop() {
        if (this.isEmpty()) {
            return;
        }
        return this._elements.pop();
    }

    peek() {
        if (this.isEmpty()) {
            return;
        }
        return this._elements[this._elements.length - 1];
    }

    isEmpty() {
        return this._elements.length === 0;
    }

    size() {
        return this._elements.length;
    }

    clear() {
        this._elements = [];
    }
}
```

## 队列
队列(Queue)，它也是一种运算受限的线性表，限定只能在表的前端进行删除操作，而在表的后端进行插入操作。先进先出(FIFO = First In First Out)。

### 操作
* enqueue(element)：入队，向队列尾部添加一个新的项。
* dequeue()：出队，移除队首项（即排在队列最前面的），并返回被移除的元素。
* front()：返回队首元素。队列不做任何变动。
* isEmpty()：判断队列是否是空。
* size()：返回队列包含的元素个数。
* clear()：清空队列。

### js实现
```js
class Queue {
    constructor() {
        this._elements = [];
    }

    enqueue(element) {
        this._elements.unshift(element);
    }

    dequeue() {
        if (this.isEmpty()) {
            return;
        }
        return this._elements.pop();
    }

    front() {
        return this._elements[this._elements.length - 1];
    }

    isEmpty() {
        return this._elements.length === 0;
    }

    size() {
        return this._elements.length;
    }

    clear() {
        this._elements = [];
    }
}
```