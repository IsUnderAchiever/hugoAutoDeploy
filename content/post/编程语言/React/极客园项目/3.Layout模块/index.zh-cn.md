---
title: 3. Layout模块
description: 3. Layout模块
date: 2025-03-03
slug: 3. Layout模块
image: 202412212156937.png
categories:
    - React
---

> 本文转载自[B站 柴柴前端教书匠](https://www.bilibili.com/video/BV1Z44y1K7Fj/?spm_id_from=333.1387.0.0)

# 3. Layout模块

## 1. 基本结构搭建
`本节目标:`  能够使用antd搭建基础布局



**实现步骤**

1. 打开 antd/Layout 布局组件文档，找到示例：顶部-侧边布局-通栏
2. 拷贝示例代码到我们的 Layout 页面中
3. 分析并调整页面布局



**代码实现**

`pages/Layout/index.js`

```jsx
import { Layout, Menu, Popconfirm } from 'antd'
import {
  HomeOutlined,
  DiffOutlined,
  EditOutlined,
  LogoutOutlined
} from '@ant-design/icons'
import './index.scss'

const { Header, Sider } = Layout

const GeekLayout = () => {
  return (
    <Layout>
      <Header className="header">
        <div className="logo" />
        <div className="user-info">
          <span className="user-name">user.name</span>
          <span className="user-logout">
            <Popconfirm title="是否确认退出？" okText="退出" cancelText="取消">
              <LogoutOutlined /> 退出
            </Popconfirm>
          </span>
        </div>
      </Header>
      <Layout>
        <Sider width={200} className="site-layout-background">
          <Menu
            mode="inline"
            theme="dark"
            defaultSelectedKeys={['1']}
            style={{ height: '100%', borderRight: 0 }}
          >
            <Menu.Item icon={<HomeOutlined />} key="1">
              数据概览
            </Menu.Item>
            <Menu.Item icon={<DiffOutlined />} key="2">
              内容管理
            </Menu.Item>
            <Menu.Item icon={<EditOutlined />} key="3">
              发布文章
            </Menu.Item>
          </Menu>
        </Sider>
        <Layout className="layout-content" style={{ padding: 20 }}>内容</Layout>
      </Layout>
    </Layout>
  )
}

export default GeekLayout
```



`pages/Layout/index.scss`

```css
.ant-layout {
  height: 100%;
}

.header {
  padding: 0;
}

.logo {
  width: 200px;
  height: 60px;
  background: url('~@/assets/logo.png') no-repeat center / 160px auto;
}

.layout-content {
  overflow-y: auto;
}

.user-info {
  position: absolute;
  right: 0;
  top: 0;
  padding-right: 20px;
  color: #fff;
  
  .user-name {
    margin-right: 20px;
  }
  
  .user-logout {
    display: inline-block;
    cursor: pointer;
  }
}
.ant-layout-header {
  padding: 0 !important;
}
```



## 2. 二级路由配置
`本节目标:`  能够在右侧内容区域展示左侧菜单对应的页面内容



**使用步骤**

1. 在 pages 目录中，分别创建：Home（数据概览）/Article（内容管理）/Publish（发布文章）页面文件夹
2. 分别在三个文件夹中创建 index.js 并创建基础组件后导出
3. 在app.js中配置嵌套子路由，在layout.js中配置二级路由出口
4. 使用 Link 修改左侧菜单内容，与子路由规则匹配实现路由切换



**代码实现**

`pages/Home/index.js`

```jsx
const Home = () => {
  return <div>Home</div>
}
export default Home
```



`pages/Article/index.js`

```jsx
const Article = () => {
  return <div>Article</div>
}
export default Article
```



`pages/Publish/index.js`

```jsx
const Publish = () => {
  return <div>Publish</div>
}
export default Publish
```



`app.js`

```jsx
<Route path="/" element={
    <AuthRoute>
      <Layout />
    </AuthRoute>
  }>
    {/* 二级路由默认页面 */}
    <Route index element={<Home />} />
    <Route path="article" element={<Article />} />
    <Route path="publish" element={<Publish />} />
</Route>
<Route path="/login" element={<Login/>}></Route>
```



`pages/Layout/index.js`

```jsx
// 配置Link组件
<Menu
    mode="inline"
    theme="dark"
    style={{ height: '100%', borderRight: 0 }}
    selectedKeys={[selectedKey]}
    >
    <Menu.Item icon={<HomeOutlined />} key="/">
      <Link to="/">数据概览</Link>
    </Menu.Item>
    <Menu.Item icon={<DiffOutlined />} key="/article">
      <Link to="/article">内容管理</Link>
    </Menu.Item>
    <Menu.Item icon={<EditOutlined />} key="/publish">
      <Link to="/publish">发布文章</Link>
    </Menu.Item>
</Menu>

// 二级路由对应显示
<Layout className="layout-content" style={{ padding: 20 }}>
  {/* 二级路由默认页面 */}
  <Outlet />
</Layout>
```



## 3. 菜单高亮显示
`本节目标:`  能够在页面刷新的时候保持对应菜单高亮



> 思路
>
> 1. Menu组件的selectedKeys属性与Menu.Item组件的key属性发生匹配的时候，Item组件即可高亮
> 2. 页面刷新时，将`当前访问页面的路由地址`作为 Menu 选中项的值（selectedKeys）即可
>



**实现步骤**

1. 将 Menu 的`key` 属性修改为与其对应的路由地址
2. 获取到当前正在访问页面的路由地址
3. 将当前路由地址设置为 `selectedKeys` 属性的值



**代码实现**

`pages/Layout/index.js`

```jsx
import { useLocation } from 'react-router-dom'

const GeekLayout = () => {
  const location = useLocation()
  // 这里是当前浏览器上的路径地址
  const selectedKey = location.pathname

  return (
    // ...
    <Menu
      mode="inline"
      theme="dark"
      selectedKeys={[selectedKey]}
      style={{ height: '100%', borderRight: 0 }}
    >
      <Menu.Item icon={<HomeOutlined />} key="/">
        <Link to="/">数据概览</Link>
      </Menu.Item>
      <Menu.Item icon={<DiffOutlined />} key="/article">
        <Link to="/article">内容管理</Link>
      </Menu.Item>
      <Menu.Item icon={<EditOutlined />} key="/publish">
        <Link to="/publish">发布文章</Link>
      </Menu.Item>
    </Menu>
  )
}
```



## 4. 展示个人信息
`本节目标:`  能够在页面右上角展示登录用户名



**实现步骤**

1. 在store中新增userStore.js模块，在其中定义获取用户信息的mobx代码
2. 在store的入口文件中组合新增的userStore模块
3. 在Layout组件中调用action函数获取用户数据
4. 在Layout组件中获取个人信息并展示



**代码实现**

`store/user.Store.js`

```javascript
// 用户模块
import { makeAutoObservable } from "mobx"
import { http } from '@/utils'

class UserStore {
  userInfo = {}
  constructor() {
    makeAutoObservable(this)
  }
  async getUserInfo() {
    const res = await http.get('/user/profile')
    this.userInfo = res.data.data
  }
}

export default UserStore
```



`store/index.js`

```javascript
import React from "react"
import LoginStore from './login.Store'
import UserStore from './user.Store'

class RootStore {
  // 组合模块
  constructor() {
    this.loginStore = new LoginStore()
    this.userStore = new UserStore()
  }
}

const StoresContext = React.createContext(new RootStore())
export const useStore = () => React.useContext(StoresContext)
```



`pages/Layout/index.js`

```jsx
import { useEffect } from 'react'
import { observer } from 'mobx-react-lite'

const GeekLayout = () => {
  const { userStore } = useStore()
  // 获取用户数据
  useEffect(() => {
    try {
      userStore.getUserInfo()
    } catch { }
  }, [userStore])
    
  return (
    <Layout>
      <Header className="header">
        <div className="logo" />
        <div className="user-info">
          <span className="user-name">{userStore.userInfo.name}</span>
        </div>
      </Header>
      {/* 省略无关代码 */}
    </Layout>
  )
}

export default observer(GeekLayout)
```



## 5. 退出登录实现
`本节目标:`  能够实现退出登录功能



**实现步骤**

1. 为气泡确认框添加确认回调事件
2. 在`store/login.Store.js` 中新增退出登录的action函数，在其中删除token
3. 在回调事件中，调用loginStore中的退出action
4. 退出后，返回登录页面



**代码实现**

`store/login.Store.js`

```javascript
class LoginStore {
  // 退出登录
  loginOut = () => {
    this.token = ''
    clearToken()
  }
}

export default LoginStore
```



`pages/Layout/index.js`

```jsx
// login out
const navigate = useNavigate()
const onLogout = () => {
    loginStore.loginOut()
    navigate('/login')
}

<span className="user-logout">
    <Popconfirm title="是否确认退出？" okText="退出" cancelText="取消" onConfirm={onLogout}>
      <LogoutOutlined /> 退出
    </Popconfirm>
</span>
```



## 6. 处理Token失效
`本节目标:`  能够在响应拦截器中处理token失效



> 说明：为了能够在非组件环境下拿到路由信息，需要我们安装一个history包
>



![202503032211919.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202503032211919.png)



**实现步骤**

1. 安装history包：`yarn add history`
2. 创建 `utils/history.js`文件
3. 在app.js中使用我们新建的路由并配置history参数
4. 通过响应拦截器处理 token 失效，如果发现是401调回到登录页



**代码实现**

`utils/history.js`

```javascript
// https://github.com/remix-run/react-router/issues/8264

import { createBrowserHistory } from 'history'
import { unstable_HistoryRouter as HistoryRouter } from 'react-router-dom'
const history = createBrowserHistory()

export {
  HistoryRouter,
  history
}
```



`app.js`

```jsx
import { HistoryRouter, history } from './utils/history'

function App() {
  return (
    <HistoryRouter history={history}>
       ...省略无关代码
    </HistoryRouter>
  )
}

export default App
```



`utils/http.js`

```javascript
import { history } from './history'

http.interceptors.response.use(
  response => {
    return response.data
  },
  error => {
    if (error.response.status === 401) {
      // 删除token
      clearToken()
      // 跳转到登录页
      history.push('/login')
    }
    return Promise.reject(error)
  }
)
```



## 7. 首页Home图表展示


`本节目标:`  实现首页echart图表封装展示



![202503032211924.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202503032211924.png)



**需求描述：**

1. 使用eharts配合react封装柱状图组件Bar
2. 要求组件的标题title，横向数据xData，纵向数据yData，样式style可定制



**代码实现**

`components/Bar/index.js`

```jsx
import * as echarts from 'echarts'
import { useEffect, useRef } from 'react'

function echartInit (node, xData, sData, title) {
  const myChart = echarts.init(node)
  // 绘制图表
  myChart.setOption({
    title: {
      text: title
    },
    tooltip: {},
    xAxis: {
      data: xData
    },
    yAxis: {},
    series: [
      {
        name: '销量',
        type: 'bar',
        data: sData
      }
    ]
  })
}

function Bar ({ style, xData, sData, title }) {
  // 1. 先不考虑传参问题  静态数据渲染到页面中
  // 2. 把那些用户可能定制的参数 抽象props (1.定制大小 2.data 以及说明文字)
  const nodeRef = useRef(null)
  useEffect(() => {
    echartInit(nodeRef.current, xData, sData, title)
  }, [xData, sData])

  return (
    <div ref={nodeRef} style={style}></div>
  )
}

export default Bar
```



`pages/Home/index.js`

```jsx
import Bar from "@/components/Bar"
import './index.scss'
const Home = () => {
  return (
    <div className="home">
      <Bar
        style={{ width: '500px', height: '400px' }}
        xData={['vue', 'angular', 'react']}
        sData={[50, 60, 70]}
        title='三大框架满意度' />

      <Bar
        style={{ width: '500px', height: '400px' }}
        xData={['vue', 'angular', 'react']}
        sData={[50, 60, 70]}
        title='三大框架使用度' />
    </div>
  )
}

export default Home
```



`pages/Home/index.scss`



```css
.home {
  width: 100%;
  height: 100%;
  align-items: center;
}
```



# 


> 更新: 2022-09-18 17:07:51  
> 原文: <https://www.yuque.com/fechaichai/tzzlh1/sa1mxw>