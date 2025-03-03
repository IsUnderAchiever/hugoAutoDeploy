---
title: 2. 登录模块
description: 2. 登录模块
date: 2025-03-03
slug: 2. 登录模块
image: 202412212156937.png
categories:
    - React
---

> 本文转载自[B站 柴柴前端教书匠](https://www.bilibili.com/video/BV1Z44y1K7Fj/?spm_id_from=333.1387.0.0)

# 2. 登录模块

## 1. 基本结构搭建
`本节目标:`  能够使用antd搭建基础布局



**实现步骤**

1. 在 Login/index.js 中创建登录页面基本结构
2. 在 Login 目录中创建 index.scss 文件，指定组件样式
3. 将 logo.png 和 login.png 拷贝到 assets 目录中



**代码实现**

`pages/Login/index.js`

```jsx
import { Card } from 'antd'
import logo from '@/assets/logo.png'
import './index.scss'

const Login = () => {
  return (
    <div className="login">
      <Card className="login-container">
        <img className="login-logo" src={logo} alt="" />
        {/* 登录表单 */}
      </Card>
    </div>
  )
}

export default Login
```



`pages/Login/index.scss`

```css
.login {
  width: 100%;
  height: 100%;
  position: absolute;
  left: 0;
  top: 0;
  background: center/cover url('~@/assets/login.png');
  
  .login-logo {
    width: 200px;
    height: 60px;
    display: block;
    margin: 0 auto 20px;
  }
  
  .login-container {
    width: 440px;
    height: 360px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    box-shadow: 0 0 50px rgb(0 0 0 / 10%);
  }
  
  .login-checkbox-label {
    color: #1890ff;
  }
}
```



## 2. 创建表单结构
`本节目标:` 能够使用antd的Form组件创建登录表单



**实现步骤**

1. 打开 antd [Form 组件文档](https://ant.design/components/form-cn/)
2. 找到代码演示的第一个示例（基本使用），点击`<>`（显示代码），并拷贝代码到组件中
3. 分析 Form 组件基本结构
4. 调整 Form 组件结构和样式



**代码实现**

`pages/Login/index.js`

```jsx
import { Form, Input, Button, Checkbox } from 'antd'
const Login = () => {
  return (
    <Form>
      <Form.Item>
        <Input size="large" placeholder="请输入手机号" />
      </Form.Item>
      <Form.Item>
        <Input size="large" placeholder="请输入验证码" />
      </Form.Item>
      <Form.Item>
        <Checkbox className="login-checkbox-label">
          我已阅读并同意「用户协议」和「隐私条款」
        </Checkbox>
      </Form.Item>
      <Form.Item>
        <Button type="primary" htmlType="submit" size="large" block>
          登录
        </Button>
      </Form.Item>
    </Form>
  )
}
```



## 3. 表单校验实现
`本节目标:` 能够为手机号和密码添加表单校验



**实现步骤**

1. 为 Form 组件添加 `validateTrigger` 属性，指定校验触发时机的集合
2. **为 Form.Item 组件添加 name 属性，这样表单校验才会生效 [容易忽略]**
3. 为 Form.Item 组件添加 `rules` 属性，用来添加表单校验



**代码实现**

`page/Login/index.js`

```jsx
const Login = () => {
  return (
    <Form validateTrigger={['onBlur', 'onChange']}>
      <Form.Item
        name="mobile"
        rules={[
          {
            pattern: /^1[3-9]\d{9}$/,
            message: '手机号码格式不对',
            validateTrigger: 'onBlur'
          },
          { required: true, message: '请输入手机号' }
        ]}
      >
        <Input size="large" placeholder="请输入手机号" />
      </Form.Item>
      <Form.Item
        name="code"
        rules={[
          { len: 6, message: '验证码6个字符', validateTrigger: 'onBlur' },
          { required: true, message: '请输入验证码' }
        ]}
      >
        <Input size="large" placeholder="请输入验证码" maxLength={6} />
      </Form.Item>
      <Form.Item name="remember" valuePropName="checked">
        <Checkbox className="login-checkbox-label">
          我已阅读并同意「用户协议」和「隐私条款」
        </Checkbox>
      </Form.Item>

      <Form.Item>
        <Button type="primary" htmlType="submit" size="large" block>
          登录
        </Button>
      </Form.Item>
    </Form>
  )
}
```



## 4. 获取登录表单数据
`本节目标:` 能够拿到登录表单中用户的手机号码和验证码



**实现步骤**

1. 为 Form 组件添加 `onFinish` 属性，该事件会在点击登录按钮时触发
2. 创建 onFinish 函数，通过函数参数 values 拿到表单值
3. Form 组件添加 `initialValues` 属性，来初始化表单值



**代码实现**

`pages/Login/index.js`

```jsx
// 点击登录按钮时触发 参数values即是表单输入数据
const onFinish = values => {
  console.log(values)
}

<Form
  onFinish={ onFinish }
  initialValues={{
    mobile: '13911111111',
    code: '246810',
    remember: true
  }}
>...</Form>
```



## 5. 封装http工具模块
`本节目标:` 封装axios，简化操作

**实现步骤**

1. 创建 utils/http.js 文件
2. 创建 axios 实例，配置 baseURL，请求拦截器，响应拦截器
3. 在 utils/index.js 中，统一导出 http



**代码实现**

`utils/http.js`

```javascript
import axios from 'axios'

const http = axios.create({
  baseURL: 'http://geek.itheima.net/v1_0',
  timeout: 5000
})
// 添加请求拦截器
http.interceptors.request.use((config)=> {
    return config
  }, (error)=> {
    return Promise.reject(error)
})

// 添加响应拦截器
http.interceptors.response.use((response)=> {
    // 2xx 范围内的状态码都会触发该函数。
    // 对响应数据做点什么
    return response
  }, (error)=> {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么
    return Promise.reject(error)
})

export { http }
```



`utils/index.js`

```javascript
import { http } from './http'
export {  http }
```



## 6. 配置登录Mobx
`本节目标:`  基于mobx封装管理用户登录的store



`store/login.Store.js`

```javascript
// 登录模块
import { makeAutoObservable } from "mobx"
import { http } from '@/utils'

class LoginStore {
  token = ''
  constructor() {
    makeAutoObservable(this)
  }
  // 登录
  login = async ({ mobile, code }) => {
    const res = await http.post('http://geek.itheima.net/v1_0/authorizations', {
      mobile,
      code
    })
    this.token = res.data.token
  }
}
export default LoginStore
```



`store/index.js`

```javascript
import React from "react"
import LoginStore from './login.Store'

class RootStore {
  // 组合模块
  constructor() {
    this.loginStore = new LoginStore()
  }
}
// 导入useStore方法供组件使用数据
const StoresContext = React.createContext(new RootStore())
export const useStore = () => React.useContext(StoresContext)
```



## 7. 实现登录逻辑
`本节目标:`  在表单校验通过之后通过封装好的store调用登录接口



**实现步骤**

1. 使用useStore方法得到loginStore实例对象
2. 在校验通过之后，调用loginStore中的login函数
3. 登录成功之后跳转到首页



**代码实现**

```jsx
import { useStore } from '@/store'
const Login = () => {
  // 获取跳转实例对象
  const navigate = useNavigate()
  const { loginStore } = useStore()
  const onFinish = async values => {
    const { mobile, code } = values
    try {
      await loginStore.login({ mobile, code })
      navigate('/')
    } catch (e) {
      message.error(e.response?.data?.message || '登录失败')
    }
  }
  return (...)
}
```



## 8. token持久化
### 封装工具函数
`本节目标:`  能够统一处理 token 的持久化相关操作



**实现步骤**

1. 创建 utils/token.js 文件
2. 分别提供 getToken/setToken/clearToken/isAuth 四个工具函数并导出
3. 创建 utils/index.js 文件，统一导出 token.js 中的所有内容，来简化工具函数的导入
4. 将登录操作中用到 token 的地方，替换为该工具函数



**代码实现**



`utils/token.js`

```javascript
const TOKEN_KEY = 'geek_pc'

const getToken = () => localStorage.getItem(TOKEN_KEY)
const setToken = token => localStorage.setItem(TOKEN_KEY, token)
const clearToken = () => localStorage.removeItem(TOKEN_KEY)

export { getToken, setToken, clearToken }
```



### 持久化设置
`本节目标:`  使用token函数持久化配置



**实现步骤**

1. 拿到token的时候一式两份，存本地一份
2. 初始化的时候优先从本地取，取不到再初始化为控制



**代码实现**

`store/login.Store.js`

```javascript
// 登录模块
import { makeAutoObservable } from "mobx"
import { setToken, getToken, clearToken, http } from '@/utils'

class LoginStore {
  // 这里哦！！
  token = getToken() || ''
  constructor() {
    makeAutoObservable(this)
  }
  // 登录
  login = async ({ mobile, code }) => {
    const res = await http.post('http://geek.itheima.net/v1_0/authorizations', {
      mobile,
      code
    })
    this.token = res.data.token
    // 还有这里哦！！
    setToken(res.data.token)
  }
 
}
export default LoginStore
```



## 9. 请求拦截器注入token
`本节目标:` 把token通过请求拦截器注入到请求头中



![202503032212598.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202503032212598.png)



> 拼接方式：config.headers.Authorization = `Bearer ${token}}`
>



`utils/http.js`

```javascript
http.interceptors.request.use(config => {
  // if not login add token
  const token = getToken()
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})
```



## 10. 路由鉴权实现
`本节目标:` 能够实现未登录时访问拦截并跳转到登录页面



**实现思路**

> 自己封装 `AuthRoute` 路由鉴权高阶组件，实现未登录拦截，并跳转到登录页面
>
> 思路为：判断本地是否有token，如果有，就返回子组件，否则就重定向到登录Login
>



**实现步骤**

1. 在 components 目录中，创建 AuthRoute/index.js 文件
2. 判断是否登录
3. 登录时，直接渲染相应页面组件
4. 未登录时，重定向到登录页面
5. 将需要鉴权的页面路由配置，替换为 AuthRoute 组件渲染



**代码实现**

`components/AuthRoute/index.js`

```jsx
// 1. 判断token是否存在
// 2. 如果存在 直接正常渲染
// 3. 如果不存在 重定向到登录路由

// 高阶组件:把一个组件当成另外一个组件的参数传入 然后通过一定的判断 返回新的组件
import { getToken } from '@/utils'
import { Navigate } from 'react-router-dom'

function AuthRoute ({ children }) {
  const isToken = getToken()
  if (isToken) {
    return <>{children}</>
  } else {
    return <Navigate to="/login" replace />
  }
}

// <AuthComponent> <Layout/> </AuthComponent>
// 登录：<><Layout/></>
// 非登录：<Navigate to="/login" replace />

export { AuthRoute }
```



`src/app.js`

```jsx
import { Router, Route } from 'react-router-dom'
import { AuthRoute } from '@/components/AuthRoute'
import Layout from '@/pages/Layout'
import Login from '@/pages/Login'

function App() {
  return (
    <Router>
      <Routes>
          {/* 需要鉴权的路由 */}
          <Route path="/*" element={
            <AuthRoute>
              <Layout />
            </AuthRoute>
          } />
          {/* 不需要鉴权的路由 */}
          <Route path='/login' element={<Login />} />
       </Routes>
    </Router>
  )
}
export default App
```



> 更新: 2022-09-17 14:53:47  
> 原文: <https://www.yuque.com/fechaichai/tzzlh1/bivd9h>