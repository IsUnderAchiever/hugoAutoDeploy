---
title: 创建Vue项目
description: 创建Vue项目
date: 2023-03-22
slug: 创建Vue项目
image: 202412211946837.png
categories:
    - Vue
---

# 创建Vue项目
## Vue脚手架的安装与卸载
>安装、卸载vue-cli2命令
```bash
npm install vue-cli -g
npm uninstall vue-cli -g
```
>安装、卸载vue-cli3命令
```bash
npm install @vue/cli -g
npm uninstall @vue/cli -g
```
## Vue项目创建
```bash
# 创建一个名为demo-vue的项目
vue create demo-vue
```
![image-20230126171311650](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301261713790.png)
![image-20230126171655053](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301261716128.png)
## Element-UI
```JavaScript
# 安装
# npm i element-ui -S
vue add element
# 引入Element
import Vue from 'vue';
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
import App from './App.vue';
Vue.use(ElementUI);
new Vue({
  el: '#app',
  render: h => h(App)
});
```
## axios
```JavaScript
npm install axios -s
// 配置request.js-------------------------
import axios from 'axios'
const request = axios.create({
	// baseURL: '/api',  // 注意！！ 这里是全局统一加上了 '/api' 前缀，也就是说所有接口都会加上'/api'前缀在，页面里面写接口的时候就不要加 '/api'了，否则会出现2个'/api'，类似 '/api/api/user'这样的报错，切记！！！
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
// 配置vue.config.js------------------------------------
// 跨域配置
module.exports = {
    devServer: {                //记住，别写错了devServer//设置本地默认端口  选填
        port: 9876,
        proxy: {                 //设置代理，必须填
            '/api': {              //设置拦截器  拦截器格式   斜杠+拦截器名字，名字可以自己定
                target: 'http://localhost:9999',     //代理的目标地址
                changeOrigin: true,              //是否设置同源，输入是的
                pathRewrite: {                   //路径重写
                    '^/api': ''                     //选择忽略拦截器里面的内容
                }
            }
        }
    }
}
```
## qs
```JavaScript
npm install qs
// 在main.js中配置
import qs from 'qs'
Vue.prototype.qs=qs
```
## vue-cookie
```
npm install vue-cookie --save
// main.js
```
## lodash
```
npm i -save lodash // 全局安装
import cloneDeep from 'lodash/cloneDeep'
```
### mockjs
```
npm install mockjs -D
```
## vuex-persistedstate
[参考博客1](https://blog.csdn.net/xm1037782843/article/details/128071142)
[参考博客2](https://blog.csdn.net/xm1037782843/article/details/128071142)
```
npm i vuex-persistedstate --save
```
