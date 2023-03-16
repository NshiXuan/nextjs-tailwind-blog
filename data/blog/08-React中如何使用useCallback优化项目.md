---
title: 'React中useCallback如何优化项目' # 标题
date: '2023/3/16' # 发布时间
# lastmod: '2022/3/10'
tags: [tailwindcss] # 标签
draft: false
summary: 'React中useCallback如何优化项目'
images: # 文章封面 必须要
  [
    'https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/569ca2d5651844fb8001a3df9e71ee08~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

`React` 中提供了一些优化的 `Hook` ，我们该在什么时候使用它们呢？

这里介绍了 `useCallback` 如何优化 `React` 项目

`useCallback` 主要是用来优化父组件给子组件传递的方法

如下：父组件中把函数 `hotTabClickHandle` 传递给了子组件 `SectionTabs`

- 在 SPA 应用中每次切换路由回到 `Home` 页面，会生成一个新的 `hotTabClickHandle` 方法导致子组件 `SectionTabs` 被重新渲染一次，我们可以使用 `useCallback` 中阻止这个行为
- 当然也可以在 `[]` 中定义依赖，当依赖改变的时候在重新渲染子组件
- 在 `SSR` 中不用考虑这个问题，因为它都会重新请求一个页面

```ts
...

export default function Home(props: IProps) {
  ...

  // 定义传递给子组件的方法
  const hotTabClickHandle = useCallback((index: number, name: NamesType) => {
    setHotName(name)
  }, [])

  return (
    <>
      ...
            <SectionTabs tabClick={hotTabClickHandle} />
      ...
    </>
  )
}
```
