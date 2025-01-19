---
title: "React Application Performance Optimization: Five Simple and Effective Methods"
description: "Five tricks to improve the performance of React applications, helping you optimize components and reduce rendering, making the application run more smoothly."
date: 2024-12-26
image: 
draft: false
tags: [React]
categories: ["Performance Optimization"]
---

## Reasonable Component Structure
In React, a reasonable component structure can effectively enhance performance and avoid unnecessary re-renders. Below is an analysis and optimization summary of a simple example.

```js
// Parent Component
const App = () => {
  const [value, setValue] = useState();
  return (
    <>
      <input value={value} onChange={(e) => setValue(e.target.value)} />
      <Child />
    </>
  );
};

// Child Component
const Child = () => {
 // Assume rendering the child component takes a long time
  for( let i = 0; i < 20000;i++){
    console.log(i);
  }
  return <div>I am the Child component, rendering takes a long time</div>;
};
```

When the input value changes, the App component re-renders, causing the Child component to also re-render, which is very time-consuming. We find that Child has no relation to the input value, so we can separate the input into its own component to avoid re-rendering Child.

```js
// Input Component
const Input = () => {
  const [value, setValue] = useState();
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
};

// Parent Component
const App = () => {
  return (
    <>
      <Input />
      <Child />
    </>
  );
};
```

Alternatively, we can pass the Child component as children to the parent component, so it won't re-render when the value updates.

```js
// Parent Component
const AppWrap = ({children}) => {
  const [value, setValue] = useState();
  return (
    <>
      <input value={value} onChange={(e) => setValue(e.target.value)} />
      {children}
    </>
  );
};
// When using App
<AppWrap>
    <Child />
</AppWrap>
```
Through these two optimization methods, we can effectively isolate the rendering logic of the input component and the Child component, thus improving the application's performance and user experience.

Choosing the right component structure and passing method is an important consideration in React development.

## Using API to Reduce Component Rendering

### shouldComponentUpdate

shouldComponentUpdate is a method in the React component lifecycle used to control component re-rendering. It is called when the component receives new props or state, returning a boolean value indicating whether the component should update.

```js
class Child extends React.Component {
    state = {}

    shouldComponentUpdate(nextProps,nexState){
         // Compare next state and props
        // Return true to allow the component to update
        return true;
    }

    render (){
         // Assume rendering the child component takes a long time
        for( let i = 0; i < 20000;i++){
          console.log(i);
        }
        return <div>I am the Child component, rendering takes a long time</div>;
    }
}
```

When the rendering performance of a component is crucial, using shouldComponentUpdate can prevent unnecessary rendering.

### React.memo
React.memo is a higher-order component provided by React to optimize the performance of functional components. It significantly improves application performance by avoiding unnecessary re-renders, especially in the following cases:

- The component tree is large.
- The cost of rendering the component is high.

#### Default Behavior

React.memo uses shallow comparison of props by default to determine whether the component should re-render.

```js
const MemoChild = React.memo(Child)
```
#### Custom Comparison Function
If the props are complex objects or arrays, you may need to provide a custom comparison function. When using a custom comparison function with React.memo, the return value has the following meanings:

- Return true: Indicates that the previous and current props are the same, and the component will not re-render.
- Return false: Indicates that the previous and current props are different, and the component will re-render.

```js
const MemoChild = React.memo(Child,(preProps,nextProps)=>{
    return prevProps.value === nextProps.value; // Custom comparison logic      
})
```

## useDeferredValue
Refer to the official documentation: [useDeferredValue](https://react.docschina.org/reference/react/useDeferredValue)

In certain scenarios, we may need to pass the value of an input box to a child component, causing the child component to re-render every time the parent component updates. This can affect user experience, especially when the child component renders slowly. To solve this problem, we can use the useDeferredValue hook.

Consider the following code：
```js
import React, { useState } from 'react';

// Parent Component
const App = () => {
  const [value, setValue] = useState("");

  return (
    <>
      <input value={value} onChange={e => setValue(e.target.value)} />
      <Child value={value} />
    </>
  );
}

// ChildComponent
const Child = ({ value }) => {
  const items = Array.from({ length: 250 }, (_, index) => (
    <SlowItem key={index} text={value} />
  ));

  return <div>{items}</div>;
}

export default React.memo(Child); // Optimize using React.memo

function SlowItem({ text }) {
  let startTime = performance.now();
  while (performance.now() - startTime < 1) {
    // Simulate slow operation
  }

  return <li className="item">Text: {text}</li>;
}

```
In this example, the value of our input box is passed to the Child component via props. Every time the input box content changes, the Child component re-renders, leading to performance issues.

### Debouncing or Throttling

To solve this problem, we can use debouncing or throttling techniques. In this case, we can set a childValue state to delay the rendering of the Child component.

```js
 import _ from 'lodash';

// Parent Component
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

The value of the input box will update first, and then the Child component will update after 200ms. Using debouncing or throttling only partially optimizes the input box experience, as the input box may still feel unresponsive during the rendering of the Child component.

**Because during the rendering of the Child component, if input operations continue, the rendering of the Child component occupies the JS engine, and the input box events will not respond. This means that during the next input, the ongoing rendering of the Child component will not be interrupted to execute higher-priority input event tasks**.

At this point, we can use useDeferredValue to solve this problem.

### Using useDeferredValue

useDeferredValue accepts a deferred value. When the deferred value changes, it first renders using the old value, and then re-renders in the background using the new value. In other words, React will render the component twice: once to display the old result and once to display the new result.

**The key point is that the background rendering can be interrupted, meaning that when there are higher-priority state updates, the rendering of the Child component can be interrupted by rendering the input box, resulting in a smoother experience compared to debouncing and throttling**.

```js
// Parent Component
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
Official documentation： [useTransition](https://react.docschina.org/reference/react/useTransition)

useTransition is a hook introduced in React 18 to handle the transition state of UI updates. It allows you to mark certain updates as “non-urgent,” so React can prioritize handling user input and other urgent updates while processing these non-urgent updates in the background, improving application responsiveness and user experience.

In this example, we set an intermediate value childValue and update it as a transition update. When the value of the input box updates, the transition update will be interrupted, and after the input box updates, the transition update will execute again.

```js
// Parent Component
function App() {
  const [value, setValue] = useState("");
  const [childValue, setChildValue] = useState("");
  const [isPending, startTransition] = useTransition();
  
  return (
    <>
      <input value={value} onChange={(e)=>{
        setValue(e.target.value)
        startTransition(()=>{
          // This is a non-urgent update that can be interrupted by higher-priority tasks
          setChildValue(e.target.value);
        })
      }}/>
      {isPending ? <div>Calculating transition...</div> : <Child value={childValue}/>}
    </>
  );
}
```

## Upgrade to React 18 or Above

**Starting from React 18, React provides automatic batching and concurrent rendering**。

### Automatic Batching

In earlier versions of React, updates could only be batched in synthetic events (such as click events). This meant that if you performed multiple state updates in an asynchronous function (like a Promise or setTimeout), React would process these updates one by one, leading to multiple renders.

React 18's automatic batching allows multiple state updates to be automatically merged anywhere (including in asynchronous functions). This feature reduces the number of renders, thus improving performance. For example:

```js
// In React 18, batch updates
setTimeout(() => {
  setCount(count + 1);
  setName('New Name');
}, 1000);
```

### Concurrent Rendering

Concurrent rendering is another important feature of React 18, allowing React to handle updates more flexibly, thus enhancing the smoothness of the user interface. Suppose there's a Child component that takes about 200ms to render, but the rendering time for each frame in the browser is typically about 16ms. During the rendering of the Child component, the page will not respond to user interactions.

#### Fiber Nodes and Task Management

Each JSX element is converted into a corresponding Fiber node, and each Fiber node has its own task. For example:

**Host nodes (like div, span): create corresponding real DOM nodes and set styles and attributes**。

**Custom components (like Child): manage the children within the component and handle their state and side effects**。

When the Child component contains a large number of nodes (for example, 10,000), React processes each Fiber node in a depth-first manner, which can take a lot of time. React 18's concurrent rendering solves this problem by breaking the rendering process into multiple small tasks.

#### Specific Implementation

In each frame, React reserves some time (for example, 5ms) to process the first 1,000 (assumed value) nodes of the Child component. As follows:

**First Frame:**

Process the first 1,000 nodes, taking about 5ms.
The remaining time (11ms) is used to respond to user clicks and input events, ensuring the page remains responsive.

**Subsequent Frames：**

If there are no user interactions, React continues processing the remaining 9,000 nodes in the next frame.
If user interactions occur (like clicks or input), React prioritizes handling these high-priority tasks.

**Task Resumption：**

After handling high-priority tasks, React resumes the rendering tasks of the Child component from the last interrupted point, continuing to process the remaining Fiber tasks.

In this way, React can allocate time across multiple frames, ensuring that the user interface remains smooth and responsive. Ultimately, the entire rendering task of the Child component will be completed in about 10 frames, without affecting the user interaction experience.
