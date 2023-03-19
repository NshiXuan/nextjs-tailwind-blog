---
title: 'React封装ScrollView组件' # 标题
date: '2023/3/19' # 发布时间
# lastmod: '2022/3/10'
tags: [React] # 标签
draft: false
summary: 'React封装ScrollView组件'
images: # 文章封面 必须要
  [
    'https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/569ca2d5651844fb8001a3df9e71ee08~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

思路：通过 Ref 获取 DOM 元素，进而获取元素的 `scrollWidth` 可滚动的宽度, `clientWidth` 占据的宽度,计算出可滚动距离用来设置是否显示滚动的 `icon` ，再获取对应的 `tab` 的下标和 `DOM` 元素的 `offsetLeft` 进行滚动

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d099e4519274497786a5bffa14ef3335~tplv-k3u1fbpfcp-watermark.image?)

```ts
import IconArrowLeft from '@/assets/svg/icon-arrow-left'
import IconArrowRight from '@/assets/svg/icon-arrow-right'
import classNames from 'classnames'
import { FC, memo } from 'react'
import { ReactNode, useEffect, useRef, useState } from 'react'

import style from './index.module.scss'

export interface IProps {
  children?: ReactNode
}

const ScrollView: FC<IProps> = memo(function (props) {
  const { children } = props

  // 1.定义是否显示滚动箭头icon
  const [showLeft, setShowLeft] = useState(false)
  const [showRight, setShowRight] = useState(false)

  // 2.定义获取滚动内容的ref
  const scrollContentRef = useRef<HTMLDivElement>(null)

  // 3.定义记录可滚动的距离
  const [totalDistance, settotalDistance] = useState(0)

  // 4.DOM加载完后通过ref获取可滚动距离
  useEffect(() => {
    // current可以拿到DOM
    // 4.1 获取scrollWidth代表可滚动的宽度
    const scrollWidth = scrollContentRef.current?.scrollWidth!
    // 4.2 获取clientWidth代表占据的宽度
    const clientWidth = scrollContentRef.current?.clientWidth!
    // 4.3 计算可滚动的距离
    const scrollDistance = scrollWidth - clientWidth

    // 4.4 记录可滚动的距离
    settotalDistance(scrollDistance)

    // 4.5 如果不能往右滚动了就隐藏右箭头
    setShowRight(scrollDistance > 0)
  }, [])

  // 5.点击箭头实现滚动
  // 5.1 记录当前需要滚动div的下标 默认为第一个
  const [posIndex, setPosIndex] = useState(0)
  function controlClickHandle(isRight: boolean) {
    // 5.2判断是否左滚动还是右滚动
    const newIndex = isRight ? posIndex + 1 : posIndex - 1

    // 5.3获取滚动内容的div
    const childrenDiv = scrollContentRef.current?.children[newIndex] as HTMLDivElement

    // 5.4通过内容的div获取距离父盒子的左距离offsetLeft作为滚动距离
    // (左边距离父盒子的宽度相对于上一级的定位元素/table、td、th 如果没有定位元素则为body 详细看MDN)
    const offsetLeft = childrenDiv.offsetLeft

    // 5.5通过ref的style实现滚动
    scrollContentRef.current!.style.transform = `translate(-${offsetLeft}px)`

    // 5.6设置下一个滚动div的下标
    setPosIndex(newIndex)

    // 5.7是否继续显示右边的按钮
    // 如果滚动距离大于左边第一个盒子距离父盒子的左距离 显示右边滚动icon
    setShowRight(totalDistance > offsetLeft)
    // 如果左边第一个盒子距离父盒子的左距离大于0 才能向左滚动
    setShowLeft(offsetLeft > 0)
  }

  return (
    <div className="relative ">
      {/* 箭头Icon */}
      {showLeft && (
        <div
          className={classNames('left-0 top-1/2 w-fit', style.control, style.left)}
          onClick={(e) => controlClickHandle(false)}
        >
          <IconArrowLeft />
        </div>
      )}
      {showRight && (
        <div
          className={classNames('right-0 top-1/2 w-fit', style.control, style.right)}
          onClick={(e) => controlClickHandle(true)}
        >
          <IconArrowRight />
        </div>
      )}

      {/* 插槽 */}
      {/* 添加动画 */}
      <div className="relative overflow-hidden">
        <div className="duration-250 flex transition " ref={scrollContentRef}>
          {children}
        </div>
      </div>
    </div>
  )
})

export default ScrollView

// 设置一个方便调试的name 可以不写 默认为组件名称
ScrollView.displayName = 'ScrollView'
```

`index.module.scss`

```css
.control {
  @apply absolute z-10 flex h-[28px] w-[28px] items-center justify-center rounded-full border bg-white text-center  shadow-sm;
}

.left {
  @apply left-0 top-1/2 cursor-pointer;
  transform: translate(-50%, -50%);
}

.right {
  @apply right-0 top-1/2 cursor-pointer;
  transform: translate(50%, -50%);
}
```

- 使用

```ts
<ScrollView>
  <div className=" mr-4 min-w-[300px] border py-[14px]">123</div>
  <div className=" mr-4 min-w-[300px] border py-[14px]">123</div>
  <div className=" mr-4 min-w-[300px] border py-[14px]">123</div>
  <div className=" mr-4 min-w-[300px] border py-[14px]">123</div>
  <div className=" mr-4 min-w-[300px] border py-[14px]">123</div>
</ScrollView>
```
