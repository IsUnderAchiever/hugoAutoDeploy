---
title: 11.动态组件
description: 11.动态组件
date: 2024-06-01
slug: 11.动态组件
image: 202412211946837.png
categories:
    - Vue
---

```vue
<template>
  <div style="display: flex;">
    <div id="att" v-for="(item,index) in com">
      <div @click="changeIndex(index)">{{item.name}}</div>
    </div>
  </div>
  <component :is="isCom"></component>
</template>
<script setup lang="ts">
import {reactive, ref} from "vue";
import ACom from "@/view/ACom.vue";
import BCom from "@/view/BCom.vue";
import CCom from "@/view/CCom.vue";
let isCom=ref(ACom);
let com = reactive([
  {
    name: 'A组件',
    comName: ACom
  },
  {
    name: 'B组件',
    comName: BCom
  },
  {
    name: 'C组件',
    comName: CCom
  }]);
let changeIndex=(index:number)=>{
  isCom.value=com[index].comName
}
</script>
<style scoped>
#att div {
  width: 50px;
  height: 30px;
  border: 1px solid black;
  text-align: center;
  line-height: 30px;
  margin: 10px 10px;
  cursor: pointer;
}
</style>
```
> 控制台警告如下
```warn
HomeView.vue?t=1689771528237:70 [Vue warn]: Vue received a Component which was made a reactive object. This can lead to unnecessary performance overhead, and should be avoided by marking the component with `markRaw` or using `shallowRef` instead of `ref`. 
Component that was made reactive:  {__name: 'ACom', __hmrId: '2b2bca80', __file: 'D:/project/jetBrains/idea/antd-demo/src/view/ACom.vue', setup: ƒ, render: ƒ} 
  at <HomeView onVnodeUnmounted=fn<onVnodeUnmounted> ref=Ref< null > > 
  at <RouterView> 
  at <App>
```
**这是因为reactive 会进行proxy 代理 而我们组件代理之后毫无用处 节省性能开销 推荐我们使用shallowRef 或者 markRaw 跳过proxy 代理**
> 修改如下
```vue
<template>
  <div style="display: flex;">
    <div id="att" v-for="(item,index) in com">
      <div @click="changeIndex(item,index)">{{ item.name }}</div>
    </div>
  </div>
  <component :is="isCom"></component>
</template>
<script setup lang="ts">
import {markRaw, reactive, ref, shallowRef} from "vue";
import ACom from "@/view/ACom.vue";
import BCom from "@/view/BCom.vue";
import CCom from "@/view/CCom.vue";
let isCom = shallowRef(ACom);
let active=ref(0);
let com = reactive([
  {
    name: 'A组件',
    comName: markRaw(ACom)
  },
  {
    name: 'B组件',
    comName: markRaw(BCom)
  },
  {
    name: 'C组件',
    comName: markRaw(CCom)
  }]);
let changeIndex = (item,index) => {
  isCom.value=item.comName
  active.value=index;
}
</script>
<style scoped>
#att div {
  width: 50px;
  height: 30px;
  border: 1px solid black;
  text-align: center;
  line-height: 30px;
  margin: 10px 10px;
  cursor: pointer;
}
</style>
```
