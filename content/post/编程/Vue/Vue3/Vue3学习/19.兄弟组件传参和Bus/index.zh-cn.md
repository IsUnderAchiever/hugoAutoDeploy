---
title: 19.兄弟组件传参和Bus
description: 19.兄弟组件传参和Bus
date: 2023-07-26
slug: 19.兄弟组件传参和Bus
image: 202412211946837.png
categories:
    - Vue
---

# 兄弟组件传参和Bus
> Home
```vue
<template>
  <A @on-click="getFlag"/>
  <B :flag="flag"/>
</template>
<script setup lang='ts'>
import {provide, reactive, readonly, ref} from 'vue'
import A from './A.vue'
import B from './B.vue'
const flag=ref<boolean>(false)
const getFlag=(props:boolean)=>{
  flag.value=props
}
</script>
```
> A
```vue
<template>
<button @click="emitB">传递flag</button>当前flag值为{{flag}}
</template>
<script setup lang="ts">
import {inject, reactive, ref} from 'vue';
import type {Ref} from "vue";
let flag=ref<boolean>(true)
const emit=defineEmits(['on-click'])
const emitB=()=>{
  flag.value=!flag.value
  emit('on-click',flag.value)
}
</script>
<style scoped>
button{
  width: 80px;
  height: 30px;
}
</style>
```
> B
```vue
<template>
  <div>B接收到值了:{{flag}}</div>
</template>
<script setup lang="ts">
import {inject, reactive, ref} from 'vue';
import type {Ref} from "vue";
type Props={
  flag:boolean
}
defineProps<Props>();
</script>
```
## Event Bus
> 安装`mitt`库
```bash
npm install mitt -S
```
```ts
import {createApp} from 'vue'
import App from './App.vue'
import mitt from "mitt";
const Mit = mitt()
//TypeScript注册
// 由于必须要拓展ComponentCustomProperties类型才能获得类型提示
declare module "vue" {
    export interface ComponentCustomProperties {
        $Bus: typeof Mit
    }
}
const app = createApp(App)
//Vue3挂载全局API
app.config.globalProperties.$Bus = Mit
```
**使用方法通过emit派发， on 方法添加事件，off 方法移除，clear 清空所有** 
A组件派发（emit）
```vue
<template>
    <div>
        <h1>我是A</h1>
        <button @click="emit1">emit1</button>
        <button @click="emit2">emit2</button>
    </div>
</template>
<script setup lang='ts'>
import { getCurrentInstance } from 'vue'
const instance = getCurrentInstance();
const emit1 = () => {
    instance?.proxy?.$Bus.emit('on-num', 100)
}
const emit2 = () => {
    instance?.proxy?.$Bus.emit('*****', 500)
}
</script>
<style>
</style>
```
B组件监听（on）
```vue
<template>
    <div>
        <h1>我是B</h1>
    </div>
</template>
<script setup lang='ts'>
import { getCurrentInstance } from 'vue'
const instance = getCurrentInstance()
instance?.proxy?.$Bus.on('on-num', (num) => {
    console.log(num,'===========>B')
})
</script>
<style>
</style>
```
监听所有事件（ on("*") ）
```vue
instance?.proxy?.$Bus.on('*',(type,num)=>{
    console.log(type,num,'===========>B')
})
```
移除监听事件（off）
```vue
const Fn = (num: any) => {
    console.log(num, '===========>B')
}
instance?.proxy?.$Bus.on('on-num',Fn)//listen
instance?.proxy?.$Bus.off('on-num',Fn)//unListen
```
清空所有监听（clear）
```vue
instance?.proxy?.$Bus.all.clear()
```
