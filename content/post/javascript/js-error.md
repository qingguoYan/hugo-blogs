---
title: "常见的JS错误及解决方案"
description: "分享日常开发常犯的JS错误，并提供了实用的解决方案。快来看看你是否也在犯这些错误吧"
date: 2024-12-07
image: 
draft: false
tags: [javascript]
categories: [JS]
---

## 对象或数组解析默认值仅对 undefined 有效

**只有解析结果严格等于undefined，默认值才会生效**

- name 与 tags 解析结果为undefined，默认赋值成功
```js
const params = { name: undefined, tags: undefined };
const { name = "hello", tags = [] } = params;
console.log(name); // hello
console.log(tags); // []
```
- name 与 tags 解析结果为null，默认赋值失败
```js
const params = { name: null, tags: null };
const { name = "hello", tags = [] } = params;
console.log(name); // null
console.log(tags); // null
```
- 解决方案：使用值的时候 使用 ?? 设置默认值， ?? 在值为null或者undefined的时候生效
```js
const params = { name: null, tags: null };
const _name = params.name ?? "hello";
const _tags = params.tags ?? [];
console.log(_name); // hello
console.log(_tags); // []
```