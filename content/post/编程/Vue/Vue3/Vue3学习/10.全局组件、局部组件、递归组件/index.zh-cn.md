---
title: 10.全局组件、局部组件、递归组件
description: 10.全局组件、局部组件、递归组件
date: 2024-02-16
slug: 10.全局组件、局部组件、递归组件
image: 202412211946837.png
categories:
    - Vue
---

# 配置全局组件
> 这里封装一个全局组件`Global.vue`
```vue
<template>
  <div id="global-component" align="center">
    全局组件
  </div>
</template>
<style scoped>
#global-component{
  border: 1px solid black;
  margin: 10px 10px;
  line-height: 25px;
  width: 80px;
  height: 25px;
}
</style>
```
> 在main.ts引入注册，类似element注册
```typescript
// main.ts
// 如果您正在使用CDN引入，请删除下面一行。
import * as ElementPlusIconsVue from '@element-plus/icons-vue'
const app = createApp(App)
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
  app.component(key, component)
}
```
```typescript
import {createApp} from 'vue'
import App from './App.vue'
import Global from "@/view/Global.vue";
const app = createApp(App)
app.component('Global',Global)
app.mount('#app')
```
在其他vue页面 立即使用即可 无需引入
```vue
<template>
  <Global/>
</template>
<script setup lang="ts">
</script>
```
# 批量注册全局组件
可以参考element ui 其实就是遍历一下然后通过 app.component 注册
![img](https://img-blog.csdnimg.cn/5b4de39d5f8b48c38f751d510b4f1f61.png)
 
# 配置局部组件
```vue
<template>
  <div class="wraps">
    <layout-menu :flag="flag" @on-click="getMenu" @on-toogle="getMenuItem" :data="menuList" class="wraps-left"></layout-menu>
    <div class="wraps-right">
      <layout-header> </layout-header>
      <layout-main class="wraps-right-main"></layout-main>
    </div>
  </div>
</template>
<script setup lang="ts">
import { reactive,ref } from "vue";
import layoutHeader from "./Header.vue";
import layoutMenu from "./Menu.vue";
import layoutMain from "./Content.vue";
```
就是在一个组件内（A） 通过import 去引入别的组件(B) 称之为局部组件
应为B组件只能在A组件内使用 所以是局部组件
如果C组件想用B组件 就需要C组件也手动import 引入 B 组件
# 配置递归组件
原理跟我们写js递归是一样的 自己调用自己 通过一个条件来结束递归 否则导致内存泄漏
案例递归树
在父组件配置数据结构 数组对象格式 传给子组件
```vue
<template>
  <TreeCom :data="data"/>
</template>
<script setup lang="ts">
import {reactive} from "vue";
import TreeCom from "@/view/TreeCom.vue";
type TreeProps = {
  name: string,
  icon?: string,
  children?: TreeProps[] | []
}
const data = reactive<TreeProps[]>([
  {
    name: "no.1",
    children: [
      {
        name: "no.1-1",
        children: [
          {
            name: "no.1-1-1",
          },
        ],
      },
    ],
  },
  {
    name: "no.2",
    children: [
      {
        name: "no.2-1",
      },
    ],
  },
  {
    name: "no.3",
  },
])
</script>
```
子组件接收值 第一个script
```typescript
type TreeProps = {
  name: string,
  icon?: string,
  children?: TreeProps[] | []
}
let props=defineProps<{
  data:TreeProps[]
}>();
```
------
# 子组件增加一个script 定义组件名称为了 递归用 
## 给我们的组件定义名称有好几种方式
### 1.在增加一个script 通过 export 添加name
```
<script lang="ts">
export default {
  name:"TreeItem"
}
</script>
```
### 2.直接使用文件名当组件名
![img](https://img-blog.csdnimg.cn/42572d5f39b94ccca67608ea8b0b8d6a.png)
3.使用插件 
[unplugin-vue-macros/README-zh-CN.md at 722a80795a6c7558debf7c62fd5f57de70e0d0bf · sxzz/unplugin-vue-macros · GitHub](https://github.com/sxzz/unplugin-vue-macros/blob/HEAD/packages/define-options/README-zh-CN.md)
 unplugin-vue-define-options
```
import DefineOptions from 'unplugin-vue-define-options/vite'
import Vue from '@vitejs/plugin-vue'
export default defineConfig({
  plugins: [Vue(), DefineOptions()],
})
```
 ts支持
```
 "types": ["unplugin-vue-define-options/macros-global"],
```
![img](https://img-blog.csdnimg.cn/703a8f3167e8448eb0fc1616884dee1a.png)
 
template 
TreeItem 其实就是当前组件 通过import 把自身又引入了一遍 如果他没有children 了就结束
```
  <div style="margin-left:10px;" class="tree">
    <div :key="index" v-for="(item,index) in data">
      <div @click='clickItem(item)'>{{item.name}}
    </div>
    <TreeItem @on-click='clickItem' v-if='item?.children?.length' :data="item.children"></TreeItem>
  </div>
  </div>
```
 ![img](https://img-blog.csdnimg.cn/e70cd74130e84cd2a6a4be220ca2a312.png)
```vue
<template>
  <div v-for="(item,index) in data">
    <input type="checkbox"><span>{{item.name}}</span>
    <TreeCom v-if="item?.children?.length" :data="item.children"/>
  </div>
</template>
<script setup lang="ts">
type TreeProps = {
  name: string,
  icon?: string,
  children?: TreeProps[] | []
}
let props=defineProps<{
  data:TreeProps[]
}>();
</script>
<style scoped>
div{
  margin-left: 10px;
}
</style>
```
