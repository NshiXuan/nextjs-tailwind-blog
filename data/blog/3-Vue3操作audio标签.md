---
title: 'Vue3获取audio标签播放音频' # 标题
date: '2023/3/15' # 发布时间
# lastmod: '2022/3/10'
tags: [Vue3] # 标签
draft: false
summary: 'Vue3获取audio标签播放音频'
images: # 文章封面 必须要
  [
    'https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/001450c70cc74e7f87a40412d20f06a9~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
theme: fancy
highlight: a11y-dark
---

## audio 常用的属性

- `controls` 是否显示控制器
- `autoplay: boolean` 是否自动播放
- `src` 播放链接

如果给 `audio` 添加 `controls` 就会显示在页面中，如果不加就不会显示

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b96dc832ef34cbe9c4fd93b185dcbeb~tplv-k3u1fbpfcp-watermark.image?" alt="image.png" width="50%" />

## vue3 获取 audio

```ts
;<audio ref="audioRef"></audio>

const audioRef = ref<HTMLAudioElement>()

// 播放
function handlePlay() {
  audioRef.value!.src = `http://baicizhan.qiniucdn.com/word_audios/${word?.value?.word}.mp3`
  // 播放 暂停也同理
  audioRef.value?.play()
}
```
