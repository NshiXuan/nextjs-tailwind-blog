---
title: 'react实现高亮active的两种方式' # 标题
date: '2023/3/16' # 发布时间
# lastmod: '2022/3/10'
tags: [tailwindcss] # 标签
draft: false
summary: 'react添加高亮的两种实现方式'
images: # 文章封面 必须要
  ['']
layout: PostLayout
---

前端开发中经常要给样式添加高亮，这里提供了 `react` 添加高亮的两种实现方式

## 通过 class 实现

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2b5e9ca3cfa4c118cc0cf008376c0ba~tplv-k3u1fbpfcp-watermark.image?)
安装第三方包

```css
pnpm add classnames --save
```

定义样式

```css
.active {
  @apply bg-secondary-color text-white;
}
```

使用

```ts
...
export interface IProps {
  tabNames: string[] | undefined
}

// memo浅层比较
const SectionTabs: FC<IProps> = memo(function (props) {
  const { tabNames } = props

  // 定义选中的标签
  const [currentIndex, setCurrentIndex] = useState(1)

  // item点击
  function itemClickHandle(index: number) {
    setCurrentIndex(index)
  }

  return (
    <div className="flex">
      {tabNames?.map((item, index) => {
        return (
          <div
            className={classNames(...',
              { active: currentIndex === index }
            )}
            key={item}
            onClick={(e) => itemClickHandle(index)}
          >
            {item}
          </div>
        )
      })}
    </div>
  )
})

```

## 通过 style 实现

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2b5e9ca3cfa4c118cc0cf008376c0ba~tplv-k3u1fbpfcp-watermark.image?)

```ts
...
export interface IProps {
  tabNames: string[] | undefined
}

// memo浅层比较
const SectionTabs: FC<IProps> = memo(function (props) {
  const { tabNames } = props

  // 定义选中的标签
  const [currentIndex, setCurrentIndex] = useState(1)

  // item点击
  function itemClickHandle(index: number) {
    setCurrentIndex(index)
  }

  return (
    <div className="flex">
      {tabNames?.map((item, index) => {
        return (
          <div
            className="..."
            style={
              currentIndex === index
                ? { background: '#00848A', color: '#fff' }
                : {}
            }
            key={item}
            onClick={(e) => itemClickHandle(index)}
          >
            {item}
          </div>
        )
      })}
    </div>
  )
})

```
