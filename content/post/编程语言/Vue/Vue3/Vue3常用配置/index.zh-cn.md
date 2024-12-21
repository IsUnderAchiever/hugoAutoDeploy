---
title: Vue3常用配置
description: Vue3常用配置
date: 2023-04-30
slug: Vue3常用配置
image: 202412211946837.png
categories:
    - Vue
---

# vue3快速上手
```bash
vue create vue3-demo
```
![image-20230430091624858](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304300916024.png)
![创建vue3项目](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304291523418.png)
## ElementPlus
```bash
npm install element-plus --save
```
> 完整引入
```js
// main.ts
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import App from './App.vue'
const app = createApp(App)
app.use(ElementPlus)
app.mount('#app')
```
> 安装图标
```
npm install @element-plus/icons-vue
```
**图标注册**
```js
// main.ts
// 如果您正在使用CDN引入，请删除下面一行。
import * as ElementPlusIconsVue from '@element-plus/icons-vue'
const app = createApp(App)
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
  app.component(key, component)
}
```
> `综合`
```js
// main.ts
import {createApp} from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import * as ElementPlusIconsVue from '@element-plus/icons-vue'
const app = createApp(App)
app.use(store)
app.use(router)
app.use(ElementPlus)
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
    app.component(key, component)
}
app.mount('#app')
```
## axios
> 安装axios
>
> 以下内容参考【[程序员青戈](https://blog.csdn.net/xqnode)】的博客
```bash
npm install axios -s
```
**utils/request.js**
```js
import axios from 'axios'
const request = axios.create({
	baseURL: '/api',  // 注意！！ 这里是全局统一加上了 '/api' 前缀，也就是说所有接口都会加上'/api'前缀在，页面里面写接口的时候就不要加 '/api'了，否则会出现2个'/api'，类似 '/api/api/user'这样的报错，切记！！！
    timeout: 5000
})
// request 拦截器
// 可以自请求发送前对请求做一些处理
// 比如统一加token，对请求参数统一加密
request.interceptors.request.use(config => {
    config.headers['Content-Type'] = 'application/json;charset=utf-8';
  
 // config.headers['token'] = user.token;  // 设置请求头
    return config
}, error => {
    return Promise.reject(error)
});
// response 拦截器
// 可以在接口响应后统一处理结果
request.interceptors.response.use(
    response => {
        let res = response.data;
        // 如果是返回的文件
        if (response.config.responseType === 'blob') {
            return res
        }
        // 兼容服务端返回的字符串数据
        if (typeof res === 'string') {
            res = res ? JSON.parse(res) : res
        }
        return res;
    },
    error => {
        console.log('err' + error) // for debug
        return Promise.reject(error)
    }
)
export default request
```
**vue.config.js**
```js
// 跨域配置
module.exports = {
    devServer: {                //记住，别写错了devServer//设置本地默认端口  选填
        port: 10020,
        proxy: {                 //设置代理，必须填
            '/api': {              //设置拦截器  拦截器格式   斜杠+拦截器名字，名字可以自己定
                target: 'http://localhost:10010',     //代理的目标地址
                changeOrigin: true,              //是否设置同源，输入是的
                pathRewrite: {                   //路径重写
                    '^/api': ''                     //选择忽略拦截器里面的内容
                }
            }
        }
    }
}
```
## vuex
> 安装vuex（已安装，main.ts配置参考element plus处）
```bash
npm install vuex --save-dev
```
> 安装持久化插件
```bash
npm i vuex-persistedstate
```
> 关闭严格模式
>
> `tsconfig.json`里将`"strict": false`设置为`"strict": true`
**sotore/modules/userts**
```js
export default {
    namespaced: true, // 为每个模块添加一个前缀名，保证模块命明不冲突
    // state中存放的就是全局共享的数据
    // sessionStorage.getItem('userState')?JSON.parse(sessionStorage.getItem('userState'))
    state: null != window.sessionStorage.getItem('state') ? JSON.parse(sessionStorage.getItem('state')) : {
        user: {
            name: "",
            age: null,
            gender: ""
        }
    },
    // 取值的方法，计算属性
    getters: {
        getUser(state) {
            return state.user;
        }
    },
    // 可以修改state值的方法，同步阻塞
    mutations: {
        // 传入user对象，更新全局的user
        updateUser(state, user) {
            state.user = user;
        }
    },
    // 异步调用mutations方法
    actions: {
        asyncUpdateUser(context, user) {
            context.commit('updateUser', user);
        }
    }
}
```
**store/indexts**
```js
import {createStore} from 'vuex'
import createPersistedState from 'vuex-persistedstate'
import user from './modules/user'
export default createStore({
  state: {
  },
  getters: {
  },
  mutations: {},
  actions: {},
  modules: {
    user
  },
  plugins: [
    // 配置vuex插件
    // 使用模块提供的方法
    createPersistedState({
      // 默认存储在localStorage 现改为sessionStorage
      storage: window.sessionStorage,
      // 本地存储数据的键名
      // 配置存储持久化的key键名
      key: 'user',
      // 指定需要存储的模块，如果是模块下具体的数据需要加上模块名称，如user.token
      // paths数组可以选择配置持久化的模块，不写代表所有
      // paths: ['user', 'cart']
    })
  ]
})
```
> 测试页面
```vue
<template>
  <div class="hello">
    <el-button icon="Search" circle @click="testVuex" />
    <br>
    <el-button type="success" @click="clearDataFromVuex">删除vuex内的数据</el-button>
    <br>
    <h3>name:{{ store.state.user.user.name }}</h3>
    <h3>age:{{ store.state.user.user.age }}</h3>
    <h3>gender:{{ store.state.user.user.gender }}</h3>
  </div>
</template>
<script lang="ts">
import {defineComponent, reactive, ref, computed} from 'vue';
import {useStore} from "vuex";
export default defineComponent({
  name: 'HelloWorld',
  props: {
    msg: String,
  },
  setup() {
    const store = useStore();
    const testVuex = () => {
      // 使用这种方式可以改变值，但是无法实现数据的持久化
      // store.state.user.user.name='假面骑士decade'
      // store.state.user.user.age=22
      // store.state.user.user.gender='男'
		
      // 下面两种方法可以实现vuex的数据持久化
      // store.commit('user/updateUser',{
      //   name: "Decade",
      //   age: 22,
      //   gender: "男"
      // });
      store.dispatch('user/asyncUpdateUser', {
        name: "假面骑士",
        age: 18,
        gender: "女"
      });
    }
    const clearDataFromVuex=()=>{
      store.dispatch('user/asyncUpdateUser', {});
    };
    return {
      testVuex,
      clearDataFromVuex,
      store
    }
  }
});
</script>
<style scoped lang="less">
</style>
```
> 简单集成ts
```vue
<template>
  <div class="hello">
    <el-button icon="Search" circle @click="testVuex"/>
    <br>
    <el-button type="success" @click="clearDataFromVuex">删除vuex内的数据</el-button>
    <br>
    <h3>name:{{ store.state.user.user.name }}</h3>
    <h3>age:{{ store.state.user.user.age }}</h3>
    <h3>gender:{{ store.state.user.user.gender }}</h3>
  </div>
</template>
<script lang="ts">
import {defineComponent, reactive, ref, computed} from 'vue';
import {useStore} from "vuex";
interface userForm {
  name: string,
  age: number,
  gender: string
}
export default defineComponent({
  name: 'HelloWorld',
  props: {
    msg: String,
  },
  setup() {
    const store = useStore();
    const testVuex = (user: userForm) => {
      user = {
        name: "Decade",
        age: 22,
        gender: "男"
      };
      store.commit('user/updateUser', user);
    }
    const clearDataFromVuex = () => {
      store.dispatch('user/asyncUpdateUser', {});
    };
    return {
      testVuex,
      clearDataFromVuex,
      store
    }
  }
});
</script>
<style scoped lang="less">
</style>
```
## cookie
```bash
npm install vue-cookies --save
```
> main.ts
```js
import {createApp} from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import * as ElementPlusIconsVue from '@element-plus/icons-vue'
import '@/assets/css/global.css'
import VueCookies from 'vue-cookies'
const app = createApp(App)
app.use(store)
app.use(router)
app.use(ElementPlus)
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
    app.component(key, component)
}
// @ts-ignore
app.config.globalProperties.$cookies = VueCookies;
app.mount('#app')
```
```vue
<template>
  <div class="cookie-test">
    <el-button @click="cookieInTest">点击存入cookie</el-button>
    <el-button @click="cookieDeleteTest">点击删除cookie</el-button>
    <br>
    <span>cookieForm:{{cookieForm}}</span>
    <br>
    <span>mydata:{{myData}}</span>
  </div>
</template>
<script lang="ts">
import {defineComponent, getCurrentInstance, ref,onMounted} from 'vue';
import VueCookies from 'vue-cookies'
export default defineComponent({
  name: '',
  setup(){
    let cookieForm=ref();
    let internalInstance = getCurrentInstance();
    let cookies = internalInstance.appContext.config.globalProperties.$cookies
    const cookieInTest=()=>{
      // 存入，可加时间限制(秒)
      // cookies.set('test', 'Hello world!',5);
      // cookies.set('test', 'Hello world!',5000);
      // 获取
      // cookieForm.value=cookies.get('test');
      // 将数据保存到cookie中
      cookies.set('myData', '测试数据', '30d')
      myData.value=cookies.get('myData');
      console.log("set--mydata,",myData.value);
    }
    const cookieDeleteTest=()=>{
      // cookies.remove('test');
      // 获取
      // cookieForm.value=cookies.get('test');
      cookies.remove('myData');
      myData.value=cookies.get('myData');
    }
    const createdData=()=>{
      // cookieForm = cookies.get('myData')
    }
    const myData = ref('')
    onMounted(() => {
      // 从cookie中读取数据
      myData.value = cookies.get('myData')
      console.log("mydata,",myData.value);
    })
    return {
      cookieForm,
      cookieInTest,
      cookieDeleteTest,
      myData
    }
  }
})
</script>
```
## 工具类
> util/index.js
```js
import Vue from 'vue'
import {Router as router} from "vue-router";
/**
 * 获取uuid
 * @returns {string} uuid
 */
export function getUUID() {
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, c => {
        return (c === 'x' ? (Math.random() * 16 | 0) : ('r&0x3' | '0x8')).toString(16)
    })
}
/**
 * 是否有权限
 * @param {*} key
 */
export function isAuth (key) {
    return JSON.parse(sessionStorage.getItem('permissions') || '[]').indexOf(key) !== -1 || false
}
/**
 * 树形数据转换
 * @param {*} data
 * @param {*} id
 * @param {*} pid
 */
export function treeDataTranslate (data, id = 'id', pid = 'parentId') {
    const res = [];
    const temp = {};
    for (let i = 0; i < data.length; i++) {
        temp[data[i][id]] = data[i]
    }
    for (let k = 0; k < data.length; k++) {
        if (temp[data[k][pid]] && data[k][id] !== data[k][pid]) {
            if (!temp[data[k][pid]]['children']) {
                temp[data[k][pid]]['children'] = []
            }
            if (!temp[data[k][pid]]['_level']) {
                temp[data[k][pid]]['_level'] = 1
            }
            data[k]['_level'] = temp[data[k][pid]]._level + 1
            temp[data[k][pid]]['children'].push(data[k])
        } else {
            res.push(data[k])
        }
    }
    return res
}
/**
 * 清除登录信息
 */
export function clearLoginInfo () {
    // Vue.cookie.delete('token')
    // store.commit('resetStore')
    // router.options.isAddDynamicMenuRoutes = false
}
```
## qs
```bash
npm install qs --save
```
### **qs的使用**
#### parse
>将`url`的参数转化为`object`对象
```js
const str = "name=%E9%A9%AC%E5%B0%8F%E8%B7%B3&age=12&address=%E5%BC%82%E6%AC%A1%E5%85%83'" 
console.log(qs.parse(str)) // Object {name: "马小跳",age: "12",address: "异次元"}
```
#### stringfy
>将`对象`转为`url参数`格式
```js
console.log(qs.stringify({
  name:'马小跳',
  age:12,
  address:'异次元'
}))
// logs name=%E9%A9%AC%E5%B0%8F%E8%B7%B3&age=12&address=%E5%BC%82%E6%AC%A1%E5%85%83
```
## echarts
> [echarts官网](https://echarts.apache.org/zh/index.html)
> 数据可视化
```bash
npm install echarts --save
```
> main.ts内引入
```js
// 全局引入相关包
import * as echarts from "echarts";
// 开启echarts
app.config.globalProperties.$echarts = echarts;
```
> 组件内使用
```vue
<template>
  <div id="myChart">
  </div>
</template>
<script lang="ts">
import {defineComponent, toRefs, reactive, onMounted} from 'vue'
import * as echarts from 'echarts'
export default defineComponent({
  name: 'EchartsTest',
  setup() {
    const state = reactive({
      option: {
        title: {
          text: 'Referer of a Website',
          subtext: 'Fake Data',
          left: 'center'
        },
        tooltip: {
          trigger: 'item'
        },
        legend: {
          orient: 'vertical',
          left: 'left'
        },
        grid: {
          top: '4%',
          left: '2%',
          right: '4%',
          bottom: '0%',
          containLabel: true,
        },
        series: [
          {
            name: 'Access From',
            type: 'pie',
            radius: '50%',
            data: [
              { value: 1048, name: 'Search Engine' },
              { value: 735, name: 'Direct' },
              { value: 580, name: 'Email' },
              { value: 484, name: 'Union Ads' },
              { value: 300, name: 'Video Ads' }
            ],
            emphasis: {
              itemStyle: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: 'rgba(0, 0, 0, 0.5)'
              }
            }
          }
        ]
      },
    })
    const initEcharts = () => {
      let myChart = echarts.init(document.getElementById('myChart'))
      // 绘制图表
      myChart.setOption(state.option)
    }
    onMounted(() => {
      initEcharts()
    })
    return {
      ...toRefs(state),
    }
  },
})
</script>
<style scoped>
#myChart{
  width: 700px;
  height: 700px;
}
</style>
```
## 其他
### 常用依赖
> [博客](https://blog.csdn.net/weixin_41897680/article/details/107560873)
### 常用库
> [博客](https://blog.csdn.net/qq_44757034/article/details/128090797)
