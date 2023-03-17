---
title: 'React子组件给父组件传递数据' # 标题
date: '2023/3/16' # 发布时间
# lastmod: '2022/3/10'
tags: [React] # 标签
draft: false
summary: 'React子组件给父组件传递数据'
images: # 文章封面 必须要
  [
    'https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/569ca2d5651844fb8001a3df9e71ee08~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

前端开发经常需要子组件给父组件传递数据，这里展示 `React` 中子组件如何给父组件传递数据
如下：点击选项把 `name` ， `index` 传递给父组件

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68bf1066091f4857bdc5462b23d8ac6d~tplv-k3u1fbpfcp-watermark.image?)
思路：在父组件中定义 `tabClick` 并传递一个函数给它，在子组件的 `props` 获取到 `tabClick` 调用传参即可

父组件

```ts
...

export default function Home(props: IProps) {
  ...

  // tab点击事件
  // 没次Home组件重新渲染 就会定义一个新的tabClickHandle方法 导致使用的子组件会重新渲染
  // 我们可以使用useCallback来优化这个问题
  const hotTabClickHandle = useCallback((index: number, name: NamesType) => {
    console.log(index, name)
    setHotName(name)
  }, [])

  return (
    <>
      <main>
        ...
            {/* 选项卡 */}
            <SectionTabs ... tabClick={hotTabClickHandle} />
        ...
      </main>
    </>
  )
}
```

子组件

```ts
...
export interface IProps {
  tabNames: string[] | undefined
  tabClick: (index: number, name: NamesType) => void
}

// memo浅层比较
const SectionTabs: FC<IProps> = memo(function (props) {
  const { tabNames, tabClick } = props

  ...

  // item点击
  function itemClickHandle(index: number, name: string) {
    setCurrentIndex(index)
    tabClick(index, name as NamesType)
  }

  return (
    <div className="flex   ">
      {tabNames?.map((item, index) => {
        return (
          <div
            ...
            onClick={(e) => itemClickHandle(index, item)}
          >
            {item}
          </div>
        )
      })}
    </div>
  )
})
```
