---
title: 06-JavaScript  异步任务编程解决方案Promise 详解
---

Promise 是 ES6 (ECMAScript 2015) 引入的一种**用于处理异步任务**的标准化解决方案。它旨在更优雅地管理异步代码，解决传统的“回调地狱”（Callback Hell）问题，并提供更好的错误处理机制。

我们可以将 Promise 看作是异步任务的管理器，它对异步任务进行了统一的抽象。

### 🌟 异步任务的统一抽象

1. 状态和结果抽象：Promise 对异步任务状态和结果进行了统一的抽象，它有三种不可逆的状态：

   - **Pending（进行中）**：初始状态，表示异步操作尚未完成。
   - **Fulfilled（已成功）**：表示异步操作成功完成。此时会有一个不可变的**结果值（value）**。
   - **Rejected（已失败）**：表示异步操作失败。此时会有一个不可变的**失败原因（reason）**。

   状态一旦从 `Pending` 变为 `Fulfilled` 或 `Rejected`，就**不可再改变**。

2. 回调注册抽象：Promise 对异步任务的回调注册进行了统一的抽象和管理，无论异步操作成功还是失败，最终都可以通过注册回调函数来处理结果。

   - **`then（）`**：用于处理异步操作成功和失败的情况，它接收两个参数 **成功和失败回调函数（onFulfilled，onRejected）**，该函数会在 Promise 状态变为 `fulfilled`或 `rejected` 时被调用，并接收异步操作的结果值作为参数。

   - **`catch()`**：用于处理异步操作失败的情况，接收一个**失败回调函数（onRejected）**，该函数会在 Promise 状态变为 `rejected` 时被调用，并接收异步操作的失败原因作为参数,它是 `then(,onRejected)` 方法的语法糖，用于处理失败情况。

   - **`finally()`**：无论 Promise 状态最终是 `fulfilled` 还是 `rejected`，`finally()` 方法中的回调函数都会被调用。这使得 `finally()` 成为执行清理操作（如关闭文件、释放资源等）的好地方。

   通过以上方法注册的回调函数是异步微任务。

3. 任务间依赖抽象：Promise 对多个异步任务的依赖关系进行了抽象， `then()` 返回一个新的 Promise 对象，允许你在当前 Promise 成功后的回调中中继续处理下一个异步任务，这种操作被称为链式调用。通过链式调用，我们可以处理多个异步任务的结果，确保它们按照正确的顺序执行，实现复杂的异步任务编排。

### 执行器 （Executor）

执行器是 Promise 构造函数的参数，它是一个函数，用于执行异步操作并根据操作结果调用 `resolve` 或 `reject` 函数。执行器函数接收两个参数：

- **`resolve`**：当异步操作成功时调用，将 Promise 状态由 `pending` 改为 `fulfilled`，并可传递结果值。
- **`reject`**：当异步操作失败时调用，将 Promise 状态由 `pending` 改为 `rejected`，并可传递错误原因。

执行器本质上是我们的异步任务，它会在 Promise 构造函数内部被立即调用，并使用 `resolve` 或 `reject` 函数来改变 Promise 的状态和结果，与 Promise 对象进行交互。

### 🛠️ 基本用法

创建一个 Promise 需要使用 `new Promise()` 构造函数，并传入一个**执行器函数（executor）**。这个执行器函数接收两个由 JavaScript 引擎提供的函数作为参数：

- **`resolve`**：当异步操作成功时调用，将 Promise 状态由 `pending` 改为 `fulfilled`，并可传递结果值。
- **`reject`**：当异步操作失败时调用，将 Promise 状态由 `pending` 改为 `rejected`，并可传递错误原因。

```javascript
const myPromise = new Promise((resolve, reject) => {
  // 模拟一个异步操作，例如 API 请求
  setTimeout(() => {
    const success = Math.random() > 0.5; // 随机模拟成功或失败
    if (success) {
      resolve("操作成功！数据是：..."); // 将状态变为 fulfilled
    } else {
      reject(new Error("操作失败！")); // 将状态变为 rejected
    }
  }, 1000);
});
```

### 🔗 处理结果：`.then()`, `.catch()`, `.finally()`

创建 Promise 后，你可以通过以下方法处理其最终状态：

- **`.then()`**：接收两个**可选**的回调函数作为参数。第一个处理 `fulfilled` 状态和结果值，第二个处理 `rejected` 状态和失败原因。
  ```javascript
  myPromise.then(
    (value) => {
      console.log("成功：", value);
    }, // 在 promise 被解析时执行
    (error) => {
      console.error("失败：", error);
    } // 在 promise 被拒绝时执行
  );
  ```
- **`.catch()`**：专门用于捕获和处理 `rejected` 状态或链中抛出的错误。相当于 `.then(null, rejectionHandler)`。
  ```javascript
  myPromise
    .then((value) => {
      console.log("成功：", value);
    })
    .catch((error) => {
      console.error("捕获错误：", error);
    }); // 捕获 then 中的错误或之前的拒绝
  ```
- **`.finally()`**：无论 Promise 最终是 `fulfilled` 还是 `rejected`，都会执行的回调。常用于清理工作。
  ```javascript
  myPromise
    .then((value) => {
      console.log("成功：", value);
    })
    .catch((error) => {
      console.error("错误：", error);
    })
    .finally(() => {
      console.log("操作结束，无论成败。");
    });
  ```

### ➰ 链式调用（Chaining）

Promise 的 `.then()`, `.catch()`, `.finally()` 方法**都会返回一个新的 Promise**，这允许你将多个异步操作按顺序串联起来，形成**链式调用**。这是解决“回调地狱”的关键。

- 如果 `.then()` 中的回调函数：
  - **返回一个值**（包括 `undefined`），新返回的 Promise 将成为 **`fulfilled`** 状态，并将该值作为结果。
  - **抛出异常**，新返回的 Promise 将成为 **`rejected`** 状态，并将抛出的错误作为原因。
  - **返回另一个 Promise**，新返回的 Promise 将**采用**这个 Promise 的最终状态和结果。

```javascript
function fetchUserData(userId) {
  return new Promise((resolve) => {
    setTimeout(() => resolve({ id: userId, name: "Alice" }), 1000);
  });
}

function fetchUserPosts(userId) {
  return new Promise((resolve) => {
    setTimeout(() => resolve(["Post 1", "Post 2"]), 1000);
  });
}

fetchUserData(123)
  .then((user) => {
    console.log("获取用户成功：", user.name);
    return fetchUserPosts(user.id); // 返回一个新的 Promise
  })
  .then((posts) => {
    console.log("获取用户帖子成功：", posts);
    return posts.length; // 返回一个普通值
  })
  .then((numberOfPosts) => {
    console.log("帖子数量：", numberOfPosts);
  })
  .catch((error) => {
    // 链中任何地方的错误都会被这里捕获
    console.error("发生错误：", error);
  });
```

### 📦 并行处理多个异步操作

Promise 提供了几个静态方法来处理多个并行的异步操作：

1.  **`Promise.all(iterable)`**：

    - 接收一个 Promise 可迭代对象（如数组）。
    - 当所有输入的 Promise 都变为 `fulfilled` 时，它返回的 Promise 才变为 `fulfilled`，结果是一个包含所有成功结果的数组，**顺序与输入一致**。
    - 如果**任何一个**输入的 Promise 被 `rejected`，则立即 `rejected`，并返回第一个失败的原因。
    - **适用场景**：多个异步任务**全部成功**才能继续，且彼此不依赖。

2.  **`Promise.race(iterable)`**：

    - 接收一个 Promise 可迭代对象。
    - 只要输入的 Promise 中**有一个**状态变为 `fulfilled` 或 `rejected`，它返回的 Promise 就采用那个**最先改变状态**的 Promise 的状态和结果。
    - **适用场景**：为异步操作设置超时、竞争获取最快结果。

3.  **`Promise.allSettled(iterable)`** (ES2020)：

    - 接收一个 Promise 可迭代对象。
    - 等待所有输入的 Promise 都**完成**（无论是 `fulfilled` 还是 `rejected`）。
    - 返回一个 Promise，其结果是描述每个 Promise 最终状态和结果（或原因）的对象数组。
    - **适用场景**：需要知道所有异步操作的最终结果，无论成败。

4.  **`Promise.resolve(value)`** & **`Promise.reject(reason)`**：
    - 创建一個立即处于 `fulfilled`（已解决）或 `rejected`（已拒绝）状态的 Promise。
    - 常用于将值转换为 Promise，或开始一个链式调用。

```javascript
// Promise.all 示例
const promise1 = Promise.resolve("结果1");
const promise2 = new Promise((resolve) => setTimeout(resolve, 100, "结果2"));
const promise3 = 42; // 非Promise值会被 Promise.resolve 转换

Promise.all([promise1, promise2, promise3])
  .then((values) => {
    console.log(values); // 输出: ["结果1", "结果2", 42]
  })
  .catch((error) => {
    console.error(error);
  });

// Promise.race 示例（超时控制）
const dataPromise = fetchData(); // 假设这是一个耗时的请求
const timeoutPromise = new Promise((_, reject) =>
  setTimeout(() => reject(new Error("请求超时")), 5000)
);

Promise.race([dataPromise, timeoutPromise])
  .then((data) => console.log("数据获取成功:", data))
  .catch((error) => console.error("错误:", error.message));
```

### ⚠️ 常见陷阱与最佳实践

1.  **避免 Promise 地狱**：虽然 Promise 解决了回调地狱，但不恰当的链式调用仍可能形成嵌套，这时可结合 **`async/await`**（ES2017）使代码更清晰。
2.  **总是捕获错误**：使用 `.catch()` 或在 `async/await` 中使用 `try...catch` 来避免未处理的 Promise 拒绝。
3.  **在 `.then()` 中返回 Promise**：为了链式调用能正确传递状态和值，在 `.then()` 的回调中若进行异步操作，应返回新的 Promise。
4.  **`async/await` 是语法糖**：`async` 函数总是返回一个 Promise，`await` 用于等待一个 Promise 的解决。它们让异步代码看起来像同步代码，但不会阻塞主线程。

```javascript
// 使用 async/await 重写链式调用的例子
async function getUserInfo() {
  try {
    const user = await fetchUserData(123);
    console.log("获取用户成功：", user.name);
    const posts = await fetchUserPosts(user.id);
    console.log("获取用户帖子成功：", posts);
    const numberOfPosts = posts.length;
    console.log("帖子数量：", numberOfPosts);
    return numberOfPosts; //  async 函数返回的 Promise 会 resolve 为这个值
  } catch (error) {
    console.error("发生错误：", error);
    throw error; // 将错误继续抛出
  }
}

getUserInfo();
```

### 💎 总结

Promise 是现代 JavaScript 异步编程的基石。它通过**状态管理**、**链式调用**和**丰富的组合方法**（如 `Promise.all`），显著改善了异步代码的可读性、可维护性和错误处理能力。虽然 `async/await` 提供了更直观的语法，但它们底层依然依赖于 Promise。因此，深入理解 Promise 对于掌握 JavaScript 异步编程至关重要。
