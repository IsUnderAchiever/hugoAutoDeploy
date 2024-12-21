---
title: 20.TSX
description: 20.TSX
date: 2023-07-27
slug: 20.TSX
image: 202412211946837.png
categories:
    - Vue
---

# TSX
> 安装、配置
```bash
npm install @vitejs/plugin-vue-jsx -D
```
> vite.config.ts
```ts
import vueJsx from '@vitejs/plugin-vue-jsx'
plugins: [
    vueJsx()
]
```
## 写法1
```tsx
export default function (){
    return (
        <div>测试tsx</div>
    )
}
```
```vue
<template>
  <Test/>
</template>
<script setup lang='ts'>
import Test from "@/view/Test.tsx";
</script>
```
## 写法2
```tsx
import {defineComponent} from "vue";
export default defineComponent({
    data(){
        let age=23
        return{
            age
        }
    },
    render(){
        return(
            <div>age:{this.age}</div>
        )
    }
})
```
> methods
```tsx
import {defineComponent} from "vue";
export default defineComponent({
    data(){
        let age=23
        return{
            age
        }
    },
    methods:{
        test(){
            console.log('测试方法运行');
        }
    },
    render(){
        return(
            <div onClick={()=>this.test()}>age:{this.age}</div>
        )
    }
})
```
## 写法3
```tsx
import {defineComponent} from "vue";
export default defineComponent({
    setup(){
        let age=23;
        let test=()=>{
            console.log('测试方法已运行');
        }
        return ()=>(
            <div onClick={test}>age:{age}</div>
        )
    }
})
```
## v-show
> 支持
```tsx
import {defineComponent} from "vue";
export default defineComponent({
    setup(){
        let flag:boolean=false;
        return ()=>(
            <div v-show={flag}>hhh</div>
        )
    }
})
```
> ref
>
> 在html标签中，需要使用`.value`来获取
```tsx
import {defineComponent, ref} from "vue";
export default defineComponent({
    setup(){
        let flag=ref<boolean>(false);
        return ()=>(
            <div v-show={flag.value}>hhh</div>
        )
    }
})
```
## v-if
> 不支持
>
> `三元表达式写法`
```tsx
import {defineComponent, ref} from "vue";
export default defineComponent({
    setup(){
        let flag=ref<boolean>(false);
        return ()=>(
            <div>{flag.value?'显示':'隐藏'}</div>
        )
    }
})
```
## v-for
> 不支持
>
> `数组.map`遍历写法
```tsx
import {defineComponent} from "vue";
export default defineComponent({
    setup(){
        let data=[
            {
                name:'aaa',
                age:12
            },
            {
                name:'bbb',
                age:22
            }
        ]
        return ()=>(
            <div>{data.map(item=>{
                return <div>name:{item.name}--age:{item.age}</div>
            })}</div>
        )
    }
})
```
## v-bind
```tsx
import {defineComponent} from "vue";
export default defineComponent({
    setup() {
        let data = [
            {
                name: 'aaa',
                age: 12
            },
            {
                name: 'bbb',
                age: 22
            }
        ]
        return () => (
            <div>{data.map(item => {
                return <div name={item.name}>{item.name}</div>
            })}</div>
        )
    }
})
```
## props/emit
> Test.tsx
```tsx
import {defineComponent} from "vue";
interface Props {
    name?: string
}
export default defineComponent({
    props: {
        name: String
    },
    emits: ['on-click'],
    setup(props: Props, {emit}) {
        let emitFun = (content: string) => {
            console.log('触发了emit');
            emit('on-click', content)
        };
        return () => (
            <>
                <p>{props.name}</p>
                <button onClick={() => emitFun('测试内容')}>点击</button>
            </>
        )
    }
})
```
> Home.vue
```vue
<template>
  <Test name="马小跳" @on-click="getEmit"/>
</template>
<script setup lang='ts'>
import Test from "@/view/Test.tsx";
let getEmit=(item:string)=>{
  console.log('item:',item);
}
</script>
```
## 插槽
```tsx
import {defineComponent} from "vue";
export default defineComponent({
    setup() {
        const A = (_, {slots}) => (
            <>
                <div>{slots.default ? slots.default() : '默认值'}</div>
            </>
        )
        const slot = {
            default: () => (<div>default slots</div>)
        }
        return () => (
            <>
                <A v-slots={slot}/>
            </>
        )
    }
})
```
