---
title: "React 应用性能优化：五个简单有效的方法"
description: "五个提升 React 应用性能的绝招，帮助你优化组件、减少渲染，让应用运行得更加顺畅。"
date: 2024-12-07
image: 
draft: false
tags: [React]
categories: ["性能优化"]
---

## 正确的组件结构
在 React 中，合理的组件结构可以有效提升性能，避免不必要的重新渲染。以下是对一个简单示例的分析与优化总结

```js
// 父组件
const App = () => {
  const [value, setValue] = useState();
  return (
    <>
      <input value={value} onChange={(e) => setValue(e.target.value)} />
      <Child />
    </>
  );
};

// 子组件
const Child = () => {
 // 假设子组件渲染需要很长时间
  for( let i = 0; i < 20000;i++){
    console.log(i);
  }
  return <div>我是Child组件, 渲染需要花费很长时间</div>;
};
```

当input改变值时，由于App重新组件渲染，Child组件也会重新渲染，但是Child渲染非常耗时。
我们发现Child与input的值并没有任何关联，所以我们可以将input单独写成一个组件，避免Child的重新渲染

```js
// Input组件
const Input = () => {
  const [value, setValue] = useState();
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
};

// 父组件
const App = () => {
  return (
    <>
      <Input />
      <Child />
    </>
  );
};
```

也可以通过第二种方式 将Child组件以Children方式传入父组件，value更新时Child也不会渲染

```js
// 父组件
const AppWrap = ({children}) => {
  const [value, setValue] = useState();
  return (
    <>
      <input value={value} onChange={(e) => setValue(e.target.value)} />
      {children}
    </>
  );
};
// 使用App时
<AppWrap>
    <Child />
</AppWrap>
```
通过以上两种优化方案，我们可以有效隔离 input 组件与 Child 组件的渲染逻辑，从而提升应用的性能和用户体验。选择合适的组件结构和传递方式是 React 开发中的重要考量。

## 减少组件渲染

### shouldComponentUpdate

shouldComponentUpdate 是 React 组件生命周期中的一个方法，用于控制组件的重新渲染。它在组件接收到新的 props 或 state 时被调用，返回一个布尔值，指示组件是否应该更新。

```js
class Child extends React.Component {
    state = {}

    shouldComponentUpdate(nextProps,nexState){
        // 对比下一次的state与props
        // 返回true允许组件更新
        return true;
    }

    render (){
        // 假设子组件渲染需要很长时间
        for( let i = 0; i < 20000;i++){
          console.log(i);
        }
        return <div>我是Child组件, 渲染需要花费很长时间</div>;
    }
}
```

当组件的渲染性能很重要时，使用 shouldComponentUpdate 可以避免不必要的渲染。

### React.memo
React.memo 是 React 提供的一种高阶组件，用于优化函数组件的性能。它通过避免不必要的渲染，显著提高应用的性能，尤其在以下情况下尤为有效：

- 组件树较大
- 组件渲染成本较高

#### 默认行为

React.memo默认使用浅比较props决定组件是否重新渲染

```js
const MemoChild = React.memo(Child)
```
#### 自定义比较函数
如果 props 是复杂对象或数组，可能需要提供自定义比较函数。在使用 React.memo 的自定义比较函数时，返回值的含义如下：

- 返回 true: 表示前后 props 相同，组件 不会重新渲染。
- 返回 false: 表示前后 props 不同，组件 会重新渲染。

```js
const MemoChild = React.memo(Child,(preProps,nextProps)=>{
    return prevProps.value === nextProps.value; // 自定义比较逻辑   
})
```

## useDeferredValue
参考官方文档：[useDeferredValue](https://react.docschina.org/reference/react/useDeferredValue)

在某些场景下，我们可能需要将输入框的值传递给子组件，导致子组件在每次父组件更新时都重新渲染。这样会影响用户体验，尤其是在子组件渲染较慢的情况下。为了解决这个问题，我们可以使用 useDeferredValue 钩子。

考虑以下代码：
```js
import React, { useState } from 'react';

// App父组件
const App = () => {
  const [value, setValue] = useState("");

  return (
    <>
      <input value={value} onChange={e => setValue(e.target.value)} />
      <Child value={value} />
    </>
  );
}

// Child组件
const Child = ({ value }) => {
  const items = Array.from({ length: 250 }, (_, index) => (
    <SlowItem key={index} text={value} />
  ));

  return <div>{items}</div>;
}

export default React.memo(Child); // 使用 React.memo 进行优化

function SlowItem({ text }) {
  let startTime = performance.now();
  while (performance.now() - startTime < 1) {
    // 模拟慢操作
  }

  return <li className="item">Text: {text}</li>;
}

```
在这个示例中，我们的输入框值通过 props 传递给 Child 组件。每当输入框内容变化时，Child 组件都会重新渲染，导致性能问题。

### 防抖或者节流

为了解决这个问题，我们可以使用防抖或节流技术。在这种情况下，我们可以设置一个 childValue 状态，用于延迟更新 Child 组件的渲染。

```js
 import _ from 'lodash';

// App父组件
const App = () => {
  const [value, setValue] = useState("");
  const [childValue, setChildValue] = useState("");

  const debounceUpdateChildValue = _.debounce((value) => {
    setChildValue(value);
  }, 200);

  return (
    <>
      <input value={value} onChange={e => {
        setValue(e.target.value);
        debounceUpdateChildValue(e.target.value);
      }} />
      <Child value={childValue} />
    </>
  );
}
```

输入框的值会先更新，200ms后Child组件更新，使用防抖或者节流的方式只是将Input框输入体验进行了部分优化，仍然会出现输入框不流畅的体验。

**因为在Child组件渲染时如果继续进行输入操作，Child组件渲染占用js引擎，输入框事件并不会响应。也就是说再次输入时，不会中断正在进行的Child组件渲染，来执行优先级更高的输入事件任务**。

这时就可以使用useDefrredValue解决这样的问题。

### 使用 useDeferredValue

useDefrredValue会接受一个延迟值，当延迟值改变时，会先使用旧的值进行渲染，然后在后台使用新值进行一次新的渲染，也就是说React会将组件渲染两次，一次展示旧的结果，一次展示新的结果。

**最关键的是，后台渲染可以被中断，这意味着当有更高级的状态更新时，可以先渲染更高级的任务。比如这个例子中的输入框值渲染可以中断后台进行的Child组件的渲染，体验效果就会比防抖和节流更丝滑。**

```js
// App父组件
function App() {
  const [value, setValue] = useState("");
  const deferredValue =  useDeferredValue(value);
  
  return (
    <>
      <input value={value} onChange={(e)=>{
        setValue(e.target.value);
      }}/>
      <Child value={deferredValue}/>
    </>
  );
}
```

## useTranstion
官方文档： [useTransition](https://react.docschina.org/reference/react/useTransition)

useTransition 是 React 18 引入的一个钩子，用于处理 UI 更新的过渡状态。它允许你标记某些更新为“非紧急”更新，这样 React 就可以优先处理用户输入和其他紧急更新，同时在后台处理这些非紧急更新，从而改善应用的响应能力和用户体验。

在这个例子中，我们设置一个中间值childValue，并将其更新为过渡更新。当输入框的值更新时，过渡更新会被中断，等待输入框更新后，过渡更新会再次执行。

```js
// 父组件
function App() {
  const [value, setValue] = useState("");
  const [childValue, setChildValue] = useState("");
  const [isPending, startTransition] = useTransition();
  
  return (
    <>
      <input value={value} onChange={(e)=>{
        setValue(e.target.value)
        startTransition(()=>{
          // 我是非紧急更新的，可以被更高级的优先级任务
          setChildValue(e.target.value);
        })
      }}/>
      {isPending ? <div>我在过渡计算中...</div> : <Child value={childValue}/>}
    </>
  );
}
```

## 升级到React18或者以上版本

**从React18开始，React提供了自动批处理和并发渲染**。

### 自动批处理

在 React 的早期版本中，更新只能在合成事件（如点击事件）中批量处理。这意味着，如果你在异步函数（例如 Promise 或 setTimeout）中进行多次状态更新，React 将会逐个处理这些更新，导致多次渲染。

React 18 的自动批处理允许在任何地方（包括异步函数）自动合并多次状态更新。这一特性减少了渲染次数，从而提升了性能。例如：

```js
// 在 React 18 中,批量更新
setTimeout(() => {
  setCount(count + 1);
  setName('New Name');
}, 1000);
```

### 并发渲染

并发渲染是 React 18 的另一项重要特性，它使得 React 能够更灵活地处理更新，从而提升用户界面的流畅性。假设有一个组件 Child，其渲染时间约为 200ms，但浏览器每帧的渲染时间通常约为 16ms。在 Child 组件渲染的过程中，页面将无法响应用户的交互。

#### Fiber 节点与任务管理

每个 JSX 元素都会转换为对应的 Fiber 节点，每个 Fiber 节点都有自己的任务。例如：

**Host 节点（如 div、span）：创建对应的真实 DOM 节点并设置样式和属性**。

**自定义组件（如 Child）：管理组件内的 children，并处理其状态和副作用**。

当 Child 组件包含大量节点（例如 10,000 个）时，React 会以深度遍历的方式处理每个 Fiber 节点，这会消耗大量时间。React 18 的并发渲染通过将渲染过程分解为多个小任务来解决这一问题。

#### 具体实现

在每一帧中，React 会预留一部分时间（例如 5ms）用于处理 Child 组件中的前 1,000(假设值) 个节点。如下所示：

**第一帧：**

处理前 1,000 个节点，约耗时 5ms。
剩余时间（11ms）用于响应用户的点击和输入事件，确保页面保持响应。

**后续帧：**

如果没有用户交互，React 会在下一帧继续处理剩余的 9,000 个节点。
如果出现用户交互（如点击或输入），React 会优先处理这些高优先级的任务。

**任务恢复：**

在处理完高优先级的任务后，React 会从上一次中断的地方恢复 Child 组件的渲染任务，继续处理剩余的 Fiber 任务。

通过这种方式，React 能够在多帧中分配时间，确保用户界面始终保持流畅和响应迅速。最终，整个 Child 组件的渲染任务将在大约 10 帧后完成，而用户的交互体验不会受到影响。
