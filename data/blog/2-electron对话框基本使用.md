---
title: 'electron对话框基本使用' # 标题
date: '2023/3/12' # 发布时间
lastmod: '2022/3/10'
tags: [electron] # 标签
draft: false
summary: 'electron对话框基本使用'
images: # 文章封面 必须要
  [
    'https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c3bfa87a-7673-48ef-9b64-155c9a0d0e98/Untitled.png',
  ]
layout: PostLayout
---

## 对话框基本使用

1. 在菜单中定义对话框 `menu.js`

```jsx
const { Menu, shell, app, BrowserWindow, dialog } = require('electron')

const isMac = process.platform == 'darwin'

const createMenu = (win) => {
  const config = [
    // window、苹果用户都有的菜单
    {
      label: '操作', // 菜单名称
      submenu: [
        {
          label: '退出',
          click: async () => {
            const res = await dialog.showMessageBox({
              title: 'codersx',
              detail: '你确定要退出吗',
              buttons: ['取消', '确定'],
              // cancelId: 1 // 定义按esc的时候默认触发1按钮 也就是确定 如果没定义 默认esc为0 也就是取消
            })

            // response=0 为取消 1为确定 按顺序来
            // res: { response: 0, checkboxChecked: false }

            // 如果为确定就退出
            if (res.response === 1) app.quit()
          },
        },
      ],
    },
  ]

  // Menu.buildFromTemplate构建菜单对象
  Menu.setApplicationMenu(Menu.buildFromTemplate(config))
}

module.exports = {
  createMenu,
}
```

- 没配置 `button` 的默认对话框

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ebf65b13-56c4-4c8d-952c-db3b889eb0ad/Untitled.png)

- 配置了 `button` 的对话框

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/46336138-b710-40ef-8e6e-ff456354385e/Untitled.png)

## 警告框与确认框结合使用 - checkbox

```
const { Menu, shell, app, BrowserWindow, dialog } = require('electron')

const isMac = process.platform == 'darwin'

const createMenu = (win) => {
  const config = [
    {
      label: '操作', // 菜单名称
      submenu: [
        // 菜单项
        {
          label: '访问网站',
          click: async () => {
            const res = await dialog.showMessageBox({
              title: 'codersx',
              detail: '你确定要访问吗',
              buttons: ['取消', '确定'], // 如果
              cancelId: 0, // 定义按esc的时候默认触发1按钮 也就是 确定
              checkboxLabel: '确定访问吗',
              checkboxChecked: true // 默认选中
            })
            // console.log(res)
            if (!res.checkboxChecked) {
              return dialog.showErrorBox('温馨提示', '你没有确认复选框')
            }
            if (res.response == 1) {
              shell.openExternal('https://www.houdunren.com')
            }
          }
        }
      ]
    }
  ]

  // Menu.buildFromTemplate构建菜单对象
  Menu.setApplicationMenu(Menu.buildFromTemplate(config))
}

module.exports = {
  createMenu
}
```

```
const { Menu, shell, app, BrowserWindow, dialog } = require('electron')

const isMac = process.platform == 'darwin'

const createMenu = (win) => {
  const config = [
    {
      label: '操作', // 菜单名称
      submenu: [
        // 菜单项
        {
          label: '访问网站',
          click: async () => {
            const res = await dialog.showMessageBox({
              title: 'codersx',
              detail: '你确定要访问吗',
              buttons: ['取消', '确定'], // 如果
              cancelId: 0, // 定义按esc的时候默认触发1按钮 也就是 确定
              checkboxLabel: '确定访问吗',
              checkboxChecked: true // 默认选中
            })
            // console.log(res)
            if (!res.checkboxChecked) {
              return dialog.showErrorBox('温馨提示', '你没有确认复选框')
            }
            if (res.response == 1) {
              shell.openExternal('https://www.houdunren.com')
            }
          }
        }
      ]
    }
  ]

  // Menu.buildFromTemplate构建菜单对象
  Menu.setApplicationMenu(Menu.buildFromTemplate(config))
}

module.exports = {
  createMenu
}
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/09683139-0222-4ac3-88b1-c1df38b19d01/Untitled.png)

- 如果没有勾选复选框，提示

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/769221aa-a702-44cb-8675-2581d2149f95/Untitled.png)

## 选择文件框

1. `html`

```html
<body>
  <button id="btn">上传文件</button>
  <div id="container"></div>
  <script src="renderer.js"></script>
</body>
```

1. 渲染进程

```jsx
window.addEventListener('DOMContentLoaded', () => {
  const btn = document.querySelector('#btn')
  btn.addEventListener('click', async () => {
    const res = await window.api.selectFilePreload()
    console.log(res)
  })
})
```

1. 预加载脚本

```jsx
// const fs = require('fs')
const { ipcRenderer, contextBridge } = require('electron')

contextBridge.exposeInMainWorld('api', {
  selectFilePreload: async () => {
    return ipcRenderer.invoke('selectFileMain')
  },
})
```

1. 主进程

`iocMain.js` 监听预加载脚本事件

```jsx
const { ipcMain, dialog } = require('electron')

ipcMain.handle('selectFileMain', async (event) => {
  const { filePaths } = await dialog.showOpenDialog({
    title: '请选择图片',
    properties: ['openFile', 'multiSelections'], // 打开文件 并且可以多选
    filters: [
      // 对选择的文件进行限制
      {
        name: 'image',
        extensions: ['jpg', 'png'],
      },
    ],
  })
  // res: {
  //   canceled: false,
  //   filePaths: [
  //     'C:\\Users\\NongSX\\Pictures\\Camera Roll\\Rio2.png.625x385_q100.png',
  //     'C:\\Users\\NongSX\\Pictures\\Camera Roll\\sunset_wide.png',
  //     'C:\\Users\\NongSX\\Pictures\\Camera Roll\\Yosemite-Color-Block.png'
  //   ]
  // }

  return filePaths
})
```

- 在 `main.js` 导入

```jsx
require('./ipcMain')
```

## 保存文件

1. `html`

```html
<body>
  <textarea name="content" id="" cols="30" rows="10"></textarea>
  <br />

  <button id="saveBtn">保存文件</button>
  <button id="btn">上传文件</button>
  <div id="container"></div>
  <script src="renderer.js"></script>
</body>
```

1. 渲染进程

```
window.addEventListener('DOMContentLoaded', () => {
  const btn = document.querySelector('#saveBtn')
  btn.addEventListener('click', async () => {
    // 1.通过属性选择器获取文本域
    const textarea = document.querySelector('[name="content"]')

    // 2.保存文件
    window.api.saveToFile(textarea.value)
  })
})
```

1. 预加载文件

```jsx
const { ipcRenderer, contextBridge } = require('electron')

contextBridge.exposeInMainWorld('api', {
  selectFilePreload: async () => {
    return ipcRenderer.invoke('selectFileMain')
  },
  // 保存文件
  saveToFile: (value) => {
    // 在主进程中保存文件
    ipcRenderer.send('saveFileMain', value)
  },
})
```

1. 主进程

`iocMain.js` 监听预加载脚本事件

```jsx
const { ipcMain, dialog } = require('electron')
const fs = require('fs')

...

// 保存文件
ipcMain.on('saveFileMain', async (event, value) => {
  // 1.获取上传的文件路径
  const { filePath } = await dialog.showSaveDialog({
    title: '保存文件'
  })
  // res: { canceled: false, filePath: 'C:\\Users\\NongSX\\Desktop\\333.html' }

  // 2.通过node存储文件
  fs.writeFileSync(filePath, value)
})
```

- 在 `main.js` 导入

```jsx
require('./ipcMain')
```

1. 使用

- 输入内容后点击保存文件

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fb9b70c9-020f-4c90-b58b-059dc5029dd0/Untitled.png)

- 输入文件名称保存

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/95ab89af-c9d2-4e51-a9fb-ca9649300ca5/Untitled.png)

- 保存后打开目录可以看到文件

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/282a5dea-d015-411c-bc86-c3ec8cc17260/Untitled.png)
