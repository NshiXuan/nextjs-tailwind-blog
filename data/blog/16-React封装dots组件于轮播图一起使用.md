---
title: 'React封装dots组件于轮播图一起使用' # 标题
date: '2023/3/23' # 发布时间
# lastmod: '2022/3/10'
tags: [React] # 标签
draft: false
summary: 'React封装dots组件于轮播图一起使用'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/569ca2d5651844fb8001a3df9e71ee08~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b37fcfe99bd4c118fdbdfc7c149c2e7~tplv-k3u1fbpfcp-watermark.image?" alt="image.png" width="50%" />

封装 `dots` 组件，计算 `dots` 滚动的算法如下

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1dd417e515024ac59f9ee0f75dd58006~tplv-k3u1fbpfcp-watermark.image?)

```ts
import React, { useEffect, useRef } from 'react'
import { ReactNode } from 'react'
import type { FC } from 'react'

import style from './index.module.scss'
import classNames from 'classnames'

export interface IProps {
  children?: ReactNode
  selectIndex?: number
}

const Indicator: FC<IProps> = function (props) {
  const { children, selectIndex = 0 } = props

  // 1.定义获取dots的ref
  const dotsRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    // 2.获取selectIndex对应的子元素
    const selectEL = dotsRef.current?.children[selectIndex!]
    // 3.获取子元素左距离
    const itemLeft = (selectEL as HTMLElement)?.offsetLeft
    // 4.获取子元素本身的宽度
    const itemWidth = (selectEL as HTMLElement)?.clientWidth

    // 5.获取dots本身的宽度
    const dotsWidth = dotsRef.current?.clientWidth!

    // 6.计算滚动的的距离
    // 如果使用这个计算距离 长度大时会居中高亮
    let distance = itemLeft + itemWidth * 0.5 - dotsWidth * 0.5
    // 如果使用这个计算距离 长度大时不会居中高亮
    // let distance = itemLeft + itemWidth - dotsWidth

    // 7.计算可滚动的总距离
    const dotsScroll = dotsRef.current?.scrollWidth!
    const totalDistance = dotsScroll - dotsWidth

    // 8.边界判断
    // 左边边界判断
    if (distance < 0) distance = 0
    // 右边边界判断
    if (distance > totalDistance) distance = totalDistance

    // console.log(distance)
    dotsRef.current!.style.transform = `translate(${-distance}px)`
  }, [selectIndex])

  return (
    <div className="overflow-hidden">
      <div
        className={classNames('flex transition duration-200 ease-in-out ', style.scroll)}
        ref={dotsRef}
      >
        {children}
      </div>
    </div>
  )
}

export default Indicator

// 设置一个方便调试的name 可以不写 默认为组件名称
Indicator.displayName = 'Indicator'
```

`Indicator` 的样式

```css
.scroll {
  > * {
    flex-shrink: 0;
  }
}
```

在 `room-item` 组件中结合 `antd` 的轮播图组件时使用

```ts
...
import { Rating } from '@mui/material'
import { Carousel } from 'antd'
import { CarouselRef } from 'antd/es/carousel'
import Indicator from '../indicator'
import style from './index.module.scss'

export interface IProps {
  itemData: EntireHomeItem | HomeItem
  width?: '20%' | '25%' | '33.33%' // 展示的宽度
  itemClick?: (item: EntireHomeItem) => void
}

const RoomItem: FC<IProps> = function (props) {
  const { itemData, width = '25%', itemClick } = props

  // 定义dots的索引
  const [selectIndex, setSelectIndex] = useState(0)

  // 点击箭头切换图片
  // 1.获取轮播图的Ref
  const carouselRef = useRef<CarouselRef>(null)
  function controlClickHandle(isNext: boolean) {
    isNext ? carouselRef.current?.next() : carouselRef.current?.prev()

    // 1.如果点击下一个 index+1 上一个-1
    let newIndex = isNext ? selectIndex + 1 : selectIndex - 1
    // 2.如果小于0 index设为最后一个
    const len = itemData.picture_urls.length
    if (newIndex < 0) newIndex = len - 1
    // 3.如果大于最后一个 index设为第一个
    if (newIndex > len - 1) newIndex = 0
    setSelectIndex(newIndex)
  }

  function itemClickHandle() {
    itemClick && itemClick(itemData as EntireHomeItem)
  }

  return (
     <div className={style.swiper}>
      {/* 滚动箭头 */}
      <div className={style.control}>
        <div className={style.left} onClick={(e) => controlClickHandle(false)}>
          <IconArrowLeft width={20} height={20} />
        </div>
        <div className={style.right} onClick={(e) => controlClickHandle(true)}>
          <IconArrowRight width={20} height={20} />
        </div>
      </div>

      {/* 轮播图 */}
      <Carousel dots={false} ref={carouselRef}>
        {itemData.picture_urls?.map((item) => {
          return (
            <div key={item}>
              <img
                className=" bottom-0 h-[200px] w-[100%] object-cover rounded-md "
                src={item}
              />
            </div>
          )
        })}
      </Carousel>

      {/* 使用dots组件 */}
      <div className=" absolute z-[999] bottom-2 left-0 right-0 mx-auto w-[40%] overflow-hidden ">
        <Indicator selectIndex={selectIndex}>
          {itemData.picture_urls?.map((item, index) => {
            return (
              // w为14.28展示7个点 20为5个 看你怎么用了
              <div
                className="flex flex-shrink-0  justify-center items-center w-[14.28%]"
                key={item}
              >
                <span
                  className="w-[6px] h-[6px] bg-white rounded-full"
                  style={
                    selectIndex == index
                      ? {
                          width: '8px',
                          height: '8px',
                          background: '#00848A'
                        }
                      : {}
                  }
                ></span>
              </div>
            )
          })}
        </Indicator>
      </div>
    </div>
      ...
  )
}

export default RoomItem

// 设置一个方便调试的name 可以不写 默认为组件名称
RoomItem.displayName = 'RoomItem'

```

`room-item` 箭头的样式

```scss
// 轮播图样式
.swiper {
  @apply cursor-pointer;

  &:hover {
    .control {
      display: flex;
    }
  }

  .control {
    @apply absolute left-0 right-0 top-0 bottom-0 z-10 hidden justify-between  text-white;

    .left {
      @apply flex w-[50px] items-center justify-center bg-gradient-to-l from-transparent to-[rgba(0,0,0,0.25)];
    }

    .right {
      @apply flex w-[50px] items-center justify-center bg-gradient-to-r from-transparent to-[rgba(0,0,0,0.25)];
    }
  }
}
```
