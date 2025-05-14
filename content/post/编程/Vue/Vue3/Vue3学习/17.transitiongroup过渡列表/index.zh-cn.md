---
title: 17.transitiongroup过渡列表
description: 17.transitiongroup过渡列表
date: 2024-06-01
slug: 17.transitiongroup过渡列表
image: 202412211946837.png
categories:
    - Vue
---

# transitiongroup过度列表
- 单个节点
- 多个节点，每次只渲染一个
那么怎么同时渲染整个列表，比如使用 `v-for`？在这种场景下，我们会使用 `<transition-group>` 组件。在我们深入例子之前，先了解关于这个组件的几个特点：
- 默认情况下，它不会渲染一个包裹元素，但是你可以通过 `tag` attribute 指定渲染一个元素。
- [过渡模式](https://v3.cn.vuejs.org/guide/transitions-enterleave.html#过渡模式)不可用，因为我们不再相互切换特有的元素。
- 内部元素**总是需要**提供唯一的 `key` attribute 值。
- CSS 过渡的类将会应用在内部的元素中，而不是这个组/容器本身。
```vue
<transition-group>
     <div style="margin: 10px;" :key="item" v-for="item in list">{{ item }</div>
</transition-group>
const list = reactive<number[]>([1, 2, 4, 5, 6, 7, 8, 9])
const Push = () => {
    list.push(123)
}
const Pop = () => {
    list.pop()
}
```
```vue
<template>
  <button @click="Push">插入</button>
  <button @click="Pop">删除</button>
  <div class="box">
    <transition-group>
      <div style="margin: 10px;" :key="item" v-for="item in list">{{ item }}</div>
    </transition-group>
  </div>
</template>
<script setup lang="ts">
import {defineAsyncComponent, reactive, ref} from "vue";
import 'animate.css'
const list = reactive<number[]>([1, 2, 3])
const Push = () => {
  list.push(list.length+1)
}
const Pop = () => {
  list.pop()
}
</script>
<style scoped lang="scss">
.box{
  display: flex;
  flex-wrap: wrap;
}
</style>
```
## 2.列表的移动过渡
**lodash**
```bash
npm install lodash -S
npm install @types/lodash -D
```
`<transition-group>` 组件还有一个特殊之处。除了进入和离开，它还可以为定位的改变添加动画。只需了解新增的 **`v-move` 类**就可以使用这个新功能，它会应用在元素改变定位的过程中。像之前的类名一样，它的前缀可以通过 `name` attribute 来自定义，也可以通过 `move-class` attribute 手动设置
下面代码很酷炫
```vue
<template>
    <div>
        <button @click="shuffle">Shuffle</button>
        <transition-group class="wraps" name="mmm" tag="ul">
            <li class="cell" v-for="item in items" :key="item.id">{{ item.number }}</li>
        </transition-group>
    </div>
</template>
  
<script setup  lang='ts'>
import _ from 'lodash'
import { ref } from 'vue'
let items = ref(Array.apply(null, { length: 81 } as number[]).map((_, index) => {
    return {
        id: index,
        number: (index % 9) + 1
    }
}))
const shuffle = () => {
    items.value = _.shuffle(items.value)
}
</script>
  
<style scoped lang="less">
.wraps {
    display: flex;
    flex-wrap: wrap;
    width: calc(25px * 10 + 9px);
    .cell {
        width: 25px;
        height: 25px;
        border: 1px solid #ccc;
        list-style-type: none;
        display: flex;
        justify-content: center;
        align-items: center;
    }
}
.mmm-move {
    transition: transform 0.8s ease;
}
</style>
```
## 3.状态过度
```vue
<template>
  <div>
    <input step="20" v-model="num.current" type="number"/>
    <div>{{ num.tweenedNumber.toFixed(0) }}</div>
  </div>
</template>
<script setup lang='ts'>
import {reactive, watch} from 'vue'
import gsap from 'gsap'
const num = reactive({
  tweenedNumber: 0,
  current: 0
})
watch(() => num.current, (newVal) => {
  gsap.to(num, {
    duration: 1,
    tweenedNumber: newVal
  })
})
</script>
```
