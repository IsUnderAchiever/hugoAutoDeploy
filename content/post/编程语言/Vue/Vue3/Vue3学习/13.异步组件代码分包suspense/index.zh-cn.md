---
title: 13.异步组件代码分包suspense
description: 13.异步组件代码分包suspense
date: 2024-06-01
slug: 13.异步组件代码分包suspense
image: 202412211946837.png
categories:
    - Vue
---

# 异步组件
> 在大型应用中，我们可能需要将应用分割成小一些的代码块 并且减少主包的体积，这时候就可以使用`异步组件`
## 顶层await
>在setup语法糖里面 使用方法
>
>`<script setup>` 中可以使用顶层 await。结果代码会被编译成 async setup()
```javascript
<script setup>
    const post = await fetch(`/api/post/1`).then(r => r.json())
</script>
```
父组件引用子组件 通过defineAsyncComponent加载异步配合import 函数模式便可以分包
```javascript
<script setup lang="ts">
import { reactive, ref, markRaw, toRaw, defineAsyncComponent } from 'vue'
const Dialog = defineAsyncComponent(() => import('../../components/Dialog/index.vue'))
//完整写法
const AsyncComp = defineAsyncComponent({
  // 加载函数
  loader: () => import('./Foo.vue'),
  // 加载异步组件时使用的组件
  loadingComponent: LoadingComponent,
  // 展示加载组件前的延迟时间，默认为 200ms
  delay: 200,
  // 加载失败后展示的组件
  errorComponent: ErrorComponent,
  // 如果提供了一个 timeout 时间限制，并超时了
  // 也会显示这里配置的报错组件，默认值是：Infinity
  timeout: 3000
})
```
# suspense
`<suspense>` 组件有两个插槽。它们都只接收一个直接子节点。`default` 插槽里的节点会尽可能展示出来。如果不能，则展示 `fallback` 插槽里的节点。
```javascript
     <Suspense>
            <template #default>
                <Dialog>
                    <template #default>
                        <div>我在哪儿</div>
                    </template>
                </Dialog>
            </template>
            <template #fallback>
                <div>loading...</div>
            </template>
        </Suspense>
```
## 案例
> 在数据请求到达之前，显示骨架，在获取到数据后，替换骨架
> public/data.json
```json
{
  "user": {
    "name": "哆啦A梦",
    "age": 15,
    "url": "https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fp7.itc.cn%2Fq_70%2Fimages03%2F20200924%2F6a36f60e6c034d78b93901b57afca05d.jpeg&refer=http%3A%2F%2Fp7.itc.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1656899135&t=c53658ab441623ce626434ff0b974318",
    "desc": "大雄的好朋友"
  }
}
```
>SkeletonVue.vue
```vue
<template>
  <div id="att">
    <div class="content">
      <span>姓名:正在请求中...</span>
    </div>
    <div class="content">
      <span>年龄:正在请求中...</span>
    </div>
    <div class="header">
      头像:<img alt="正在请求中..." src="">
    </div>
    <div class="content">
      简介:<span>正在请求中...</span>
    </div>
  </div>
</template>
```
>Sync.vue
```vue
<template>
  <div id="att">
    <div class="content">
      <span>姓名:{{ user.name }}</span>
    </div>
    <div class="content">
      <span>年龄:{{ user.age }}</span>
    </div>
    <div class="header">
      头像:<img :src="user.url">
    </div>
    <div class="content">
      简介:<span>{{ user.desc }}</span>
    </div>
  </div>
</template>
<script setup lang="ts">
import {ref} from "vue";
import axios from "axios";
interface UserProps {
  user: {
    name: string,
    age: number,
    url: string;
    desc: string
  }
}
const {data} = await axios.get<UserProps>('./data.json')
const user = ref<{
  name: string,
  age: number,
  url: string;
  desc: string
}>();
user.value = data.user
console.log('data:', data);
</script>
<style scoped>
.header > img {
  width: 50px;
  height: 50px;
}
</style>
```
> APP.vue
```vue
<template>
  <div>
    <Suspense>
      <template #default>
        <Sync>
        </Sync>
      </template>
      <template #fallback>
        <SkeletonVue>
        </SkeletonVue>
      </template>
    </Suspense>
  </div>
</template>
<script setup lang="ts">
import {defineAsyncComponent, reactive, ref} from "vue";
import SkeletonVue from "@/view/SkeletonVue.vue";
const Sync = defineAsyncComponent(() => import('@/view/Sync.vue'))
</script>
```
