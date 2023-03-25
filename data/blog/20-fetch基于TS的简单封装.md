---
title: 'fetch基于TS的简单封装' # 标题
date: '2023/3/25' # 发布时间
# lastmod: '2022/3/10'
tags: [React] # 标签
draft: false
summary: 'fetch基于TS的简单封装'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4cb854cb50ce4a57bc45d14e23a6c4f0~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

`JS` 提供了 `fetch` 方法来请求数据，我们在这里对它进行简单的 `TS` 封装

## 封装 fetch

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f6ee4936f8f4d9987ed117e7d0c5837~tplv-k3u1fbpfcp-watermark.image?)

- `fetch`

```ts
const BASE_URL = '<http://codercba.com:1888/airbnb/api>'

type configType = { url: string; method: 'GET' | 'POST' | 'PUT' | 'DELETE'; data: any }

class SxRequest {
  baseURL: string

  constructor(config: { baseURL: string }) {
    this.baseURL = config.baseURL
  }

  async request<T = any>(config: configType) {
    // fetch中GET方法没有请求体 需要拼接到url中 参数直接拼接
    const fetchURL = this.baseURL + config.url

    // 如果是POST 把data传递给body
    const fetchConfig: RequestInit =
      config.method === 'POST'
        ? {
            method: config.method,
            body: config.data,
          }
        : {
            method: config.method,
          }

    // 获取fetch的数据并返回
    const res = await fetch(fetchURL, fetchConfig)
    const data: T = await res.json()
    return data
  }

  get<T = any>(url: string, data?: any) {
    return this.request<T>({ url, method: 'GET', data })
  }

  posst<T>(url: string, data?: any) {
    return this.request<T>({ url: url, method: 'POST', data })
  }
}

export default new SxRequest({
  baseURL: BASE_URL,
})
```

`home`

```ts
import sxRequest from '../fetch'
import { HomeGoodPriceData } from './home.type'

export const getHomeGoodPriceData = () => {
  return sxRequest.get<HomeGoodPriceData>('/home/goodprice')
}
```

- 使用

```ts
// 网络请求
const res = await getHomeGoodPriceData()
```
