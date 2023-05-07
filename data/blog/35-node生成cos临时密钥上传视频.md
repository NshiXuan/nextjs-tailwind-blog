---
title: 'node生成cos临时密钥上传视频' # 标题
date: '2023/5/07' # 发布时间
# lastmod: '2022/3/10'
tags: [Nestjs] # 标签
draft: false
summary: 'node生成cos临时密钥上传视频'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ecdbb7610e3648d0885d86143fbab531~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

## nestjs 后端生成 cos 临时密钥

- 简单示例

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c99b7f1ea8b47beac539414e948bb35~tplv-k3u1fbpfcp-watermark.image?)

```ts
import { Controller, Get } from '@nestjs/common'
import { getCredential } from 'qcloud-cos-sts'
import { AuthService } from './auth.service'

/**
 * 授权策略
 * @param {String} LongBucketName 桶名
 * @param {String} Region 桶区域
 */
const policy = function () {
  const LongBucketName = 'video-1309614912'
  const Region = 'ap-guangzhou'
  const ShortBucketName = LongBucketName.substr(0, LongBucketName.indexOf('-'))
  const AppId = LongBucketName.substr(LongBucketName.indexOf('-') + 1)
  return {
    version: '2.0' /* 策略语法版本，默认为2.0 */,
    statement: [
      {
        action: [
          // '*'
          // 所有 action 请看文档 https://cloud.tencent.com/document/product/436/31923
          // 简单上传
          'name/cos:PutObject',
          'name/cos:PostObject',
          // 分片上传
          'name/cos:InitiateMultipartUpload',
          'name/cos:ListMultipartUploads',
          'name/cos:ListParts',
          'name/cos:UploadPart',
          'name/cos:CompleteMultipartUpload',
          'name/cos:uploadFiles',
        ] /* 此处是指 COS API，根据需求指定一个或者一序列操作的组合或所有操作(*) */,
        effect: 'allow' /* 有 allow （允许）和 deny （显式拒绝）两种情况 */,
        principal: { qcs: ['*'] } /* 委托人 授权子账户权限 */,
        resource: [
          'qcs::cos:' +
            Region +
            ':uid/' +
            AppId +
            ':prefix//' +
            AppId +
            '/' +
            ShortBucketName +
            '/*',
        ] /* 授权操作的具体数据，可以是任意资源、指定路径前缀的资源、指定绝对路径的资源或它们的组合 */,
      },
    ],
  }
}

@Controller('auth')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Get('key')
  async getCosKey() {
    const res = await this.getKey()

    console.log('🚀 ~ file: auth.controller.ts:74 ~ AuthController ~ getCosKey ~ res:', res)

    return res
  }

  getKey() {
    return new Promise((resolve, reject) => {
      getCredential(
        {
          secretId: 'cos的secretId',
          secretKey: 'cos的secretKey',
          policy: policy(),
          durationSeconds: 7200, // 7200 秒
        },
        (err, credential) => {
          // console.log(err || credential); // 这里拿到错误或者临时签名信息。
          /*  R.send()为封装的统一返回方法，请自行删减。 */

          return resolve(credential)
        }
      )
    })
  }
}
```

## 前端获取临时 cos 上传视频

1.  封装上传工具方法

```js
import COS from 'cos-js-sdk-v5'
import { uploadImageApi } from '@/services/modules/common/upload'
import { getCosKey } from '@/services'

/*                               将base64转成file文件方法                                     */
function dataURLtoFile(dataurl, filename, type) {
  // 获取到base64编码
  const arr = dataurl.split(',')
  // 将base64编码转为字符串
  const bstr = window.atob(arr[1])
  let n = bstr.length
  const u8arr = new Uint8Array(n) // 创建初始化为0的，包含length个元素的无符号整型数组
  while (n--) {
    u8arr[n] = bstr.charCodeAt(n)
  }
  return new File([u8arr], filename, {
    type: type,
  })
}

/*                                   上传图片                                    */
export function uploadImg(file, insertFn) {
  // 1.定义文件阅读器 获取图片文件原本数据
  const readObj = new FileReader()

  // 2.压缩图片
  readObj.onload = async () => {
    // 获取图片原始的base64数据
    // console.log(readObj.result, 'readObj.result')

    // 2.1 创建canvas于img标签 通过canvas压缩图片
    let canvas = document.createElement('canvas')
    let img = document.createElement('img')

    // 2.2 给img标签赋值原本的base64数据
    img.src = readObj.result

    // 2.3 开始压缩
    img.onload = async () => {
      // 压缩从如下宽高 宽度、高度看需求
      // 2.3.1 定义压缩起始位置 压缩多少宽高
      // 获取图片的宽高 通过宽高比例进行压缩 如果把300下调成100 压更模糊 更小 反之亦然
      // console.log(img.width, img.height)
      canvas.width = 300
      canvas.height = 300 * (img.height / img.width)
      let ctx = canvas.getContext('2d')
      // 0 0 代表左上角开始 300代表宽高
      ctx.drawImage(img, 0, 0, 300, 300 * (img.height / img.width))

      // 2.3.2 压缩语法 压缩后返回新的base64
      // canvas.toDataURL(type,encoderOptions)
      // toDataURL用于将canvas对象转为base64位编码
      // type表示图片格式 默认为image/png
      // encoderOptions 0到1之间取值，用于表示图片质量
      let newImgBase64 = canvas.toDataURL(file.type, 10 / 100)
      // console.log(newImgBase64)

      // 2.3.3 调用dataURLtoFile方法 把压缩后的base64转为file文件
      const newFile = dataURLtoFile(newImgBase64, file.name, file.type)
      console.log(newFile, 'newFile')

      // 2.3.4 创建form表单 把压缩后的文件放大form中发请求上传图片
      const form = new FormData()
      form.append('file', newFile, newFile.name)
      const res = await uploadImageApi(form)

      // 2.3.5 调用wangEditor的inserFn方法把路径插入到img标签
      insertFn(res.data.url, '', '')
    }
  }

  // 3.这个必须要 不然readObj.onload不起作用
  readObj.readAsDataURL(file)
}

/*                             上传视频                                   */
export function uploadVideo(editor, file, insertFn) {
  // 1.获取cos临时密钥
  // getCosKey().then(res => {
  //   console.log(res.data.credentials)
  // })

  var cos = new COS({
    getAuthorization: function (options, callback) {
      var url = 'http://localhost:3000/auth/key'
      var xhr = new XMLHttpRequest()
      xhr.open('GET', url, true)
      xhr.onload = function (e) {
        try {
          var data = JSON.parse(e.target.responseText)
          var credentials = data.credentials
        } catch (e) {}
        if (!data || !credentials) {
          return console.error('credentials invalid:\n' + JSON.stringify(data, null, 2))
        }
        callback({
          TmpSecretId: credentials.tmpSecretId,
          TmpSecretKey: credentials.tmpSecretKey,
          SecurityToken: credentials.sessionToken,
          // 建议返回服务器时间作为签名的开始时间，避免用户浏览器本地时间偏差过大导致签名错误
          StartTime: data.startTime, // 时间戳，单位秒，如：1580000000
          ExpiredTime: data.expiredTime, // 时间戳，单位秒，如：1580000000
        })
      }
      xhr.send()
    },
  })

  //  2.判断视频是否存在
  // const res = FileExist(cos, file)
  // console.log(res, 'FileExist')

  // 3.上传视频 uploadFile方法上传文件时，如果上传的文件大小大于等于5MB，则会自动分片上传 但是可以通过在options参数中设置partSize属性来自定义分片大小
  cos.uploadFile(
    {
      Bucket: 'video-1309614912',
      Region: 'ap-guangzhou' /* 存储桶所在地域，必须字段 */,
      Key: file.name, // 文件名
      Body: file, // 上传文件对象
      SliceSize: 1024 * 1024 * 5, // 大于5M分块上传
      onProgress: function (progressData) {
        console.log(JSON.stringify(progressData))
        editor.showProgressBar(progressData.percent * 100)
      },
      onFileFinish: function (err, data, options) {
        console.log(options.Key + '上传' + (err ? '失败' : '完成'))
        if (!err) {
        }
      },
    },
    (err, data) => {}
  )

  // 4.最后插入视频标签
  const videoURL = getFileURL(cos, file)
  insertFn(videoURL)
}

/*                          查询视频url                            */
function getFileURL(cos, file, Bucket = 'video-1309614912', Region = 'ap-guangzhou') {
  // 1.配置信息
  const url = cos.getObjectUrl(
    {
      Bucket: Bucket,
      Region: Region /* 存储桶所在地域，必须字段 */,
      Key: file.name, // 文件名
      Sign: true /* 获取带签名的对象 URL */,
    },
    function (err, data) {
      if (err) return console.log(err)
      /* url为对象访问 url */
      var url = data.Url
      /* 复制 downloadUrl 的值到浏览器打开会自动触发下载 */
      var downloadUrl =
        url + (url.indexOf('?') > -1 ? '&' : '?') + 'response-content-disposition=attachment' // 补充强制下载的参数
    }
  )

  return url
}

//
function FileExist(cos, file, Bucket = 'video-1309614912', Region = 'ap-guangzhou') {
  cos.headObject(
    {
      Bucket: Bucket /* 填入您自己的存储桶，必须字段 */,
      Region: Region /* 存储桶所在地域，例如ap-beijing，必须字段 */,
      Key: file.name /* 存储在桶里的对象键（例如1.jpg，a/b/test.txt），必须字段 */,
    },
    function (err, data) {
      if (data) {
        console.log('对象存在')
        return true
      } else if (err.statusCode == 404) {
        console.log('对象不存在')
        return false
      } else if (err.statusCode == 403) {
        console.log('没有该对象读权限')
      }
    }
  )
}
```

2.  使用 `wangEditor` 上传

```js
<template>
  <div
    class="pt-editor"
    style="border: 1px solid #ccc; border-radius: 5px; overflow: auto"
  >
    <Toolbar
      style="border-bottom: 1px solid #ccc"
      :editor="editor"
      :defaultConfig="toolbarConfig"
      :mode="mode"
    />

    <Editor
      class=""
      :style="`height: ${height}px; overflow-y: hidden`"
      v-bind:value="editorValue"
      :defaultConfig="editorConfig"
      :mode="mode"
      @onCreated="onCreated"
      @onChange="onChange"
    />
  </div>
</template>

<script>
import Vue from 'vue'
import { Editor, Toolbar } from '@wangeditor/editor-for-vue'

import { uploadImg, uploadVideo } from '@/utils/uploadTools'

// 可以直接v-model双向绑定使用
export default Vue.extend({
  components: { Editor, Toolbar },
  // 实现双向绑定
  model: {
    prop: 'editorValue',
    event: 'change'
  },
  props: {
    height: {
      // 编辑器高度
      type: Number,
      default: 500
    },
    editorValue: {
      // 编辑器的默认内容 这个不需要传 直接通过v-model绑定即可
      type: String,
      default: ''
    },
    mode: {
      // 显示工具类模式
      type: String,
      default: 'default' // 只有default与simple两个值
    }
  },
  data() {
    // self用来实现在data中调用method的方法
    let self = this
    return {
      editor: null,
      // html: '<p>hello</p>', html从props中获取默认内容展示在编辑器上
      // 如果内容没有用p标签包裹 需要用p标签包裹
      html: this.editorValue.startsWith('<p>')
        ? this.editorValue
        : `<p>${this.editorValue}</p>`,
      toolbarConfig: {
        excludeKeys: [
          'blockquote', // 取消引用
          'bgColor', // 取消背景色
          'todo', // 取消待办
          'emotion', // 取消表情
          // 'group-video', // 取消视频
          'fullScreen', // 取消全屏
          'insertTable' // 取消表格
        ]
      },
      editorConfig: { placeholder: '请输入内容...' },
      // mode: 'default', // or 'simple'
      showContent: '',

      // 编辑器配置上传图片接口
      editorConfig: {
        MENU_CONF: {
          // 图片上传
          uploadImage: {
            fieldName: 'file',
            // 自定义上传 并压缩
            async customUpload(file, insertFn) {
              uploadImg(file, insertFn)
            }
          },

          // 视频生成
          uploadVideo: {
            // 自定义上传
            async customUpload(file, insertFn) {
              uploadVideo(self.editor, file, insertFn)
            }
          }
        }
      }
    }
  },
  methods: {
    onCreated(editor) {
      this.editor = Object.seal(editor) // 一定要用 Object.seal() ，否则会报错
    },

    // 监听编辑器内容改变
    onChange(editor) {
      // console.log('onChange', editor.getHtml()) // onChange 时获取编辑器最新内容
      this.$emit('change', this.editor.getHtml())
    }
  },

  mounted() {},

  beforeDestroy() {
    const editor = this.editor
    if (editor == null) return
    editor.destroy() // 组件销毁时，及时销毁编辑器
  }
})
</script>

<style src="@wangeditor/editor/dist/css/style.css"></style>
<style>
/**
.w-e-progress-bar {
  position: relative;
  z-index: 99;
  height: 5px;
  background-color: rgb(10, 251, 10);
}
</style>
```
