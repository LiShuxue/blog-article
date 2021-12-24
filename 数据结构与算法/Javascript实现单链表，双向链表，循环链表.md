## 单链表
单链表是一种链式存取的数据结构，链表中的数据是以结点来表示的，每个结点的构成：元素 + 指针(指示后继元素存储位置)

### 操作
* append(data)：向列表尾部添加一个新的项。链表的创建分为头插法和尾插法，我们采用尾插法。
* insert(position, data)：向列表的特定位置插入一个新的项。
* get(position)：获取指定位置的节点中的数据。
* indexOf(data)：返回元素的位置。如果列表中没有该元素则返回-1。
* update(position, data) 修改指定位置节点的数据，返回修改的节点。
* removeAt(position)：从列表的特定位置移除一项，返回删除的节点。
* remove(data)：从列表中移除一项，返回删除的节点。
* isEmpty()：判断链表是否为空。
* size()：返回链表包含的元素个数。

### js实现
```js
// 封装节点类
class Node {
    constructor(data) {
        this.data = data;
        this.next = null;
    }
}

class LinkedList {
    constructor() {
        this.length = 0;
        // 链表的第一个节点
        this.head = null;
    }

    append(data) {
        // 创建新节点
        const node = new Node(data);

        // 判断链表是否为空
        if (this.head === null) {
            this.head = node;
        } else {
            // 从头遍历链表的每一个节点，遍历到最后一个节点，即next是null的，将新的节点添加到最后
            let currentNode = this.head;
            while(currentNode.next !== null) {
                currentNode = currentNode.next;
            }
            currentNode.next = node;
        }
        // 追加完新节点后，链表长度 + 1
        this.length++;
    }

    insert(position, data) {
        // 越界判断
        if (position === null || position === undefined || position < 0 || position > this.length) {
            return;
        }

        // 创建新节点
        const node = new Node(data);

        if (position === 0) {
            // 新节点的下一个是原来的head，链表的head变为新的
            node.next = this.head;
            this.head = node;
        } else {
            // 遍历链表，找到要插入的位置的节点，和他的上一个节点。
            let index = 0;
            let previous = null;
            let current = this.head;
            while(index < position) {
                previous = current;
                current = current.next;
                index++;
            }
            // 该位置原来的节点即为新节点的next，源节点的上一个节点的next变为该新节点
            node.next = current;
            previous.next = node;
        }

        this.length++;
        return node;
    }

    get(position) {
        // 越界判断
        if (position === null || position === undefined || position < 0 || position >= this.length) {
            return;
        }

        // 遍历链表，找到该位置的节点。
        let index = 0;
        let current = this.head;
        while(index < position) {
            current = current.next;
            index++;
        }

        return current.data;
    }

    indexOf(data) {
        // 遍历链表，找到该位置的节点。
        let index = 0;
        let current = this.head;
        while(current) {
            if (current.data === data) {
                return index;
            }
            current = current.next;
            index++;
        }
        return -1;
    }

    update(position, data) {
        // 越界判断
        if (position === null || position === undefined || position < 0 || position >= this.length) {
            return;
        }

        // 遍历链表，找到该位置的节点。
        let index = 0;
        let current = this.head;
        while(index < position) {
            current = current.next;
            index++;
        }

        // 替换数据
        current.data = data;
        return current;
    }

    removeAt(position) {
        // 越界判断
        if (position === null || position === undefined || position < 0 || position >= this.length) {
            return;
        }

        let current = this.head;
        if (position === 0) {
            // 将头部元素改为原来的下一个
            this.head = this.head.next;
        } else {
            // 遍历链表，找到删除的位置的节点。
            let index = 0;
            let previous = null;
            while(index < position) {
                previous = current;
                current = current.next;
                index++;
            }
            // 原节点的上一个节点的next变为该节点的next，即为删除了该节点
            previous.next = current.next;
        }

        this.length--;
        return current;
    }

    remove(data) {
        return this.removeAt(this.indexOf(data));
    }

    isEmpty() {
        return this.length === 0;
    }

    size() {
        return this.length;
    }
}
```

## 双向链表
在单链表的基础上，对于每一个结点设计一个前驱结点，前驱结点与前一个结点相互连接，构成一个链表。它的每个数据结点中都有两个指针。

### js实现
不同点：
* Node中添加了一个this.prev属性, 该属性用于指向上一个节点
* 属性中添加了一个this.tail属性, 该属性指向末尾的节点
```js
// 封装节点类
class DoublyNode extends Node {
    constructor(data) {
        super(data);
        this.prev = null;
    }
}

class DoublyLinkedList extends LinkedList {
    constructor() {
        super();
        // 链表的最后一个节点
        this.tail = null;
    }

    append(data) {
        // 创建新节点
        const node = new DoublyNode(data);

        // 判断链表是否为空
        if (this.head === null) {
            this.head = node;
            this.tail = node;
        } else {
            // 跟单向链表不同，不用从头遍历，因为我们有最后一个元素。所以直接在最后一个元素上拼下一个
            this.tail.next = node;
            node.prev = this.tail;
            this.tail = node;
        }
        // 追加完新节点后，链表长度 + 1
        this.length++;
    }

    insert(position, data) {
        // 越界判断
        if (position === null || position === undefined || position < 0 || position > this.length) {
            return;
        }

        // 创建新节点
        const node = new DoublyNode(data);

        if (position === 0) { // 在第一个位置插入
            if (this.head === null) {
                this.head = node;
                this.tail = node;
            } else {
                // 用新的node替换head
                node.next = this.head;
                this.head.prev = node;
                this.head = node;
            }
        } else if (position === this.length) { // 在最后一个位置插入
            this.tail.next = node;
            node.prev = this.tail;
            this.tail = node;
        } else {
            // 遍历链表，找到要插入的位置的节点，和他的上一个节点。
            let index = 0;
            let previous = null;
            let current = this.head;
            while(index < position) {
                previous = current;
                current = current.next;
                index++;
            }
            // 交换节点信息
            node.next = current;
            current.prev = node;
            previous.next = node;
            node.prev = previous;
        }

        this.length++;
        return node;
    }

    update(position, data) {
        // 删除 position 位置的节点
        this.removeAt(position);

        // 在 position 位置插入元素
        const node = this.insert(position, data);
        return node;
    }

    removeAt(position) {
        // 越界判断
        if (position === null || position === undefined || position < 0 || position >= this.length) {
            return;
        }

        let current = this.head;
        if (position === 0) { // 删除第一个
            if (this.length === 1) { // 链表内只有一个节点的情况
                this.head = null;
                this.tail = null;
            } else { // 链表内有多个节点的情况
                this.head = this.head.next;
                this.head.prev = null;
            }
        } else if (position === this.length - 1) { // 删除最后一个
            current = this.tail;
            this.tail.prev.next = null;
            this.tail = this.tail.prev;
        } else {
            // 遍历链表，找到删除的位置的节点。
            let index = 0;
            let previous = null;
            while(index < position) {
                previous = current;
                current = current.next;
                index++;
            }
            // 原节点的上一个节点的next变为该节点的next，即为删除了该节点
            previous.next = current.next;
            current.next.prev = previous;
        }

        this.length--;
        return current;
    }
}
```

## 循环链表
循环链表和非循环链表其实创建的过程以及思路几乎完全一样，唯一不同的是，非循环链表的尾结点指向null，而循环链表的尾指针指向的是链表的开头。

### js实现
```js
class CircularLinkedList extends LinkedList {
    constructor() {
        super()
    }

    append(data) {
        // 创建新节点
        const node = new Node(data);

        // 判断链表是否为空
        if (this.head === null) {
            this.head = node;
            node.next = this.head; // 最重要的是要把这最后节点的next指向头
        } else {
            // 从头遍历链表的每一个节点，遍历到最后一个节点，即next是head的，将新的节点添加到最后
            let currentNode = this.head;
            while(currentNode.next !== this.head) {
                currentNode = currentNode.next;
            }
            currentNode.next = node;
            node.next = this.head; // 最重要的是要把这最后节点的next指向头
        }
        // 追加完新节点后，链表长度 + 1
        this.length++;
    }

    findLast() {
        // 遍历链表，找到最后一个节点。
        let index = 0;
        let current = this.head;
        while(index < this.length - 1) {
            current = current.next;
            index++;
        }

        return current;
    }

    insert(position, data) {
        // 越界判断
        if (position === null || position === undefined || position < 0 || position > this.length) {
            return;
        }

        // 创建新节点
        const node = new Node(data);

        if (position === 0) {
            // 最后一个节点
            let last = this.findLast();

            // 新节点的下一个是原来的head，链表的head变为新的
            node.next = this.head;
            this.head = node;
            last.next = this.head;
        } else if (position === this.length) {
            // 最后一个节点
            let last = this.findLast();
            last.next = node;
            node.next = this.head;
        } else {
            // 遍历链表，找到要插入的位置的节点，和他的上一个节点。
            let index = 0;
            let previous = null;
            let current = this.head;
            while(index < position) {
                previous = current;
                current = current.next;
                index++;
            }
            // 该位置原来的节点即为新节点的next，源节点的上一个节点的next变为该新节点
            node.next = current;
            previous.next = node;
        }

        this.length++;
        return node;
    }

    indexOf(data) {
        // 遍历链表，找到该位置的节点。
        let index = 0;
        let current = this.head;
        while(current) {
            if (current.data === data) {
                return index;
            }
            current = current.next;
            index++;
            if (index >= this.length) {
                break;
            }
        }
        return -1;
    }

    removeAt(position) {
        // 越界判断
        if (position === null || position === undefined || position < 0 || position >= this.length) {
            return;
        }

        let current = this.head;
        if (position === 0) {
            let last = this.findLast();

            // 将头部元素改为原来的下一个
            this.head = this.head.next;
            last.next = this.head;
        } else {
            // 遍历链表，找到删除的位置的节点。
            let index = 0;
            let previous = null;
            while(index < position) {
                previous = current;
                current = current.next;
                index++;
            }
            // 原节点的上一个节点的next变为该节点的next，即为删除了该节点
            previous.next = current.next;
        }

        this.length--;
        return current;
    }
}
```