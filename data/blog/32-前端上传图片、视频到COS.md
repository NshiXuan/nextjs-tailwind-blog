---
title: '前端直接上传图片、视频到COS' # 标题
date: '2023/4/16' # 发布时间
# lastmod: '2022/3/10'
tags: [React] # 标签
draft: false
summary: '前端直接上传图片、视频到COS'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/569ca2d5651844fb8001a3df9e71ee08~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

为什么要在前端上传视频到 `COS` 呢？

- 因为考虑到服务器的带宽，大文件对服务器的带宽有较大的压力，所以选择直接在前端上传到 `COS` ，而不是走自己的后端接口

## 安装依赖

    pnpm add cos-js-sdk-v5 --save

## 上传测试

`COS` 提供了一个 `uploadFile` 的方法上传大文件，并实现了断点续传

- SliceSize 定义文件多大时分片上传（默认为 1MB），如下如果上传的文件大于等于 `5MB` ，则会自动分片上传
- 可以通过在 options 参数中设置 partSize 属性来自定义分片大小

这里测试直接把密钥写死了，这里如果在开发时需要后端返回临时的密钥

```ts
import React, { useState } from 'react'
import type { FC } from 'react'

import COS from 'cos-js-sdk-v5'
import { Progress, Space } from 'antd'

export interface IProps {}

const Test: FC<IProps> = function (props) {
  const [progress, setProgress] = useState(0)
  const [videoUrl, setVideoUrl] = useState<any>()

  // 上传文件、视频
  async function uploadFileOrPic(file: any) {
    var cos = new COS({
      SecretId: '密钥id',
      SecretKey: '密钥key',
    })

    // uploadFile方法上传文件时，如果上传的文件大小大于等于5MB，则会自动分片上传 默认的分片大小为8MB。但是可以通过在options参数中设置partSize属性来自定义分片大小
    cos.uploadFile(
      {
        Bucket: 'video-1309614912',
        Region: 'ap-guangzhou' /* 存储桶所在地域，必须字段 */,
        Key: 'test1.mp4', // 文件名
        Body: file, // 上传文件对象
        SliceSize: 1024 * 1024 * 5, // 大于5M分块上传
        onProgress: function (progressData) {
          console.log(JSON.stringify(progressData))
          setProgress(Math.ceil(progressData.percent * 100))
        },
        onFileFinish: function (err, data, options) {
          console.log(options.Key + '上传' + (err ? '失败' : '完成'))
        },
      },
      (err, data) => {}
    )
  }

  // 选择文件
  const handleFileChange = async (event: React.ChangeEvent<HTMLInputElement>) => {
    const selectedFile = event.target.files?.[0]
    if (selectedFile) {
      uploadFileOrPic(selectedFile)
    }
  }

  // 获取视频地址
  function getVideoUrl() {
    var cos = new COS({
      SecretId: '密钥id',
      SecretKey: '密钥key',
    })

    cos.getObjectUrl(
      {
        Bucket: 'video-1309614912',
        Region: 'ap-guangzhou' /* 存储桶所在地域，必须字段 */,
        Key: 'test1.mp4', // 文件名
        Sign: true /* 获取带签名的对象 URL */,
      },
      function (err, data) {
        if (err) return console.log(err)
        /* url为对象访问 url */
        var url = data.Url
        /* 复制 downloadUrl 的值到浏览器打开会自动触发下载 */
        var downloadUrl =
          url + (url.indexOf('?') > -1 ? '&' : '?') + 'response-content-disposition=attachment' // 补充强制下载的参数
        setVideoUrl(downloadUrl)
      }
    )
  }

  return (
    <div>
      <input type="file" onChange={handleFileChange} />
      <Space wrap>
        <Progress type="circle" percent={progress} />
      </Space>
      <div>
        <button onClick={getVideoUrl}>查询视频</button>
      </div>

      {videoUrl && <video className="w-[80%]" src={videoUrl} controls></video>}
    </div>
  )
}

export default Test
```
