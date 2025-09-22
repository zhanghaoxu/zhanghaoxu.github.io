---
title: 04-JavaScript 立即调用函数表达式IIFE 详解
---

立即调用函数表达式（IIFE，Immediately Invoked Function Expression）是 JavaScript 中一种在定义时就会立即执行的函数模式。它主要用于**创建独立的作用域**，从而避免变量污染全局命名空间。

<!--more-->

下面是一个简单的 IIFE 示例：

```javascript
(function () {
  console.log("This is an IIFE");
})();
```

### 🔍 IIFE 的核心语法与变体

IIFE 的核心思路是将函数定义包裹在括号内，使其成为**表达式**，然后通过紧随其后的 `()` 立即执行它。

- **经典写法**：最常见的形式是用括号将整个函数定义包裹起来。

  ```javascript
  (function () {
    // 代码逻辑
  })();

  // 也可以将执行括号放在外面
  (function () {
    // 代码逻辑
  })();
  ```

- **使用运算符的变体**：利用一元运算符（如 `!`, `+`, `-`, `~` 等）也可以将函数声明转换为表达式。

  ```javascript
  !(function () {
    console.log("IIFE with !");
  })();

  +(function () {
    console.log("IIFE with +");
  })();
  ```

- **箭头函数形式**（ES6+）：箭头函数也可以用于构建 IIFE，使语法更简洁。
  ```javascript
  (() => {
    console.log("IIFE with arrow function");
  })();
  ```
- **传递参数**：你可以像普通函数一样向 IIFE 传递参数。
  ```javascript
  (function (name) {
    console.log("Hello, " + name);
  })("World"); // 输出 "Hello, World"
  ```

### 🛠️ IIFE 的主要应用场景

IIFE 在过去（ES6 之前）的 JavaScript 开发中极为重要，以下是它的几个关键应用场景：

1.  **创建独立作用域与模块化（ES6 前）**：这是 IIFE 最核心的用途。在 ES6 引入模块系统之前，IIFE 通过**闭包**模拟模块化，隐藏私有变量，只暴露公共接口。

    ```javascript
    var myModule = (function () {
      // 私有变量
      var privateVar = "I am private";

      // 私有方法
      function privateMethod() {
        console.log(privateVar);
      }

      // 返回公共接口
      return {
        publicMethod: function () {
          privateMethod(); // 公共方法可以访问私有成员
        },
      };
    })();

    myModule.publicMethod(); // 输出 "I am private"
    console.log(myModule.privateVar); // undefined, 无法直接访问私有变量
    ```

2.  **解决循环中的变量作用域问题（针对 `var`）**：在使用 `var` 的循环中创建异步回调（如 `setTimeout`）时，IIFE 可以**捕获每次迭代的状态**。

    ```javascript
    // 不使用 IIFE 时的问题（使用 var）
    for (var i = 0; i < 5; i++) {
      setTimeout(function () {
        console.log(i); // 输出 5, 5, 5, 5, 5
      }, 100);
    }

    // 使用 IIFE 解决（为每个迭代创建新的函数作用域）
    for (var i = 0; i < 5; i++) {
      (function (j) {
        // 捕获当前迭代的 i 值
        setTimeout(function () {
          console.log(j); // 输出 0, 1, 2, 3, 4
        }, 100);
      })(i);
    }
    ```

    - **现代替代方案**：使用 ES6 的 `let` 声明变量，它天生具有块级作用域，能更优雅地解决此问题。

    ```javascript
    for (let i = 0; i < 5; i++) {
      // 使用 let
      setTimeout(function () {
        console.log(i); // 输出 0, 1, 2, 3, 4
      }, 100);
    }
    ```

3.  **初始化代码与避免全局污染**：IIFE 常用于封装一些初始化脚本，确保其中定义的变量不会泄漏到全局。

    ```javascript
    (function () {
      var initData = loadInitialData(); // 该变量不会污染全局
      setupConfig(initData);
      console.log("Initialization complete.");
    })();
    // console.log(initData); // ReferenceError: initData is not defined
    ```

4.  **依赖注入与封装私有状态**：IIFE 可以接收外部参数，有助于管理依赖和创建真正的私有成员。
    ```javascript
    (function ($, window) {
      // 在这里可以安全地使用 $ 和 window
      // 即使外部 jQuery 的别名不是 $ 或其他库覆盖了 $，这里也不受影响
      $("#button").click(function () {
        console.log("Clicked!");
      });
    })(jQuery, this); // 将外部依赖作为参数传入
    ```

### ⚖️ IIFE 的优缺点与现代替代方案

| 优点/特点                          | 缺点/注意事项                                  |
| :--------------------------------- | :--------------------------------------------- |
| 立即执行，封装初始化代码           | **无法手动再次调用**（除非返回内部函数并保存） |
| 创建独立作用域，**避免全局污染**   | 语法对初学者可能有些怪异                       |
| 通过闭包实现**私有变量**（ES6 前） | 调试时匿名函数可能增加栈追踪难度               |
| 在 ES6 之前是重要的**模块化**手段  |                                                |

随着 ES6（ES2015）及后续版本的普及，IIFE 的许多传统用途有了更现代的替代方案：

- **块级作用域 (`let`/`const`)**：`let` 和 `const` 提供了天然的块级作用域，无需 IIFE 来创建临时作用域。
  ```javascript
  {
    let scopedVar = "I am scoped";
    console.log(scopedVar); // 正常工作
  }
  // console.log(scopedVar); // ReferenceError
  ```
- **ES6 模块 (`import`/`export`)**：ES6 模块提供了官方的、标准化的模块方案，具有更清晰的依赖管理和更好的静态分析能力（如 Tree Shaking）。

  ```javascript
  // math.js
  export function add(a, b) {
    return a + b;
  }

  // main.js
  import { add } from "./math.js";
  console.log(add(2, 3)); // 5
  ```

### 💡 总结

IIFE 是 JavaScript 发展历程中一个**非常重要的模式**。虽然在现代开发中，由于块级作用域和 ES6 模块的出现，IIFE 的使用频率有所下降，但理解 IIFE 依然很有价值：

1.  **维护旧代码**：许多遗留项目大量使用 IIFE。
2.  **理解作用域和闭包**：IIFE 是学习 JavaScript 作用域、闭包和模块化概念的优秀范例。
3.  **特定场景仍有用武之地**：在某些需要立即执行的、隔离的初始化代码或快速封装中，IIFE 依然简单有效。

希望这些信息能帮助你更好地理解和使用 IIFE。
