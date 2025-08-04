---
title: 手把手创建React19项目
description: 手把手创建React19项目
date: 2025-05-25
slug: 手把手创建React19项目
image: 202506081450325.png
categories:
    - Vite
---
# 手把手创建React19项目

## 初始化

```shell
npm create vite@latest
```
![Pasted image 20250525121822.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081448712.png)
```shell
cd react-app
pnpm install
pnpm run dev
```
## 路径别名
```shell
pnpm install --save-dev @types/node
```
> 配置`vite.config.ts`

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react-swc'
import * as path from 'path'

// https://vite.dev/config/
export default defineConfig({
  plugins: [
    react()
  ],
  // 配置路径别名
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'), // 建议指向 src 根目录
      '@assets': path.resolve(__dirname, './src/common/assets'),
      '@components': path.resolve(__dirname, './src/components')
    }
  }
})
```
> 配置`tsconfig.app.json`
> 配置`baseUrl`和`paths`即可

```json
{
  "compilerOptions": {
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.app.tsbuildinfo",
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": [
      "ES2020",
      "DOM",
      "DOM.Iterable"
    ],
    "module": "ESNext",
    "skipLibCheck": true,
    // 设置 baseUrl 为项目根目录
    "baseUrl": ".",
    "paths": {
      // 定义 @/ 别名指向 src 目录下的所有文件/文件夹
      "@/*": [
        "src/*"
      ]
    },
    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "moduleDetection": "force",
    "noEmit": true,
    "jsx": "react-jsx",
    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "erasableSyntaxOnly": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedSideEffectImports": true
  },
  "include": [
    "src"
  ]
}
```
## Mock
```shell
pnpm i mockjs vite-plugin-mock -D
```
> 配置`vite.config.ts`

```typescript
import react from '@vitejs/plugin-react-swc'
import * as path from 'path'
import { viteMockServe } from 'vite-plugin-mock'
import type { UserConfigExport, ConfigEnv } from 'vite'

// https://vite.dev/config/
export default ({ command }: ConfigEnv): UserConfigExport => {
  return {
    plugins: [
      react(),
      viteMockServe({
        // default
        mockPath: 'mock',
        enable: command === 'serve',
      }),
    ],
    resolve: {
      alias: {
        '@': path.resolve(__dirname, './src'), // 建议指向 src 根目录
        '@assets': path.resolve(__dirname, './src/common/assets'),
        '@components': path.resolve(__dirname, './src/components')
      }
    }
  }
}
```
> 在根目录新建`mock/user.ts`

user.ts代码如下
```typescript
const createUserList = () => {
  return [
    {
      id: 1,
      name: '张三',
      age: 21
    },
    {
      id: 2,
      name: '李四',
      age: 22,
    }
  ]
}
export default [
  // 获取用户信息接口
  {
    url: '/api/user',
    method: 'get',
    response: (request: any) => {
      return {
        code: 200,
        data: createUserList
      }
    }
  }
]
```
![Pasted image 20250525124545.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081448713.png)
## Tailwindcss4
```shell
pnpm install tailwindcss @tailwindcss/vite
```
> 配置`vite.config.ts`

```typescript
import react from '@vitejs/plugin-react-swc'
import * as path from 'path'
import { viteMockServe } from 'vite-plugin-mock'
import type { UserConfigExport, ConfigEnv, PluginOption } from 'vite'
import tailwindcss from '@tailwindcss/vite'

// https://vite.dev/config/
export default ({ command }: ConfigEnv): UserConfigExport => {
  return {
    plugins: [
      react(),
      tailwindcss() as PluginOption,
      viteMockServe({
        // default
        mockPath: 'mock',
        enable: command === 'serve',
      }),
    ],
    resolve: {
      alias: {
        '@': path.resolve(__dirname, './src'), // 建议指向 src 根目录
        '@assets': path.resolve(__dirname, './src/common/assets'),
        '@components': path.resolve(__dirname, './src/components')
      }
    }
  }
}
```
> 在全局css(比如`index.css`)里引入如下代码

```css
@import "tailwindcss";
```

> 测试效果

```shell
import { useState } from 'react'

function App() {
  const [count, setCount] = useState(0)

  return (
    <>
      <button
        className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
        onClick={() => setCount(count + 1)}>
        count+1
      </button>
      <span>{count}</span>
    </>
  )
}

export default App
```

## Antd5
```shell
pnpm install antd --save

# 下载react19兼容包
pnpm add @ant-design/v5-patch-for-react-19 --save
```

> 在入口处引入兼容包，比如main.tsx

```tsx
import tailwindcss from '@tailwindcss/vite'
```

## Router
```shell
pnpm i react-router-dom
```

>在src下新建`router.ts`

```ts
import { createBrowserRouter } from "react-router-dom";
import App from '@/App'
import Home from '@/Home'

const router = createBrowserRouter([
  {
    path: "/",
    Component: App,
  }, {
    path: "/home",
    Component: Home,
  }
]);

export default router;
```
配置`main.tsx`
```tsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import '@ant-design/v5-patch-for-react-19'
import { RouterProvider } from 'react-router-dom'
import router from '@/router'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>
)
```

可参考[文档](https://juejin.cn/post/7241863128962482234)
## 约定式路由
> 实现约定式路由可以使用`vite-plugin-pages`插件
> 参考[Vite 中文文档](https://www.viterc.cn/en/vite-plugin-pages.html)、[约定式路由生成神器：vite-plugin-pages](https://juejin.cn/post/7347251816617705522)

```shell
pnpm install -D vite-plugin-pages
```
目录结构如下
![Pasted image 20250525191237.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081448714.png)
vite.config.ts
```ts
import react from '@vitejs/plugin-react-swc'
import * as path from 'path'
import { viteMockServe } from 'vite-plugin-mock'
import type { UserConfigExport, ConfigEnv, PluginOption } from 'vite'
import tailwindcss from '@tailwindcss/vite'
import Pages from 'vite-plugin-pages'

// https://vite.dev/config/
export default ({ command }: ConfigEnv): UserConfigExport => {
  return {
    plugins: [
      react(),
      tailwindcss() as PluginOption,
      Pages({
        dirs: 'src/pages',
        extensions: ['tsx', 'jsx'],
        // 排除组件目录
        exclude: ['**/components/**'],
        onRoutesGenerated(routes) {
          // 将 404 路由移至最后
          const notFoundRoute = routes.find(r => r.path === '/:404*')
          if (notFoundRoute) {
            return [
              ...routes.filter(r => r !== notFoundRoute),
              { ...notFoundRoute, path: '*' }
            ]
          }
          return routes
        }
      }) as PluginOption,
      viteMockServe({
        // default
        mockPath: 'mock',
        enable: command === 'serve',
      }),
    ],
    resolve: {
      alias: {
        // 建议指向 src 根目录
        '@': path.resolve(__dirname, './src'),
        '@assets': path.resolve(__dirname, './src/common/assets'),
        '@components': path.resolve(__dirname, './src/components')
      }
    }
  }
}
```
main.tsx
```tsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import '@ant-design/v5-patch-for-react-19'
import App from './App'
import { BrowserRouter } from 'react-router-dom'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>
)
```
App.tsx
```tsx
import './App.css'
import { useRoutes } from 'react-router-dom'
// @ts-ignore
import routes from '~react-pages'
import { Suspense } from 'react'

function App() {
  return <Suspense fallback={<p>Loading...</p>}>{useRoutes(routes)}</Suspense>
}

export default App
```
## 防抖实现

> 这里使用`lodash`实现防抖

```shell
pnpm add lodash @types/lodash
```
```tsx
import { Input } from 'antd'
import { debounce } from 'lodash'
import { useCallback, useState } from 'react'

function Home() {
  const [inputValue, setInputValue] = useState('')

  // 使用 useCallback 缓存防抖函数，避免重复创建
  const handleDebouncedSearch = useCallback(
    // 创建 500ms 防抖函数
    debounce((value: string) => {
      // 实际业务逻辑
      console.log('防抖值:', value)
    }, 500),
    // 空依赖数组确保防抖函数只创建一次
    []
  )

  return (
    <div>
      <Input
        value={inputValue}
        onChange={(e) => {
          // 同步更新状态
          setInputValue(e.target.value)
          // 异步防抖处理
          handleDebouncedSearch(e.target.value)
        }}
        placeholder="请输入内容"
      />
    </div>
  )
}

export default Home
```
