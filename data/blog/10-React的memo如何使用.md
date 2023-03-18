---
title: 'React的memo如何使用' # 标题
date: '2023/3/18' # 发布时间
# lastmod: '2022/3/10'
tags: [React] # 标签
draft: false
summary: 'React的memo如何使用'
images: # 文章封面 必须要
  [
    'https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/569ca2d5651844fb8001a3df9e71ee08~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

对于 `React.memo` ，它可以做缓存优化，渲染子组件之前，对比子组件的 `props` ，发现 `props` 没有发生改变返回组件上一次渲染的值

> **`React.memo` 仅检查 `props` 变更**。如果函数组件被 `React.memo` 包裹，且其实现中拥有 `useState` ， `useReducer` 或 `useContext` 的 `Hook` ，当 `state` 或 `context` 发生变化时，它仍会重新渲染。

如下： 再点击父组件的按钮，子组件已经不会重新渲染了

```tsx
function ReactMemoDemo() {
  const [count, setCount] = React.useState(0)

  return (
    <div>
      <div>Parent Count: {count}</div>
      <button onClick={() => setCount((count) => count + 1)}>+</button>
      <Child name="Son" />
    </div>
  )
}

const Child = React.memo((props) => {
  console.log('子组件渲染了')
  return <p>Child Name: {props.name}</p>
})

render(<ReactMemoDemo />)
```

使用太多缓存，反而容易带来负优化，组件使用缓存策略后，在更新前会比较 `props` ，如果子组件参数多且复杂，也会比较消耗性能，甚至有可能超过 `虚拟DOM` 的生成，当然大部分情况下还是 `虚拟DOM` 计算更消耗性能

也因此，在 React 社区中，开发者们也一致的认为，不必要的情况下，不需要使用  `React.memo`

什么时候该用？

- 组件渲染过程特别消耗性能，以至于能感觉到到，比如：长列表、图表等
- 确认子组件的 `props` 不会改变且可能与其他组件操作重新渲染的子组件

什么时候不该用？

- 组件参数结构十分庞大复杂，比如未知层级的对象，或者列表（城市，用户名）等
