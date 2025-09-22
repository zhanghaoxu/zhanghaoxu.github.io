---
title: 02-JavaScript this的绑定 详解
---

JavaScript 中的 `this` 是一个特殊关键字，它代表函数执行时的上下文对象，其值取决于函数的调用方式，而非定义位置。理解 `this` 的绑定规则对于编写可预测和可维护的 JavaScript 代码至关重要。

<!--more-->

### `this` 的主要绑定规则：

| 绑定规则     | 调用方式                          | `this` 指向                                                          | 示例代码简要说明                                       |
| :----------- | :-------------------------------- | :------------------------------------------------------------------- | :----------------------------------------------------- |
| **默认绑定** | 独立函数调用                      | 非严格模式：全局对象（浏览器中为 `window`）<br>严格模式：`undefined` | `function foo() { console.log(this); } foo();`         |
| **隐式绑定** | 作为对象方法调用                  | **调用该方法的对象**                                                 | `obj.foo();` // `foo` 中的 `this` 指向 `obj`           |
| **显式绑定** | 使用 `call`, `apply`, `bind` 调用 | **指定的对象**                                                       | `foo.call(obj);` // `foo` 中的 `this` 指向 `obj`       |
| **new 绑定** | 使用 `new` 关键字调用构造函数     | **新创建的实例对象**                                                 | `let inst = new Foo();` // Foo 中的 `this` 指向 `inst` |
| **箭头函数** | 箭头函数内使用 `this`             | **继承其定义时所在外层（词法）作用域的 `this`**                      | `const bar = () => { console.log(this); }`             |

---

### 🔍 详细绑定规则

#### 1. 默认绑定（Default Binding）

当函数被独立调用，不带有任何上下文对象时，发生默认绑定。

- **非严格模式**：`this` 指向全局对象（浏览器中为 `window`，Node.js 中为 `global`）。
  ```javascript
  function show() {
    console.log(this);
  }
  show(); // 在浏览器非严格模式下输出 Window 对象
  ```
- **严格模式**（`'use strict'`）：`this` 为 `undefined`。
  ```javascript
  function showStrict() {
    "use strict";
    console.log(this);
  }
  showStrict(); // undefined
  ```

#### 2. 隐式绑定（Implicit Binding）

当函数作为某个对象的方法被调用时，`this` 会绑定到**调用该方法的对象**上。

```javascript
const person = {
  name: "Alice",
  greet: function () {
    console.log(`Hello, ${this.name}!`);
  },
};
person.greet(); // Hello, Alice!  this 指向 person 对象
```

**隐式丢失**：将对象方法赋值给一个变量再调用，或作为回调函数传递时，容易发生“隐式丢失”，`this` 会回退到默认绑定。

```javascript
const greetFunc = person.greet;
greetFunc(); // Hello, undefined! (非严格模式下，this 指向全局对象，全局对象上无 name 属性)

setTimeout(person.greet, 100); // Hello, undefined! 同理
```

#### 3. 显式绑定（Explicit Binding）

使用 `Function.prototype` 上的 `call`, `apply`, 或 `bind` 方法，可以显式地设置函数执行时 `this` 的值。

- **`call` 和 `apply`**：立即调用函数，并指定 `this` 和参数（`call` 接收参数列表，`apply` 接收参数数组）。
  ```javascript
  function introduce(lang, year) {
    console.log(`I code in ${lang} since ${year}. My name is ${this.name}`);
  }
  const dev = { name: "Bob" };
  introduce.call(dev, "JavaScript", 2015); // I code in JavaScript since 2015. My name is Bob
  introduce.apply(dev, ["Python", 2018]); // I code in Python since 2018. My name is Bob
  ```
- **`bind`**：创建一个新的函数，该函数的 `this` 永久绑定到指定的对象（**硬绑定**）。
  ```javascript
  const boundIntroduce = introduce.bind(dev, "Rust");
  boundIntroduce(2020); // I code in Rust since 2020. My name is Bob
  // 即使再次用 call 或 apply 试图改变 this，也不会生效
  ```

#### 4. new 绑定（new Binding）

使用 `new` 关键字调用构造函数（或 ES6 的类）时，`this` 会绑定到**新创建的对象实例**上。

```javascript
function Person(name) {
  this.name = name;
  // 构造函数中的 this 指向新实例
}
const john = new Person("John");
console.log(john.name); // John
```

#### 5. 箭头函数（Arrow Functions）

箭头函数**没有自己的 `this` 绑定**。它内部的 `this` **继承自其定义时所在的外层（词法）作用域**的值，并且在定义后就不会改变，即使使用 `call`, `apply`, `bind` 或作为其他对象的方法调用也是如此。

```javascript
const obj = {
  value: 42,
  regularFunc: function () {
    console.log(this.value); // 42, this 指向 obj
  },
  arrowFunc: () => {
    console.log(this.value); // undefined, this 继承自外层（可能是全局作用域）
  },
};
obj.regularFunc();
obj.arrowFunc();

// 箭头函数在解决回调中 this 丢失问题时非常有用
const obj2 = {
  name: "Charlie",
  getNameLater: function () {
    setTimeout(() => {
      console.log(this.name); // Charlie, 箭头函数继承了 getNameLater 的 this (即 obj2)
    }, 100);
  },
};
obj2.getNameLater(); //
```

---

### ⚠️ 注意事项与技巧

1.  **API 的 `thisArg` 参数**：许多数组方法（如 `forEach`, `map`, `filter`）以及 `setTimeout` 等接受一个可选的 `thisArg` 参数，用于指定回调函数内的 `this`。

    ```javascript
    const numbers = [1, 2, 3];
    const counter = { count: 0 };
    numbers.forEach(function (num) {
      this.count += num;
    }, counter); // 将 counter 对象作为 thisArg
    console.log(counter.count); // 6
    ```

2.  **忽略的 `this`**：如果你不关心函数内的 `this` 指向（例如只使用其参数），可以传入 `null` 或 `undefined` 作为 `call`, `apply`, `bind` 的第一个参数。在非严格模式下，函数内的 `this` 会指向全局对象；在严格模式下则为 `null` 或 `undefined`。

3.  **类中的 `this`**：在 ES6 类中，构造函数和方法中的 `this` 通常指向类的实例。但类方法若被提取出来单独调用，同样会遇到 `this` 丢失的问题。可以在构造函数中使用 `.bind(this)`，或使用类字段和箭头函数来定义方法，以自动绑定实例到 `this`（但需注意每个实例都会创建新函数副本的内存影响）。

---

### 💎 总结与优先级

`this` 绑定的优先级通常为：**new 绑定 > 显式绑定 > 隐式绑定 > 默认绑定**。箭头函数的 `this` 由词法作用域决定，无法被更改，其“优先级”实为静态查找。

| 场景                                                  | `this` 指向建议                                        |
| :---------------------------------------------------- | :----------------------------------------------------- |
| 直接调用独立函数                                      | 小心非严格模式的全局对象，尽量用严格模式或避免此种方式 |
| 调用对象方法                                          | 通常是该对象，注意隐式丢失风险                         |
| 需要使用特定上下文调用函数                            | 显式使用 `call`, `apply`, `bind`                       |
| 构造函数调用                                          | 新创建的实例对象                                       |
| 回调函数、事件处理器等需要固定上下文或使用外部 `this` | 考虑使用箭头函数或 `bind`                              |
| 需要定义不绑定自身 `this` 的函数                      | 使用箭头函数                                           |

希望这份详细的介绍能帮助你更好地理解和运用 JavaScript 中的 `this`。
