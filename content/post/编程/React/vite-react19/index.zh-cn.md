---
title: React19 + Vite + TailwindCSS4 + AntD5
description: React
date: 2025-04-20
slug: React19 + Vite + TailwindCSS4 + AntD5
image: 202412212156937.png
categories:
  - React
---

# React19 + Vite + TailwindCSS4 + AntD5

## 新建 Vite 项目

```shell
npm create vite@latest
# 安装tailwindcss4
pnpm install tailwindcss @tailwindcss/vite
# 安装antd
pnpm install antd --save
# 适配react19
pnpm add @ant-design/v5-patch-for-react-19 --save
```

> 配置 tailwindcss

配置`vite.config.ts`文件

```ts
import { defineConfig } from 'vite'
import tailwindcss from '@tailwindcss/vite'
export default defineConfig({
  plugins: [tailwindcss()],
})
```

在`全局css样式(index.css)`中引入 tailwindcss

```css
@import 'tailwindcss';
```

> antd 适配 react19

在`应用入口处(main.tsx)`引入兼容包

```ts
import '@ant-design/v5-patch-for-react-19'
```
