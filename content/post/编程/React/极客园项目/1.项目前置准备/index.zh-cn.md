---
title: 1. 项目前置准备
description: 1. 项目前置准备
date: 2025-03-03
slug: 1. 项目前置准备
image: 202412212156937.png
categories:
    - React
---

> 本文转载自[B站 柴柴前端教书匠](https://www.bilibili.com/video/BV1Z44y1K7Fj/?spm_id_from=333.1387.0.0)

# 1. 项目前置准备

## 1. 项目介绍
`本节目标:`  了解项目的定位和功能



+ 项目功能演示 
    - 登录、退出
    - 首页
    - 内容（文章）管理：文章列表、发布文章、修改文章
+ 技术 
    - React 官方脚手架 `create-react-app`
    - react hooks
    - 状态管理：mobx
    - UI 组件库：`antd` v4
    - ajax请求库：`axios`
    - 路由：`react-router-dom` 以及 `history`
    - 富文本编辑器：`react-quill`
    - CSS 预编译器：`sass`



## 2. 项目搭建
`本节目标:`  能够基于脚手架搭建项目基本结构



**实现步骤**

1.  使用create-react-app生成项目   `npx create-react-app geek-pc` 
2.  进入根目录  `cd geek-pc` 
3.  启动项目   `yarn start` 
4.  调整项目目录结构 

```bash
/src
  /assets         项目资源文件，比如，图片 等
  /components     通用组件
  /pages          页面
  /store          mobx 状态仓库
  /utils          工具，比如，token、axios 的封装等
  App.js          根组件
  index.css       全局样式
  index.js        项目入口
```

 

**保留核心代码**

`src/index.js`

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
)
```



`src/App.js`

```jsx
export default function App() {
  return <div>根组件</div>
}
```



## 3. 使用gitee管理项目
`本节目标:`  能够将项目推送到gitee远程仓库



**实现步骤**

1. 在项目根目录打开终端，并初始化 git 仓库（如果已经有了 git 仓库，无需重复该步），命令：`git init`
2. 添加项目内容到暂存区：`git add .`
3. 提交项目内容到仓库区：`git commit -m '项目初始化'`
4. 添加 remote 仓库地址：`git remote add origin [gitee 仓库地址]`
5. 将项目内容推送到 gitee：`git push origin master -u`

## 4. 使用scss预处理器
`本节目标:`  能够在CRA中使用sass书写样式



> `SASS` 是一种预编译的 CSS，作用类似于 Less。由于 React 中内置了处理 SASS 的配置，所以，在 CRA 创建的项目中，可以直接使用 SASS 来写样式
>



**实现步骤**

1.  安装解析 sass 的包：`yarn add sass -D` 
2.  创建全局样式文件：`index.scss` 



```css
body {
  margin: 0;
}

#root {
  height: 100%;
}
```

 

## 5. 配置基础路由
`本节目标:` 能够配置登录页面的路由并显示到页面中



**实现步骤**



1. 安装路由：`yarn add react-router-dom`
2. 在 pages 目录中创建两个文件夹：Login、Layout
3. 分别在两个目录中创建 index.js 文件，并创建一个简单的组件后导出
4. 在 App 组件中，导入路由组件以及两个页面组件
5. 配置 Login 和 Layout 的路由规则



**代码实现**

`pages/Login/index.js`

```jsx
const Login = () => {
  return <div>login</div>
}
export default Login
```



`pages/Layout/index.js`

```jsx
const Layout = () => {
  return <div>layout</div>
}
export default Layout
```



`app.js`

```jsx
// 导入路由
import { BrowserRouter, Route, Routes } from 'react-router-dom'

// 导入页面组件
import Login from './pages/Login'
import Layout from './pages/Layout'

// 配置路由规则
function App() {
  return (
    <BrowserRouter>
      <div className="App">
       <Routes>
            <Route path="/" element={<Layout/>}/>
            <Route path="/login" element={<Login/>}/>
        </Routes>
      </div>
    </BrowserRouter>
  )
}

export default App
```



## 6. 组件库antd使用
`本节目标:`  能够使用antd的Button组件渲染按钮



**实现步骤**

1. 安装 antd 组件库：`yarn add antd`
2. 全局导入 antd 组件库的样式
3. 导入 Button 组件
4. 在 Login 页面渲染 Button 组件进行测试



**代码实现**

`src/index.js`

```javascript
// 先导入 antd 样式文件
// https://github.com/ant-design/ant-design/issues/33327
import 'antd/dist/antd.min.css'
// 再导入全局样式文件，防止样式覆盖！
import './index.css'
```



`pages/Login/index.js`

```jsx
import { Button } from 'antd'

const Login = () => (
  <div>
    <Button type="primary">Button</Button>
  </div>
)
```



## 7. 配置别名路径
`本节目标:`  能够配置@路径简化路径处理



> [自定义 CRA 的默认配置](https://ant.design/docs/react/use-with-create-react-app-cn#%E9%AB%98%E7%BA%A7%E9%85%8D%E7%BD%AE)  
[craco 配置文档](https://github.com/gsoft-inc/craco/blob/master/packages/craco/README.md#configuration)
>
> + CRA 将所有工程化配置，都隐藏在了 `react-scripts` 包中，所以项目中看不到任何配置信息
> + 如果要修改 CRA 的默认配置，有以下几种方案： 
>     1. 通过第三方库来修改，比如，`@craco/craco`  （推荐）
>     2. 通过执行 `yarn eject` 命令，释放 `react-scripts` 中的所有配置到项目中
>





**实现步骤**



1. 安装修改 CRA 配置的包：`yarn add -D @craco/craco`
2. 在项目根目录中创建 craco 的配置文件：`craco.config.js`，并在配置文件中配置路径别名
3. 修改 `package.json` 中的脚本命令
4. 在代码中，就可以通过 `@` 来表示 src 目录的绝对路径
5. 重启项目，让配置生效



**代码实现**

`craco.config.js`

```javascript
const path = require('path')

module.exports = {
  // webpack 配置
  webpack: {
    // 配置别名
    alias: {
      // 约定：使用 @ 表示 src 文件所在路径
      '@': path.resolve(__dirname, 'src')
    }
  }
}
```



package.json

```json
// 将 start/build/test 三个命令修改为 craco 方式
"scripts": {
  "start": "craco start",
  "build": "craco build",
  "test": "craco test",
  "eject": "react-scripts eject"
}
```



## 8. @别名路径提示
`本节目标:`  能够让vscode识别@路径并给出路径提示

**实现步骤**

1. 在项目根目录创建 `jsconfig.json` 配置文件
2. 在配置文件中添加以下配置



**代码实现**

```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```



vscode会自动读取`jsconfig.json` 中的配置，让vscode知道@就是src目录



## 9. 安装dev-tools调试工具
> [https://gitee.com/react-cp/react-pc-doc](https://gitee.com/react-cp/react-pc-doc)  这里找到dev-tools.crx文件
>



> 更新: 2022-07-07 21:38:08  
> 原文: <https://www.yuque.com/fechaichai/tzzlh1/obkdhx>