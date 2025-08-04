---
title: 约定式路由
description: 约定式路由
date: 2025-05-25
slug: 约定式路由
image: 202506081450325.png
categories:
    - Vite
---
# 约定式路由

## 基本实现

> 常见`route.ts`配置

```ts
import { createBrowserRouter } from "react-router-dom";
import App from '@/App'
import Home from '@/pages/Home'
import About from '@/pages/About'

const router = createBrowserRouter([
  {
    path: "/",
    Component: App,
  },
  {
    path: "/home",
    Component: Home,
  },
  {
    path: "/about",
    Component: About,
  }
]);

export default router;
```

> 也可以自己实现一些meta属性
> 比如，`title`用来动态设置浏览器标签页的标题
> `description`用于设置页面的 meta description 标签，对 SEO 有益
> `requiresAuth`用于设置页面级别的权限控制
> `layout`设置页面布局
> `transitionName`设置页面过渡/动画
> 类似vue里的`keep-alive`实现缓存控制


```ts
const router = createBrowserRouter([
  {
    path: "/",
    Component: App,
    handle: {
      meta: {
        title: '首页',
        description: '这是首页',
        requiresAuth: false,
      }
    }
  }
]);

export default router;
```

在入口文件中使用指定`route`配置
```tsx
import route from '@/route'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <RouterProvider router={route} />
  </StrictMode>
)
```
> 在一些常用的前端框架中其实都有`约定式路由`的功能，比如nuxtjs、nextjs、umi、icejs...
> 他们不必配置route文件，而是会根据当前项目目录结构自动生成路由配置

![Pasted image 20250525163729.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081448562.png)
> 接下来我们要在vite里实现类似的效果

我们需要保证生成的route.tsx配置 类似如效果
```tsx
import { createBrowserRouter } from 'react-router-dom'
import App from '@/App'
import Loading from '@/pages/Loading'
import NotFound from '@/pages/NotFound'

const router = createBrowserRouter([
  {
    path: '/',
    Component: App,
  },
  {
    path: '/home',
    hydrateFallbackElement: <Loading />,
    async lazy() {
      let { default: Home } = await import('@/pages/Home')
      return {
        Component: Home,
      }
    },
  },
  {
    path: '/about',
    hydrateFallbackElement: <Loading />,
    async lazy() {
      let { default: About } = await import('@/pages/About')
      return {
        Component: About,
      }
    },
  },
  {
    path: '*',
    Component: NotFound,
  },
])

export default router
```
新建如下目录结构
![Pasted image 20250525173850.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081448563.png)
`types/page.ts`
```ts
// 定义页面基础配置接口
export interface PageConfig {
  /** 页面标题 (必填) */
  title: string;
  /** 菜单排序 (选填，默认0) */
  menuOrder?: number;
  /** 其他扩展属性 */
  [key: string]: any;
}
```
`pages/xxx/config.ts`
```ts
import type { PageConfig } from '@/types/page';

const config: PageConfig = {
  title: 'NotFound',
  menuOrder: 99
};

export default config;
```
我们认为只要pages目录下存在config.ts的目录都需要生成对应的路由配置
这样在新建页面时，copy一下config.ts的内容就能实现route自动生成

> 在vite中，可以通过如下方式实现模块导入

```ts
const pageModules = import.meta.glob('./pages/**/config.ts')
console.log('pageModules:', pageModules)
```
打印结果如下
![Pasted image 20250525173015.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081448564.png)
这里打印的key就是导入模块的属性名，value就是动态导入的函数

如果希望使用全局导入的方法，需要配置`eager`
```ts
const pageModules = import.meta.glob('./pages/**/config.ts', {
  eager: true,
})
console.log('pageModules:', pageModules)
```
![Pasted image 20250525173243.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081448565.png)
这里其实有一个default属性，如下
![Pasted image 20250525173332.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081448566.png)
可以直接将default进行导入
```ts
const pageModules = import.meta.glob('./pages/**/config.ts', {
  eager: true,
  import: 'default',
})
console.log('pageModules:', pageModules)
```
![Pasted image 20250525173423.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081448567.png)
接下来需要将配置转成数组的格式
```ts
const pageModules = import.meta.glob('./pages/**/config.ts', {
  eager: true,
  import: 'default',
})
const routes = Object.entries(pageModules)
console.log('routes:', routes)
```
![Pasted image 20250525173605.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081448568.png)
我们希望能够配置一个类似如下结构的路由
```ts
const pageModules = import.meta.glob('./pages/**/config.ts', {
  eager: true,
  import: 'default',
})
const routes = Object.entries(pageModules).map(([pagePath, config]) => {
  return {
    path: '/home',
    hydrateFallbackElement: <Loading />,
    async lazy() {
      let { default: Home } = await import('@/pages/Home')
      return {
        Component: Home,
      }
    },
    handle: {
      meta: {
        title: '首页',
        description: '这是首页',
        requiresAuth: false,
      },
    },
  }
})
```
这里的meta其实就是`types/page.ts`的结构，只是我们现在定义的类型并不是title、description和requiresAuth

这里的结构其实能简化成如下形式
```json
{
  path: '/home',
  hydrateFallbackElement: <Loading />,
  lazy: {
    Component: async () => (await import('@/pages/Home')).default,
  },
  handle: {
    meta: {
      title: '首页',
      description: '这是首页',
      requiresAuth: false,
    },
  },
}
```
关于路径其实是根据打印的pagePath来实现的(pagePath: ./pages/About/config.ts)
去掉`./pages`和`/config.ts`即可
```ts
const pageModules = import.meta.glob('./pages/**/config.ts', {
  eager: true,
  import: 'default',
})
const routes = Object.entries(pageModules).map(([pagePath, config]) => {
  console.log('pagePath:', pagePath)
  console.log('config:', config)
  const path = pagePath.replace('./pages', '').replace('/config.ts', '')
  console.log('path:', path)

  return {
    path: '/home',
    hydrateFallbackElement: <Loading />,
    lazy: {
      Component: async () => (await import('./pages/Home')).default,
    },
    handle: {
      meta: config,
    },
  }
})
console.log('routes:', routes)
```
发现打印path是大写，因为目录是大写，如`path: /About`，修改目录结构，改成小写，如`pages/about/index.tsx`
我们这里的首页是`pages/home/index.tsx`，所以显示结果正常`/home`
但如果首页是`pages/index.tsx`，显示结果就会是空，所以还要加一层判断，如果为空，则替换为`/`
```ts
const pageModules = import.meta.glob('./pages/**/config.ts', {
  eager: true,
  import: 'default',
})
const routes = Object.entries(pageModules).map(([pagePath, config]) => {
  console.log('pagePath:', pagePath)
  console.log('config:', config)
  let path = pagePath.replace('./pages', '').replace('/config.ts', '')
  // 确保首页正常显示
  path = path || '/'
  return {
    path: path,
    hydrateFallbackElement: <Loading />,
    lazy: {
      Component: async () => (await import('./pages/home')).default,
    },
    handle: {
      meta: config,
    },
  }
})
console.log('routes:', routes)
```

接下来是import的部分，这里需要注意一个点，可能会写成如下代码
```ts
const pageModules = import.meta.glob('./pages/**/config.ts', {
  eager: true,
  import: 'default',
})
const routes = Object.entries(pageModules).map(([pagePath, config]) => {
  console.log('pagePath:', pagePath)
  console.log('config:', config)
  let path = pagePath.replace('./pages', '').replace('/config.ts', '')
  // 确保首页正常显示
  path = path || '/'
  const compPath = pagePath.replace('config.ts', 'index.tsx')
  // ./pages/loading/index.tsx
  console.log('compPath:', compPath)
  return {
    path: path,
    hydrateFallbackElement: <Loading />,
    lazy: {
      // 直接 import compPath
      Component: async () => (await import(compPath)).default,
    },
    handle: {
      meta: config,
    },
  }
})
console.log('routes:', routes)
```
这里在开发环境没有问题，但是在生产环境会出现问题
vite使用的是rollup进行打包，有一个要求，`import`内部不允许使用变量，必须要使用字面量的形式，不然会影响静态分析
警告如下
```
18:23:56 [vite] (client) warning:
C:/Users/Administrator/Desktop/vite/react-app/src/route.tsx
21 |          }, this),
22 |          lazy: {
23 |              Component: async ()=>(await import(compPath)).default
   |                                                 ^^^^^^^^
24 |          },
25 |          handle: {
The above dynamic import cannot be analyzed by Vite.
See https://github.com/rollup/plugins/tree/master/packages/dynamic-import-vars#limitations for supported dynamic import formats. If this is intended to be left as-is, you can use the /* @vite-ignore */ comment inside the import() call to suppress this warning.

  Plugin: vite:import-analysis
  File: C:/Users/Administrator/Desktop/vite/react-app/src/route.tsx
```
解决方法是在一开始导入组件模块即可
```tsx
import { createBrowserRouter, type RouteObject } from 'react-router-dom'
import App from '@/App'
import Loading from '@/pages/loading'
import NotFound from '@/pages/NotFound'

type ModuleWithDefault = {
  default: React.ComponentType<any>
}

const compModules = import.meta.glob('./pages/**/index.tsx')

const pageModules = import.meta.glob('./pages/**/config.ts', {
  eager: true,
  import: 'default',
})

const routes = Object.entries(pageModules).map(([pagePath, config]) => {
  let path = pagePath.replace('./pages', '').replace('/config.ts', '')
  path = path || '/'
  const compPath = pagePath.replace('config.ts', 'index.tsx')
  return {
    path: path,
    hydrateFallbackElement: <Loading />,
    lazy: () =>
      compModules[compPath]().then((module) => ({
        Component: (module as ModuleWithDefault).default,
      })),
    handle: {
      meta: config,
    },
  }
}) as RouteObject[]

const router = createBrowserRouter([
  {
    path: '/',
    Component: App,
  },
  ...routes,
  {
    path: '*',
    Component: NotFound,
  },
])

export default router
```
现在大致实现了类似的效果，但是如果需要实现完整的功能，还有一些需要注意的细节
1. 嵌套路由
2. 路由参数
3. 导航守卫
## vite插件
> 实现约定式路由可以使用`vite-plugin-pages`插件
> 参考[Vite 中文文档](https://www.viterc.cn/en/vite-plugin-pages.html)、[约定式路由生成神器：vite-plugin-pages](https://juejin.cn/post/7347251816617705522)

```shell
pnpm install -D vite-plugin-pages
```
目录结构如下
![Pasted image 20250525191237.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081448570.png)
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
