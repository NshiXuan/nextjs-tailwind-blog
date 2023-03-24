---
title: 'React封装PictrueBrowser预览图片组件' # 标题
date: '2023/3/24' # 发布时间
# lastmod: '2022/3/10'
tags: [React] # 标签
draft: false
summary: 'React封装PictrueBrowser预览图片组件'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/569ca2d5651844fb8001a3df9e71ee08~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8eb11ebbc7dc4633b7a2877de795dd1d~tplv-k3u1fbpfcp-watermark.image?)

`PictrueBrowser` 组件

- `Indicator` 看之前封装的 `dots` 组件

```ts
import { FC, useState } from 'react'
import { useEffect } from 'react'

import IconArrowLeft from '@/assets/svg/icon-arrow-left'
import IconArrowRight from '@/assets/svg/icon-arrow-right'
import IconClose from '@/assets/svg/icon-close'
import IconTriangleBottom from '@/assets/svg/icon-triangle-bottom'
import Indicator from '../indicator'

export interface IProps {
  pictureUrls?: string[] // 图片数组
  closeClick: (e: any) => void // 关闭图片
}

const PictrueBrowser: FC<IProps> = function (props) {
  const { pictureUrls, closeClick } = props

  // 记录当前浏览的图片
  const [currentIndex, setCurrentIndex] = useState(0)

  // 当图片展示出来 让滚动功能消失
  useEffect(() => {
    document.body.style.overflow = 'hidden'
    return () => {
      // 关闭图片浏览后 恢复默认
      document.body.style.overflow = 'auto'
    }
  }, [])

  // 点击控制器切换图片
  function controlClickHandle(isNext: boolean) {
    let newIndex = isNext ? currentIndex + 1 : currentIndex - 1
    if (newIndex < 0) newIndex = pictureUrls?.length! - 1
    if (newIndex > pictureUrls?.length! - 1) newIndex = 0
    setCurrentIndex(newIndex)
  }

  return (
    <div className="fixed left-0 right-0 bottom-0 top-0 z-[999] flex flex-col bg-[#333] ">
      {/* 关闭按钮 设为层级最高 */}
      <div className=" relative h-[86px] ">
        <div
          className=" absolute top-[15px] right-[25px] z-[9999] cursor-pointer"
          onClick={closeClick}
        >
          <IconClose />
        </div>
      </div>

      {/* 图片展示 */}
      <div className=" relative flex flex-1 items-center px-[10px]">
        {/* 控制器 设为层级第二高 */}
        <div className="absolute top-0 left-0 right-0 bottom-0 z-10 flex w-[100%] items-center justify-between text-white ">
          <div className=" cursor-pointer" onClick={(e) => controlClickHandle(false)}>
            <IconArrowLeft width={77} height={77} />
          </div>
          <div className=" cursor-pointer" onClick={(e) => controlClickHandle(true)}>
            <IconArrowRight width={77} height={77} />
          </div>
        </div>

        {/* 图片 */}
        <div className="relative flex h-[100%] w-[100%] overflow-hidden  ">
          {/* 可以用react-transition-group库来实现切换动画 */}
          <img
            className=" absolute top-0 left-0 right-0 mx-auto h-[100%] max-w-[105vh] select-none "
            src={pictureUrls?.[currentIndex]}
            alt=""
          />
        </div>
      </div>

      {/* 指示器 */}
      <div className="my-[20px] flex h-[100px] justify-center ">
        <div className=" b-[10px] max-w-[105vh] text-white ">
          {/* 描述 */}
          <div className="flex justify-between  ">
            <div>
              <span>
                {currentIndex + 1}/{pictureUrls?.length}
              </span>
              <span>room apartment图片{currentIndex + 1}</span>
            </div>
            <div className="flex cursor-pointer items-center ">
              <span>隐藏图片列表</span>
              <IconTriangleBottom />
            </div>
          </div>

          {/* 图片列表 */}
          <div className="mt-2">
            <Indicator selectIndex={currentIndex}>
              {pictureUrls?.map((item, index) => {
                return (
                  <div
                    className="mr-[15px] cursor-pointer"
                    key={item}
                    onClick={(e) => setCurrentIndex(index)}
                  >
                    <img
                      className="h-[67px] opacity-50"
                      src={item}
                      alt=""
                      style={currentIndex == index ? { opacity: '1' } : {}}
                    />
                  </div>
                )
              })}
            </Indicator>
          </div>
        </div>
      </div>
    </div>
  )
}

export default PictrueBrowser

// 设置一个方便调试的name 可以不写 默认为组件名称
PictrueBrowser.displayName = 'PictrueBrowser'
```

使用,点击图片即可预览

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/272ced8c107541e8b48d5591c5f6d969~tplv-k3u1fbpfcp-watermark.image?)

```ts
import PictrueBrowser from '@/components/picture-browser'
import type { FC } from 'react'
import { useState } from 'react'
import style from './index.module.scss'

export interface IProps {
  pictureUrls?: string[]
}

const DetailPicture: FC<IProps> = function (props) {
  const { pictureUrls } = props

  // 定义是否展示图片浏览 PictrueBrowser
  const [showBrowser, setShowBrowser] = useState(false)

  // 点击图片展示图片浏览
  function showBrowserHandle() {
    setShowBrowser(true)
  }

  return (
    <div className={style.wrapper}>
      <div className={style.top}>
        <div className={style.left}>
          <div className={style.item} onClick={showBrowserHandle}>
            <img src={pictureUrls?.[0]} alt="" />
            <div className={style.cover}></div>
          </div>
        </div>

        <div className={style.right}>
          {pictureUrls?.slice(1, 5).map((item, index) => {
            return (
              <div className={style.item} key={item} onClick={showBrowserHandle}>
                <img src={item} alt="" />
                <div className={style.cover}></div>
              </div>
            )
          })}
        </div>
      </div>

      <div className={style.showBtn} onClick={showBrowserHandle}>
        查看照片
      </div>

      {/* 预览图片 */}
      {showBrowser && (
        <PictrueBrowser pictureUrls={pictureUrls} closeClick={(e) => setShowBrowser(false)} />
      )}
    </div>
  )
}

export default DetailPicture

// 设置一个方便调试的name 可以不写 默认为组件名称
DetailPicture.displayName = 'DetailPicture'
```
