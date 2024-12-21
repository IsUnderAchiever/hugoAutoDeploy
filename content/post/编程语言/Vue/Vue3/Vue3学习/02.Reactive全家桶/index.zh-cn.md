---
title: 02.Reactive全家桶
description: 02.Reactive全家桶
date: 2023-07-13
slug: 02.Reactive全家桶
image: 202412211946837.png
categories:
    - Vue
---

# 02.Reactive全家桶
## reactive
> reactive('')不支持string类型
>
> 查看源码可知，只支持引用类型，object、map、set
>
> 而ref都支持
![image-20230713221217293](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132214501.png)
![image-20230713221436279](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132214316.png)
>reactive在赋值时无需使用`.value`
>
>而ref需要
```vue
<template>
  <div>
    <p>{{user.name}}--{{user.gender}}</p>
  </div>
  <a-button danger @click="test1">test1</a-button>
</template>
<script setup lang="ts">
import {defineComponent, reactive, ref} from 'vue';
import {Button} from "ant-design-vue";
let user=reactive({
  name:'马小跳',
  gender:'男'
});
let test1=()=>{
  user.name='洛洛';
  console.log(user);
}
</script>
<style scoped>
</style>
```
> 阻止默认事件
>
> `@click.prevent`
```html
  <form>
      <button @click.prevent="test1">test2</button>
  </form>
```
> reactive是经过proxy代理的对象，直接赋值会失去响应式
```vue
<template>
  <div>
    <p v-for="user in users">{{user}}</p>
  </div>
  <a-button danger @click="test1">test1</a-button>
</template>
<script setup lang="ts">
import {defineComponent, reactive, ref} from 'vue';
let users=reactive<string[]>([]);
let test1=()=>{
  // 假设res为获取到的数据
  let res=['小满','辣辣','路西'];
  // 失去响应式
  users=res;
  console.log(users);
}
</script>
```
> 正确方法如下
**方法1**
```vue
<template>
  <div>
    <p v-for="user in users">{{user}}</p>
  </div>
  <a-button danger @click="test1">test1</a-button>
</template>
<script setup lang="ts">
import {defineComponent, reactive, ref} from 'vue';
let users=reactive<string[]>([]);
let test1=()=>{
  // 假设res为获取到的数据
  let res=['小满','辣辣','路西'];
  // 正确方法1
  // 结构后赋值
  users.push(...res);
  console.log(users);
}
</script>
```
**方法2**
```vue
<template>
  <div>
    <p v-for="user in users.arr">{{user}}</p>
  </div>
  <a-button danger @click="test1">test1</a-button>
</template>
<script setup lang="ts">
import {defineComponent, reactive, ref} from 'vue';
let users=reactive<{arr:string[]}>({arr:[]});
let test1=()=>{
  // 假设res为获取到的数据
  let res=['小满','辣辣','路西'];
  // 正确方法2
  users.arr=res;
  console.log(users);
}
</script>
```
## readonly
> 无法修改readonly修饰的值，报错`Attempt to assign to const or readonly variable `
![image-20230713223221913](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132232965.png)
> 但是会受`reactive`影响
```vue
<template>
  <a-button danger @click="test1">test1</a-button>
</template>
<script setup lang="ts">
import {defineComponent, reactive, readonly, ref} from 'vue';
import {message} from "ant-design-vue";
let user=reactive({name:'小荷'});
const readUser=readonly(user);
let test1=()=>{
  user.name='小满';
  message.success(readUser.name)
}
</script>
```
![image-20230713223406071](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132234118.png)
## shallowReactive
> 与`shallowRef`类似
>
> `只能对浅层的数据 如果是深层的数据只会改变值 不会改变视图`
```vue
<template>
  {{ user.name.firstName }}{{ user.name.lastName }}--{{user.gender}}
  <a-button danger @click="test1">test1</a-button>
</template>
<script setup lang="ts">
import {defineComponent, reactive, readonly, ref, shallowReactive} from 'vue';
import {message} from "ant-design-vue";
let user = shallowReactive({
  name: {
    firstName: '赵',
    lastName: '子龙'
  },
  gender: '男'
});
let test1 = () => {
  user.name.firstName = '貂'
  user.name.lastName = '蝉'
  user.gender = '女'
}
</script>
```
![image-20230713224921165](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132249201.png)
> 不是说好不改变视图的吗？
>
> 和`shallowRef`一样，shallowReactive也会受reactive影响
>
> 当gender属性改变时，触发了强制更新
>
> 所以如下代码即可
```vue
<template>
  {{ user.name.firstName }}{{ user.name.lastName }}--{{user.gender}}
  <a-button danger @click="test1">test1</a-button>
</template>
<script setup lang="ts">
import {defineComponent, reactive, readonly, ref, shallowReactive} from 'vue';
import {message} from "ant-design-vue";
let user = shallowReactive({
  name: {
    firstName: '赵',
    lastName: '子龙'
  },
  gender: '男'
});
let test1 = () => {
  user.name.firstName = '貂'
  user.name.lastName = '蝉'
  // user.gender = '女'
}
</script>
```
