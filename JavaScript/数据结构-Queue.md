# [Queue：队列](https://github.com/trekhleb/javascript-algorithms/tree/master/src/data-structures/queue)

> 队列主要比较的对象是栈，队列是先进先出First-In-First-Out

特点：

- 有顺序，列表
- 先进先出
- 队列是运算受到限制的一种线性表
- 严格按照插入只能在队尾插入，删除只能删除队头

```js
import LinkedList from '../linked-list/LinkedList';

export default class Queue {
  constructor() {
    // We're going to implement Queue based on LinkedList since the two
    // structures are quite similar. Namely, they both operate mostly on
    // the elements at the beginning and the end. Compare enqueue/dequeue
    // operations of Queue with append/deleteHead operations of LinkedList.
    this.linkedList = new LinkedList();
  }

  /**
   * @return {boolean}
   */
  isEmpty() {
    return !this.linkedList.head;
  }

  /**
   * Read the element at the front of the queue without removing it.
   * @return {*}
   */
  peek() {
    if (!this.linkedList.head) {
      return null;
    }

    return this.linkedList.head.value;
  }

  /**
   * Add a new element to the end of the queue (the tail of the linked list).
   * This element will be processed after all elements ahead of it.
   * @param {*} value
   */
  enqueue(value) {
    this.linkedList.append(value);
  }

  /**
   * Remove the element at the front of the queue (the head of the linked list).
   * If the queue is empty, return null.
   * @return {*}
   */
  dequeue() {
    const removedHead = this.linkedList.deleteHead();
    return removedHead ? removedHead.value : null;
  }

  /**
   * @param [callback]
   * @return {string}
   */
  toString(callback) {
    // Return string representation of the queue's linked list.
    return this.linkedList.toString(callback);
  }
}
```

- 基础存储结构采用链表
- 进队和出队是必须有的`enqueue`、`dequeue`
- 由于基本存储结构采用链表，可以通过头部来判断当前队列是否为空`isEmpty`
- `peek`查看队列最前端的元素，也即即将出队的
