---
title: 07.watchEffect高级侦听器
description: 07.watchEffect高级侦听器
date: 2023-07-17
slug: 07.watchEffect高级侦听器
image: 202412211946837.png
categories:
    - Vue
---

# watchEffect高级侦听器
> [原博客](https://xiaoman.blog.csdn.net/article/details/122802130)
> 执行传入的函数，同时响应式追踪其依赖，当依赖改变时重新运行该函数	
```typescript
import {ref, watchEffect} from 'vue';
let message = ref<number>(0)
watchEffect(() => {
  console.log('message', message.value);
})
```
**清除副作用**
> 在触发监听之前会调用一个函数来处理逻辑，如防抖
```typescript
<script setup lang="ts">
import { watchEffect, ref } from 'vue'
let message = ref<number>(0)
watchEffect((beforeTest) => {
  beforeTest(()=>{
    console.log('测试前');
  })
  console.log('message', message.value);
})
</script>
```
**停止跟踪**
> watchEffect 返回一个函数 调用之后将停止更新
```vue
<template>
  <div>
    <div>{{ message }}</div>
    <button @click="message++">测试</button>
    <button @click="stopWatch">停止监听</button>
  </div>
</template>
<script setup lang="ts">
import { watchEffect, ref } from 'vue'
let message = ref<number>(0)
let stopWatch=watchEffect((beforeTest) => {
  beforeTest(()=>{
    console.log('测试前');
  })
  console.log('message', message.value);
})
</script>
```
**刷新时机**
|          |       pre        |    sync    |       post       |
| :------: | :--------------: | :--------: | :--------------: |
| 更新时机 | 组件`更新前`执行 | `同步`执行 | 组件`更新后`执行 |
```vue
<template>
  <div>
    <div>{{ message }}</div>
    <button @click="message++">测试</button>
    <button @click="stopWatch">停止监听</button>
  </div>
</template>
<script setup lang="ts">
import {watchEffect, ref} from 'vue'
let message = ref<number>(0)
let stopWatch = watchEffect((beforeTest) => {
  beforeTest(() => {
    console.log('测试前');
  })
  console.log('message', message.value);
}, {
  flush: 'post',
  onTrigger() {
    console.log('更新前');
  }
})
</script>
```
