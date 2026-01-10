# Stack & Queue – In-Depth Concepts & Implementations

---

## 1. What is a Stack?

### Introduction

A **Stack** is a linear data structure that follows the **LIFO (Last In, First Out)** principle.

- The element inserted last is removed first.
- Real-world example: a stack of plates, browser back button, undo/redo operations.

### Basic Operations

- **push(x)** → Insert an element
- **pop()** → Remove the top element
- **top()/peek()** → Get the top element without removing it
- **isEmpty()** → Check if stack is empty

---

## What is a Queue?

### Introduction

A **Queue** is a linear data structure that follows the **FIFO (First In, First Out)** principle.

- The element inserted first is removed first.
- Real-world example: ticket queue, CPU scheduling, printer queue.

### Basic Operations

- **push/enqueue(x)** → Insert element at rear
- **pop/dequeue()** → Remove element from front
- **front()/peek()** → Get front element
- **isEmpty()** → Check if queue is empty

---

## 2. Implement Stack Using an Array

### 1️⃣ Approach / Intuition

- Use a fixed-size array to store stack elements.
- Maintain an index (`topIndex`) that always points to the **top element**.
- Initially, `topIndex = -1` (stack empty).
- Increment `topIndex` on push, decrement on pop.

### 2️⃣ Code&#x20;

```js
class Stack {
  constructor(stackSize = 10) {
    this.stack = new Array(stackSize);
    this.stackSize = stackSize;
    this.topIndex = -1; // renamed to avoid conflict
  }

  push(element) {
    if (this.topIndex >= this.stackSize - 1) {
      console.log("Stack overflow");
      return;
    }

    this.stack[++this.topIndex] = element;
    console.log(this.stack);
  }

  top() {
    if (this.isEmpty()) {
      console.log("Stack is empty");
      return -1;
    }

    console.log(this.stack[this.topIndex]);
    return this.stack[this.topIndex];
  }

  pop() {
    if (this.isEmpty()) return -1;

    return this.stack[this.topIndex--];
  }

  size() {
    return this.topIndex + 1;
  }

  isEmpty() {
    return this.topIndex === -1;
  }
}
```

### 3️⃣ Time & Space Complexity

- **Push:** O(1)
- **Pop:** O(1)
- **Top:** O(1)
- **Space:** O(N)

---

## 3. Implement Stack Using a Linked List

### 1️⃣ Approach / Intuition

- Use a **singly linked list**.
- Head node represents the **top of the stack**.
- Push → Insert at head
- Pop → Remove from head

### 2️⃣ Code

```js
class Node {
  constructor(d) {
    this.val = d;
    this.next = null;
  }
}

class LinkedListStack {
  constructor() {
    this.head = null;
    this.sizeCount = 0;
  }

  push(x) {
    const element = new Node(x);
    element.next = this.head;
    this.head = element;
    this.sizeCount++;
  }

  pop() {
    if (this.head === null) return -1;

    const top = this.head.val;
    this.head = this.head.next;
    this.sizeCount--;
    return top;
  }

  top() {
    return this.head ? this.head.val : -1;
  }

  size() {
    return this.sizeCount;
  }
}
```

### 3️⃣ Time & Space Complexity

- **Push:** O(1)
- **Pop:** O(1)
- **Space:** O(N)

---

## 4. Implement Queue Using a Linked List

### 1️⃣ Approach / Intuition

- Maintain two pointers: **start (front)** and **end (rear)**.
- Enqueue at rear, dequeue from front.

### 2️⃣ Code

```js
class Node {
  constructor(value) {
    this.next = null;
    this.val = value;
  }
}

class QueueLL {
  constructor() {
    this.size = 0;
    this.start = this.end = null;
  }

  push(val) {
    const newNode = new Node(val);

    if (this.start === null) {
      this.start = this.end = newNode;
    } else {
      this.end.next = newNode;
      this.end = newNode;
    }
    this.size++;
  }

  pop() {
    if (this.start === null) return -1;

    const value = this.start.val;
    this.start = this.start.next;
    this.size--;
    return value;
  }

  top() {
    return this.start ? this.start.val : -1;
  }
}
```

### 3️⃣ Time & Space Complexity

- **Push:** O(1)
- **Pop:** O(1)
- **Space:** O(N)

---

## 5. Implement Queue Using Array (Circular Queue)

### 1️⃣ Approach / Intuition

- Use **circular indexing** to reuse array space.
- Maintain `startPtr`, `endPtr`, and `currSize`.

### 2️⃣ Code

```js
class Queue {
  constructor(capacity = 10) {
    this.capacity = capacity;
    this.currSize = 0;
    this.queueDS = new Array(capacity);
    this.startPtr = -1;
    this.endPtr = -1;
  }

  push(element) {
    if (this.currSize === this.capacity) return "Queue Overflow";

    if (this.currSize === 0) {
      this.startPtr = this.endPtr = 0;
    } else {
      this.endPtr = (this.endPtr + 1) % this.capacity;
    }

    this.queueDS[this.endPtr] = element;
    this.currSize++;
  }

  pop() {
    if (this.currSize === 0) return -1;

    const element = this.queueDS[this.startPtr];

    if (this.currSize === 1) {
      this.startPtr = this.endPtr = -1;
    } else {
      this.startPtr = (this.startPtr + 1) % this.capacity;
    }

    this.currSize--;
    return element;
  }

  top() {
    return this.currSize === 0 ? -1 : this.queueDS[this.startPtr];
  }
}
```

### 3️⃣ Time & Space Complexity

- **Push:** O(1)
- **Pop:** O(1)
- **Space:** O(N)

---

## 6. Implement Stack Using a Queue

### 1️⃣ Approach / Intuition

- Use **one queue**.
- After pushing an element, rotate the queue so the new element comes to the front.

### 2️⃣ Code

```cpp
class MyStack {
public:
    queue<int> q;

    void push(int x) {
        int s = q.size();
        q.push(x);
        for (int i = 0; i < s; i++) {
            q.push(q.front());
            q.pop();
        }
    }

    int pop() {
        int val = q.front();
        q.pop();
        return val;
    }

    int top() { return q.front(); }
    bool empty() { return q.empty(); }
};
```

### 3️⃣ Time & Space Complexity

- **Push:** O(N)
- **Pop:** O(1)
- **Space:** O(N)

---

## 7. Implement Queue Using Two Stacks

### 1️⃣ Approach / Intuition

- Use two stacks:
  - `s1` for push
  - `s2` for pop/peek
- Transfer elements only when `s2` is empty (amortized optimization)

### 2️⃣ Code

```cpp
class MyQueue {
public:
    stack<int> s1, s2;

    void push(int x) { s1.push(x); }

    int pop() {
        if (s2.empty()) {
            while (!s1.empty()) {
                s2.push(s1.top());
                s1.pop();
            }
        }
        int x = s2.top();
        s2.pop();
        return x;
    }

    int peek() {
        if (s2.empty()) {
            while (!s1.empty()) {
                s2.push(s1.top());
                s1.pop();
            }
        }
        return s2.top();
    }

    bool empty() {
        return s1.empty() && s2.empty();
    }
};
```

### 3️⃣ Time & Space Complexity

- **Push:** O(1)
- **Pop:** Amortized O(1)
- **Space:** O(N)

---

✅ These implementations cover **all classical stack & queue variations** asked in interviews.

