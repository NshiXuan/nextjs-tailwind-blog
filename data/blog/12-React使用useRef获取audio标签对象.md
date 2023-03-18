---
title: 'React使用useRef获取audio标签对象' # 标题
date: '2023/3/18' # 发布时间
# lastmod: '2022/3/10'
tags: [React] # 标签
draft: false
summary: 'React使用useRef获取audio标签对象'
images: # 文章封面 必须要
  [
    'https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/569ca2d5651844fb8001a3df9e71ee08~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

前端开发中，经常需要获取标签对象对其进行操作

比如下面的音频标签，需要通过 `xxxRef.current` 获取到标签对象在调用 `pause` 或 `play` 来控制播放或者暂停

```ts
...

const AppPlayerBar: FC<IProps> = function (props) {
  // 控制是否播放
  const [isPlaying, setIsPlaying] = useState(false)

  // 1.定义audioRef获取audio标签
  const audioRef = useRef<HTMLAudioElement>(null)

  // 进入页面播放音乐
  useEffect(() => {
    // DOM渲染完成后给audio赋值播放地址
    audioRef.current!.src =
      'http://baicizhan.qiniucdn.com/word_audios/favor.mp3'

    // 播放
    audioRef.current
      ?.play()
      .then(() => {
        console.log('播放成功')
      })
      .catch((err) => {
        setIsPlaying(false)
        console.log('播放失败', err)
      })
  }, [])

  // 3.点击播放、暂停按钮播放音乐
  function handlePlayBtnClick() {
    // 控制播放器的播放/暂停
    isPlaying
      ? audioRef.current?.pause()
      : audioRef.current?.play().catch(() => setIsPlaying(false))

    // 改变isPlaying的状态
    setIsPlaying(!isPlaying)
  }

  return (
    <PlayerBarWrapper className="sprite_playbar">
       ...
        {/* 播放控制器 */}
          <button
            className="btn sprite_playbar play"
            onClick={handlePlayBtnClick}
          ></button>
        ...

      {/* 2.给audio绑定ref 播放器 */}
      <audio ref={audioRef} />
    </PlayerBarWrapper>
  )
}

export default AppPlayerBar
```
