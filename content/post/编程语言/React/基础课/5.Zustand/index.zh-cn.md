---
title: Zustand
description: Zustand
date: 2025-03-03
slug: Zustand
image: 202412212156937.png
categories:
    - React
---

> 本文转载自[B站 柴柴前端教书匠](https://www.bilibili.com/video/BV1Z44y1K7Fj/?spm_id_from=333.1387.0.0)

# Zustand

![202503032202934.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202503032202934.png)



> 一个简单的，快速的状态管理解决方案，api设计基于函数式和hooks
>

# 1. 基础使用
> 让我们实现一个非常简单的计数器案例完成我们的第一个store
>

1- 创建一个counterStore

```javascript
import create from 'zustand'

const useCounterStore = create((set) => ({
  // 数据
  count: 0,
  // 修改数据的方法
  increase: () => set(state => ({ count: state.count + 1 })),
  decrease: () => set(state => ({ count: state.count - 1 }))
}))


export default useCounterStore
```



2- 绑定到组件

```jsx
import useCounterStore from './store'

const App = () => {
  const count = useCounterStore((state) => state.count)
  const decrease = useCounterStore((state) => state.increase)
  const increase = useCounterStore((state) => state.decrease)
  return (
    <div>
      <button onClick={decrease}>+</button>
      <span>{count}</span>
      <button onClick={increase}>-</button>
    </div>
  )
}

export default App
```

# 2. 异步支持
1- 创建异步action

```javascript
import create from 'zustand'

const fetchApi = () => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(['vue', 'react'])
    }, 2000)
  })
}

const useListStore = create((set) => ({
  // 数据
  list: [],
  // 修改数据的方法
  fetchList: async () => {
    const res = await fetchApi()
    set({ list: res })
  }
}))


export default useListStore
```

2- 绑定组件

```jsx
import { useEffect } from 'react'
import useListStore from './store'

const App = () => {
  const list = useListStore((state) => state.list)
  const fetchList = useListStore((state) => state.fetchList)
  useEffect(() => {
    fetchList()
  }, [])
  return (
    <div>
      {JSON.stringify(list)}
    </div>
  )
}

export default App
```

# 3. 增加调试
> 简单的调试我们可以安装一个 名称为 simple-zustand-devtools 的调试工具
>

1- 安装调试包

```bash
$ yarn add simple-zustand-devtools
```



2- 配置调试工具

```javascript
import create from 'zustand'

// 导入核心方法
import { mountStoreDevtool } from 'simple-zustand-devtools'

const useStore = create((set) => ({}))

// 开发环境开启调试
if (process.env.NODE_ENV === 'development') {
  // 第一个参数为调试的store标识
  mountStoreDevtool('counterStore', useStore)
}


export default useStore
```

3- 打开 React调试工具

![202503032202085.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202503032202085.png)



> 更新: 2023-10-18 14:16:10  
> 原文: <https://www.yuque.com/fechaichai/qeamqf/ngfwmf>