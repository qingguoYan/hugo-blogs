---
title: "JS学习经验总结"
description: "分享日常开发常犯的JS错误或者经验，并提供了实用的解决方案"
date: 2024-12-07
image:
draft: false
tags: [javascript]
categories: [JS]
---

## 对象或数组解析默认值仅对 undefined 有效

只有解析结果严格等于 undefined，默认值才会生效

- name 与 tags 解析结果为 undefined，默认赋值成功

```js
const params = { name: undefined, tags: undefined };
const { name = "hello", tags = [] } = params;
console.log(name); // hello
console.log(tags); // []
```

- name 与 tags 解析结果为 null，默认赋值失败

```js
const params = { name: null, tags: null };
const { name = "hello", tags = [] } = params;
console.log(name); // null
console.log(tags); // null
```

- 解决方案：使用值的时候 使用 ?? 设置默认值， ?? 在值为 null 或者 undefined 的时候生效

```js
const params = { name: null, tags: null };
const _name = params.name ?? "hello";
const _tags = params.tags ?? [];
console.log(_name); // hello
console.log(_tags); // []
```

## 使用事件委托

将事件监听器附加到多个 DOM 元素可能效率低下。事件委托允许你在 DOM 的更高级别处理事件

不要这样做

```js
const buttons = document.querySelectorAll(".myButton");
buttons.forEach((button) => {
  button.addEventListener("click", function () {
    // Handle click
  });
});
```

利用事件委托,这样做更好

```js
document.body.addEventListener("click", function (event) {
  if (event.target.classList.contains("myButton")) {
    // Handle click
  }
});
```

## 永远不要忘记移除 DOM 事件监听器

为什么必须移除事件监听器?

1. 内存泄漏风险：未移除的事件监听器会阻止相关 DOM 元素和 JavaScript 对象被垃圾回收
2. 性能问题：残留的监听器会持续响应事件，造成不必要的处理开销
3. 意外的 bug: 在元素已移除或页面状态改变后，旧监听器可能触发不期望的操作

```js
const handler = () => {
  console.log("Button clicked");
};

// 添加监听器
element.addEventListener("click", handler);

// 移除监听器（必须使用相同的引用）
element.removeEventListener("click", handler);
```
