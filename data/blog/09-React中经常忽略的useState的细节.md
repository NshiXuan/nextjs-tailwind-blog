---
title: 'React中经常忽略的useState的细节' # 标题
date: '2023/3/17' # 发布时间
# lastmod: '2022/3/10'
tags: [React] # 标签
draft: false
summary: 'React中经常忽略的useState的细节'
images: # 文章封面 必须要
  [
    'https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/569ca2d5651844fb8001a3df9e71ee08~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

当动态设置 `useState` 的值时， `useState` 的值有这样的变化，第一次的值为 `undefinded` ，第二次为请求获取到的数据或者从 `redux` 中获取的数据，但是 `useState` 并没有拿到第二次的值，**因为 `useState` 只对第一个传入的值有效 后面传天王老子也没有用**

解决方法：

- 可以通过 `useEffect(()=>{},[useState的值])` ，来监听 `useState` 的变化在重新赋值， 不过当 `useState` 变化时会造成组件多渲染一次
- 推荐方法：在保证第一次有值的情况下在设置给 `useState`

```ts
// 如果子组件的setState需要用到discountInfo中的值 确保discountInfo存在再传递
{
  discountInfo && <HomeSectionV1 homeSectionData={discountInfo} />
}
```

```ts
// 1.如果是在子组件中使用useState 给父组件一个判断useState的初始值存在再展示子组件
// 2.也可以通过useEffect(()=>{},[initData]) 不过当initData变化时会造成组件多渲染一次
const [name, setName] = useState<NamesType>(homeSectionData?.dest_address[0].name as NamesType)
```
