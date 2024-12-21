---
title: Vue3学习
description: Vue3学习
date: 2023-04-16
slug: Vue3学习
image: 202412211946837.png
categories:
    - Vue
---

## Vue3学习
> 要求，点击按钮，改变数据
> vue2的写法也可以实现
```vue
<template>
  <div class="home">
    <button @click="test">点击</button>
    <br>
    <span>count:{{count}}</span>
  </div>
</template>
<script lang="ts">
import { defineComponent } from 'vue';
export default defineComponent({
  name: 'Home',
  props: {
    msg: String,
  },
  data(){
    return{
      count:0
    }
  },
  methods:{
    test(){
      this.count++;
    }
  }
});
</script>
<style scoped>
button{
  width: 80px;
  height: 30px;
}
</style>
```
> 接下来看vue3的写法
```vue
<template>
  <div class="home">
    <button @click="test">点击</button>
    <br>
    <span>count:{{count}}</span>
  </div>
</template>
<script lang="ts">
import { defineComponent } from 'vue';
export default defineComponent({
  name: 'Home',
  props: {
    msg: String,
  },
  setup(){
    // 变量
    let count=0;
    // 方法
    function test(){
      count++;
      console.log("测试:",count);
    }
    return{
      count:count,
      test
    }
  }
});
</script>
<style scoped>
button{
  width: 80px;
  height: 30px;
}
</style>
```
> 然而此时的页面并没有变化，为什么？
>
> `因为此时的数据并不是响应式数据`
![image-20230416203338618](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304162033788.png)
### ref
> 正确写法
> `ref一般用来定义一个基本类型的响应式数据`
```vue
<template>
  <div class="home">
    <button @click="test">点击</button>
    <br>
    <span>count:{{count}}</span>
  </div>
</template>
<script lang="ts">
import {defineComponent, ref} from 'vue';
export default defineComponent({
  name: 'Home',
  props: {
    msg: String,
  },
  setup(){
    // ref一般用来定义一个基本类型的响应式数据，若需要将一个对象变成响应式数据该用谁呢？（答:reactive）
    let count=ref(0);
    // 方法
    function test(){
      // count是一个ref对象，所以不能直接count++
      // 而上面并没有写{{count.value}}来进行渲染，而是{{count}}
      count.value++;
      console.log("测试:",count);
    }
    return{
      count:count,
      test
    }
  }
});
</script>
<style scoped>
button{
  width: 80px;
  height: 30px;
}
</style>
```
### reactive
> 要求，显示用户的相关信息，点击按钮后更新用户信息
```vue
<template>
  <div class="home">
    <button @click="test">点击</button>
    <br>
    <p>姓名:{{user.name}}</p>
    <p>年龄:{{user.age}}</p>
    <p>座驾:{{user.cars}}</p>
    <ul>
      <li>父亲姓名:{{user.father.name}}</li>
      <li>父亲年龄:{{user.father.age}}</li>
    </ul>
    <ul>
      <li>母亲姓名:{{user.mather.name}}</li>
      <li>母亲年龄:{{user.father.age}}</li>
    </ul>
  </div>
</template>
<script lang="ts">
import {defineComponent, reactive, ref} from 'vue';
export default defineComponent({
  name: 'Home',
  props: {
    msg: String,
  },
  /*
  作用:定义多个数据的响应式
  const proxy=reactive(obj):接收一个普通对象然后返回该普通对象的响应式代理器对象
  响应式转换是“深层的”:会影响对象内部所有嵌套的属性
  内部基于ES6的Proxy实现，通过代理对象操作源对象内部数据都是响应式的
  */
  setup() {
    // 把数据编程响应式的数据
    // 返回的是一个proxy代理对象，被代理的对象是传入的对象
    // testObj是代理对象，而obj是目标对象
    // obj={}
    // const testObj=reactive(obj);
    // obj.name='aaa'
    // 直接使用目标对象的方式来更新属性的值，是无法改变的，只能使用代理对象来进行修改
    // 正确使用方式如下:
    // testObj.name='aaa'
    let user = reactive({
      name: "王明",
      age: 20,
      cars: ['风火轮', '哮天犬', '筋斗云'],
      father: {
        name: "王海",
        age: 45
      },
      mather: {
        name: "李花花",
        age: 43
      }
    });
    // 方法
    const test=()=>{
      user.name+='-';
      user.age++;
      user.cars=['葫芦','飞剑']
    }
    return {
      user: user,
      test
    }
  }
});
</script>
<style scoped>
button {
  width: 80px;
  height: 30px;
}
</style>
```
> 新需求，
>
> user->代理对象
> obj ->目标对象
>
> 1. user对象或者obj对象`添加`一个`新的属性`，哪种方式会影响界面的更新？
> 2. user对象或者obj对象`移除`一个`已存在的属性`，哪种方式会影响界面的更新？
```vue
<template>
  <div class="home">
    <button @click="test">点击</button>
    <br>
    <p>姓名:{{user.name}}</p>
    <p>年龄:{{user.age}}</p>
    <p>性别:{{user.gender}}</p>
    <p>座驾:{{user.cars}}</p>
  </div>
</template>
<script lang="ts">
import {defineComponent, reactive, ref} from 'vue';
export default defineComponent({
  name: 'Home',
  props: {
    msg: String,
  },
  setup() {
    // 为了在使用【obj.gender='男';】时不出现错误信息才采用这种写法
    // let obj:any={
    let obj:any={
      name: "王明",
      age: 20,
      cars: ['风火轮', '哮天犬', '筋斗云']
    };
    let user = reactive(obj);
    // 方法
    const test=()=>{
      // 这种方式页面并没有更新渲染，但是obj中的属性已被添加
      // obj.gender='男';
      // 为了不出现错误提示，可【let obj:any={】或者【let user = reactive<any>(obj);】
      // 这种方式可以更新界面，而且属性也被添加到obj对象了
      user.gender='男'
      // 界面没有更新，但是obj中的属性已被删除
      // delete obj.age;
      // 页面更新渲染,obj中属性已被删除
      delete user.age;
      console.log(user);
    }
    return {
      user: user,
      test
    }
  }
});
</script>
<style scoped>
button {
  width: 80px;
  height: 30px;
}
</style>
```
> `总结：`如果操作代理对象,目标对象中的数据也会随之变化，同时如果想要在操作数据的时候，界面也要跟着重新更新渲染，那么需要操作代理对象
### vue2与vue3响应式的对比
> vue2
- 核心:
  - 对象:通过defineProperty对对象的已有属性值的读取和修改进行劫持(监视/拦截)
  - 数组:通过重写数组更新数组一系列更新元素的方法来实现元素修改的劫持
- 问题
  - 对象直接新添加的属性或删除已有属性,界面不会自动更新
  - 直接通过下标替换元素或更新length,界面不会自动更新arr[1]= {}
> vue3
- 核心:
  - 通过Proxy(代理):拦截对data任意属性的任意(13种)操作,包括属性值的读写,属性的添加，属性的删除等..
  - 通过Reflect(反射):动态对被代理对象的相应属性进行特定的操作
### 响应式数据
> 响应式数据原理:
>
> 直接在vue项目中新建一个html页面
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>响应式的原理</title>
</head>
<body>
<script>
    // 目标对象
    const user = {
        name: '佐助',
        age: 20,
        wife: {
            name: '小樱',
            age: 19
        }
    }
    // 代理对象
    const proxyUser = new Proxy(user, {
        // 获取目标对象的某个属性值
        get(target, prop) {
            console.log("get方法被调用了");
            return Reflect.get(target, prop);
        },
        // 修改目标对象的属性值 & 为目标对象添加新的属性
        set(target, prop, newValue) {
            console.log("set方法被调用了");
            return Reflect.set(target, prop, newValue);
        },
        // 删除目标对象上的某个属性
        deleteProperty(target, prop) {
            console.log("deleteProperty方法被调用了");
            return Reflect.deleteProperty(target, prop);
        }
    });
    // 通过代理对象获取目标对象的信息
    console.log(proxyUser.name);
    // 通过代理对象更新目标对象上的属性值
    proxyUser.name = '鸣人';
    console.log(user);
    // 通过代理对象向目标对象中添加一个新的属性
    proxyUser.gender='男';
    // 通过代理对象删除目标对象中的属性
    delete proxyUser.name;
    console.log(user);
    // 更新目标对象中的某个属性的对象的值
    proxyUser.wife.name='雏田';
    console.log(user);
</script>
</body>
</html>
```
> 响应式数据的测试
```vue
<template>
  <div class="home">
    <button @click="test">点击</button>
    <br>
    <p>姓名:{{user.name}}</p>
    <p>年龄:{{user.age}}</p>
    <p>性别:{{user.gender}}</p>
    <p>座驾:{{user.cars}}</p>
  </div>
</template>
<script lang="ts">
import {defineComponent, reactive, ref} from 'vue';
export default defineComponent({
  name: 'Home',
  props: {
    msg: String,
  },
  setup() {
    // 为了在使用【obj.gender='男';】时不出现错误信息才采用这种写法
    // let obj:any={
    let obj:any={
      name: "王明",
      age: 20,
      cars: ['风火轮', '哮天犬', '筋斗云']
    };
    let user = reactive(obj);
    // 方法
    const test=()=>{
      user.cars[0]='炼丹炉';
      // 数组中添加新元素
      user.cars[3]='葫芦';
    }
    return {
      user: user,
      test
    }
  }
});
</script>
<style scoped>
button {
  width: 80px;
  height: 30px;
}
</style>
```
### setup细节
```vue
<template>
  <div>
    <h2>子级组件</h2>
    <p>msg:{{msg}}</p>
  </div>
</template>
<script lang="ts">
import {defineComponent} from 'vue';
export default defineComponent({
  name: 'Child',
  props:['msg'],
  // setup细节问题:
  // 1.setup在beforeCreate之前就会执行
  setup(){
    console.log("setup执行了");
    // setup中一般都是返回一个对象，对象中的属性和方法可以都在html模板中使用
    return{
    }
  },
  // 数据初始化的生命周期的回调
  beforeCreate() {
    console.log("beforeCreate执行了");
  }
});
</script>
<style scoped>
</style>
```
