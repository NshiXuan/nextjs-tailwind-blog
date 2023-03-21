---
title: 'React封装标签栏组件' # 标题
date: '2023/3/21' # 发布时间
# lastmod: '2022/3/10'
tags: [React] # 标签
draft: false
summary: 'React封装标签栏组件'
images: # 文章封面 必须要
  [
    'https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/569ca2d5651844fb8001a3df9e71ee08~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

在前端开发中经常需要使用 `Tabs` 栏，这里外面使用 `React` 封装一个
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce532b9717a44b50bca9be8f78077fba~tplv-k3u1fbpfcp-watermark.image?)

```ts
import { FC, useState } from 'react'

export interface IProps {
  tabList?: string[]
}

const EntireFilter: FC<IProps> = function (props) {
  const { tabList } = props

  // 1.记录选中item的数组
  const [selectItems, setSelectItems] = useState<any>([])

  // 2.点击高亮 再点击取消高亮
  function itemClickHandle(item: any) {
    const newItems = [...selectItems]
    // 2.1如果高亮数组中已经有了 移除
    if (newItems.includes(item)) {
      const itemIndex = newItems.findIndex((name) => name == item)
      newItems.splice(itemIndex, 1)
    } else {
      // 2.2 没有加入数组
      newItems.push(item)
    }
    setSelectItems(newItems)
  }

  return (
    <div className="flex h-12 items-center border-b bg-white pl-4">
      {tabList?.map((item, index) => {
        return (
          <div
            className=" text-font-primary-color mx-2 cursor-pointer rounded border py-[6px] px-3 "
            onClick={(e) => itemClickHandle(item)}
            style={selectItems.includes(item) ? { background: '#008489', color: '#fff' } : {}}
            key={item}
          >
            {item}
          </div>
        )
      })}
    </div>
  )
}

export default EntireFilter

// 设置一个方便调试的name 可以不写 默认为组件名称
EntireFilter.displayName = 'EntireFilter'
```

- 使用
  定义 `tabs`

```json
[
  "人数",
  "可免费取消",
  "房源类型",
  "价格",
  "位置区域",
  "闪定",
  "卧室/床数",
  "促销/优惠",
  "更多筛选条件"
]
```

在组件中使用

```ts
import tabList from '@/assets/data/filter_data.json'

const Entire: FC<IProps> = function (props) {
  ...
      <EntireFilter tabList={tabList} />
  ...
  )
}
```
