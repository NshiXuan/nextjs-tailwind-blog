---
title: 'React+TS实现分片上传' # 标题
date: '2023/4/8' # 发布时间
# lastmod: '2022/3/10'
tags: [React、TypeScript] # 标签
draft: false
summary: 'React+TS实现分片上传'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5cdcaf5b1b54b20aecbd8738470e13e~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

前端上传大文件时，需要进行分片上传，这里展示 `React` + `TS` 实现分片上传

后端实现分片上传看一看：[Java 构建 minio 文件系统 - 掘金 (juejin.cn)](https://juejin.cn/post/7216681666106409016)

## 安装依赖

安装 `md5` 加密文件

```
pnpm add ts-md5
```

如果 `TS` 报模块不存在可以声明模块

```ts
declare module 'ts-md5/dist/md5'
```

## 分片上传

```ts
import { useState } from 'react'
import { Md5 } from 'ts-md5/dist/md5'

export default function Home() {
  const [video, setVideo] = useState<any>()

  // 定义每个分块为5M
  const size = 5 * 1024 * 1024

  let uploadUrl = 'http://localhost:9001/upload/uploadchunk'
  let checkUrl = 'http://localhost:9001/upload/checkchunk'
  let mergeUrl = 'http://localhost:9001/upload/mergechunks'

  const handleFileChange = async (event: React.ChangeEvent<HTMLInputElement>) => {
    const selectedFile = event.target.files?.[0]
    if (selectedFile) {
      // 1.计算分块总数
      const total = Math.ceil(selectedFile.size / size)

      // 2.定义记录第几个分块
      let chunkIndex = 0

      // 3.定义文件的fileMd5 用来给后端校验是否已上传分块
      let fileMd5 = ''
      // 获取第一个分片进而获取第一个分片的md5作为文件存储目录 如果加密整个文件 如果文件太大会崩溃
      const firstChunk = selectedFile.slice(0, size)

      // 4.通过md5加密获取fileMd5
      const reader = new FileReader()
      // reader.onload为异步
      reader.onload = (event: any) => {
        const arrayBuffer = event.target.result as ArrayBuffer
        // 将 ArrayBuffer 对象转换为字符串进行哈希
        fileMd5 = Md5.hashStr(new TextDecoder().decode(arrayBuffer))
      }

      // 5.确定获取到fileMd5后再上传 最后合并
      reader.onloadend = async function (event) {
        for (let i = 0; i < selectedFile.size; i += size) {
          // 5.1分块
          const chunk = selectedFile.slice(i, i + size)

          // 5.2检查分块是否已上传
          const checkForm = new FormData()
          checkForm.append('fileMd5', fileMd5)
          checkForm.append('chunkIndex', chunkIndex + '')
          await fetch(checkUrl, { method: 'post', body: checkForm }).then(async (res) => {
            const data = await res.json()

            // 5.3如果没上传再上传
            if (data.code == 0 && data.data != true) {
              // 上传分块
              const uploadForm = new FormData()
              uploadForm.append('fileMd5', fileMd5)
              uploadForm.append('chunk', chunk)
              uploadForm.append('chunkIndex', chunkIndex + '')
              await fetch(uploadUrl, { method: 'post', body: uploadForm })
            }
          })

          // 5.4获取下个分块序号
          chunkIndex++
        }

        // 5.5合并分块
        const mergeForm = new FormData()
        mergeForm.append('fileMd5', fileMd5)
        mergeForm.append('fileName', selectedFile.name)
        mergeForm.append('chunkTotal', total + '')
        await fetch(mergeUrl, { method: 'post', body: mergeForm })
      }
      reader.readAsArrayBuffer(firstChunk)
    }
  }

  // 获取视频
  function handleButtonClick() {
    fetch('http://localhost:9001/files').then(async (res) => {
      const data = await res.json()

      // 获取最新的视频
      setVideo(data.data[data.data.length - 1])
    })
  }

  return (
    <div>
      <div>hello</div>
      <input type="file" onChange={handleFileChange} />

      <button onClick={handleButtonClick}>获取视频</button>

      {video?.url && <video className="w-[80%]" src={video.url} controls></video>}
    </div>
  )
}
```
