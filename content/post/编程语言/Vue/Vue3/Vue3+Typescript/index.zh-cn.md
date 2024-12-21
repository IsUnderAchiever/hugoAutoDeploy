---
title: Vue3+Typescript
description: Vue3+Typescript
date: 2023-04-30
slug: Vue3+Typescript
image: 202412211946837.png
categories:
    - Vue
---

# vue3 & typescript学习
> [学习文档](https://24kcs.github.io/vue3_study)
>
> 我这个已经安装过Element-plus了
## 初识语法
### 类型注解
```vue
<template>
    <div>
        <h3>{{user.name}}</h3>
        <hr>
        <el-button type="info" icon="Message" circle @click="testGreeter"/>
    </div>
</template>
<script lang="ts">
    import {defineComponent, reactive} from 'vue';
    export default defineComponent({
        name: '',
        setup() {
            // 1.类型注解
            const greeter = (person: string) => {
                return 'Hello,' + person
            }
            const user = reactive({
                name: "张三",
                age: 25
            });
            function testGreeter() {
                user.name = greeter(user.name);
            }
            return {
                testGreeter,
                user
            }
        }
    })
</script>
```
### 接口
```vue
<template>
    <div>
        <el-input v-model="input" placeholder="Please input"/>
        <hr>
        <el-button type="info" icon="Message" circle @click="testGreeter"/>
    </div>
</template>
<script lang="ts">
    import {defineComponent, reactive, ref} from 'vue';
    // 2.接口
    interface UserInfo {
        firstName: string;
        lastName: string;
    }
    export default defineComponent({
        name: '',
        setup() {
            const greeter = (user: UserInfo) => {
                return 'Hello,' + user.firstName + '_' + user.lastName
            }
            const user = reactive({
                firstName: "南宫",
                lastName: "问天"
            });
            let input = ref();
            function testGreeter() {
                input.value = greeter(user);
            }
            return {
                testGreeter,
                input
            }
        }
    })
</script>
```
### 类
```vue
<template>
    <div>
        <el-input v-model="input" placeholder="Please input"/>
        <hr>
        <el-button type="info" icon="Message" circle @click="testGreeter"/>
    </div>
</template>
<script lang="ts">
    import {defineComponent, reactive, ref} from 'vue';
    // 3.类
    class User {
        fullName: string;
        firstName: string;
        lastName: string;
        constructor(firstName: string, lastName: string) {
            this.firstName = firstName;
            this.lastName = lastName;
            this.fullName = firstName + '_' + lastName;
        }
    }
    interface UserInfo {
        firstName: string;
        lastName: string;
    }
    export default defineComponent({
        name: '',
        setup() {
            const greeter = (user: UserInfo) => {
                return 'Hello,' + user.firstName + '_' + user.lastName
            }
            const obj = new User('南宫', '问天');
            const user = reactive(obj);
            let input = ref();
            function testGreeter() {
                input.value = greeter(user);
            }
            return {
                testGreeter,
                input
            }
        }
    })
</script>
```
## 常用语法
> [查看原文档](https://24kcs.github.io/vue3_study/chapter2/1_type.html#%E5%B8%83%E5%B0%94%E5%80%BC)
### 基础类型
#### 布尔值
```typescript
let isDone: boolean = false;
isDone = true;
// isDone = 2 // error
```
#### 数字
```typescript
let a1: number = 10 // 十进制
let a2: number = 0b1010  // 二进制
let a3: number = 0o12 // 八进制
let a4: number = 0xa // 十六进制
```
#### 字符串
```typescript
let name:string = 'tom'
name = 'jack'
// name = 12 // error
let age:number = 12
const info = `My name is ${name}, I am ${age} years old!`
```
#### undefined & null
```typescript
let u: undefined = undefined
let n: null = null
```
#### 数组
```typescript
let list1: number[] = [1, 2, 3]
```
**数组泛型**
```typescript
let list2: Array<number> = [1, 2, 3]
```
#### 元组 tuple
元组类型允许表示一个已知元素数量和类型的数组，`各元素的类型不必相同`。 比如，你可以定义一对值分别为 `string` 和 `number` 类型的元组。
```typescript
let t1: [string, number]
t1 = ['hello', 10] // OK
t1 = [10, 'hello'] // Error
```
#### 枚举
`enum` 类型是对 JavaScript 标准数据类型的一个补充。 使用枚举类型可以`为一组数值赋予友好的名字`。
```typescript
enum Color {
  Red,
  Green,
  Blue
}
// 枚举数值默认从0开始依次递增
// 根据特定的名称得到对应的枚举数值
let myColor: Color = Color.Green  // 0
console.log(myColor, Color.Red, Color.Blue)
// 输出:1 0 2
```
默认情况下，从 `0` 开始为元素编号。 你也可以手动的指定成员的数值。 例如，我们将上面的例子改成从 `1` 开始编号：
```typescript
enum Color {Red = 1, Green, Blue}
let c: Color = Color.Green
```
或者，全部都采用手动赋值：
```typescript
enum Color {Red = 1, Green = 2, Blue = 4}
let c: Color = Color.Green
```
枚举类型提供的一个便利是你可以由枚举的值得到它的名字。 例如，我们知道数值为 2，但是不确定它映射到 Color 里的哪个名字，我们可以查找相应的名字：
```typescript
enum Color {Red = 1, Green, Blue}
let colorName: string = Color[2]
console.log(colorName)  // 'Green'
```
有时候，我们会想要为那些在编程阶段还不清楚类型的变量指定一个类型。 这些值可能来自于动态的内容，比如来自用户输入或第三方代码库。 这种情况下，我们不希望类型检查器对这些值进行检查而是直接让它们通过编译阶段的检查。 那么我们可以使用 `any` 类型来标记这些变量：
```typescript
let notSure: any = 4
notSure = 'maybe a string'
notSure = false // 也可以是个 boolean
```
在对现有代码进行改写的时候，`any` 类型是十分有用的，它允许你在编译时可选择地包含或移除类型检查。并且当你只知道一部分数据的类型时，`any` 类型也是有用的。 比如，你有一个数组，它包含了不同的类型的数据：
```typescript
let list: any[] = [1, true, 'free']
list[1] = 100
```
#### void
某种程度上来说，`void` 类型像是与 `any` 类型相反，它`表示没有任何类型`。 当一个函数没有返回值时，你通常会见到其返回值类型是 `void`：
```typescript
/* 表示没有任何类型, 一般用来说明函数的返回值不能是undefined和null之外的值 */
function fn(): void {
  console.log('fn()')
  // return undefined
  // return null
  // return 1 // error
}
```
声明一个 `void` 类型的变量没有什么大用，因为你只能为它赋予 `undefined` 和 `null`：
```typescript
let unusable: void = undefined
```
#### object
`object` 表示非原始类型，也就是除 `number`，`string`，`boolean`之外的类型。
使用 `object` 类型，就可以更好的表示像 `Object.create` 这样的 `API`。例如：
```typescript
function fn2(obj:object):object {
  console.log('fn2()', obj)
  return {}
  // return undefined
  // return null
}
console.log(fn2(new String('abc')))
// console.log(fn2('abc') // error
console.log(fn2(String))
```
#### 联合类型
联合类型（Union Types）表示取值可以为多种类型中的一种
需求1: 定义一个一个函数得到一个数字或字符串值的字符串形式值
```typescript
function toString2(x: number | string) : string {
  return x.toString()
}
```
需求2: 定义一个一个函数得到一个数字或字符串值的长度
```typescript
function getLength(x: number | string) {
  // return x.length // error
  if (x.length) { // error
    return x.length
  } else {
    return x.toString().length
  }
}
```
#### 类型断言
通过类型断言这种方式可以告诉编译器，“相信我，我知道自己在干什么”。 类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时的影响，只是在编译阶段起作用。 TypeScript 会假设你，程序员，已经进行了必须的检查。
类型断言有两种形式。 其一是“尖括号”语法, 另一个为 `as` 语法
```typescript
/* 
类型断言(Type Assertion): 可以用来手动指定一个值的类型
语法:
    方式一: <类型>值
    方式二: 值 as 类型  tsx中只能用这种方式
*/
/* 需求: 定义一个函数得到一个字符串或者数值数据的长度 */
function getLength(x: number | string) {
  if ((<string>x).length) {
    return (x as string).length
  } else {
    return x.toString().length
  }
}
console.log(getLength('abcd'), getLength(1234))
```
#### 类型推断
类型推断: TS会在没有明确的指定类型的时候推测出一个类型
有下面2种情况: 1. 定义变量时赋值了, 推断为对应的类型. 2. 定义变量时没有赋值, 推断为any类型
```typescript
/* 定义变量时赋值了, 推断为对应的类型 */
let b9 = 123 // number
// b9 = 'abc' // error
/* 定义变量时没有赋值, 推断为any类型 */
let b10  // any类型
b10 = 123
b10 = 'abc'
```
### 接口
#### 简单用法
```vue
<template>
  <div>
    <span>次数:{{num}}</span><br>
    <el-button @click="test">点击</el-button>
  </div>
</template>
<script lang="ts">
import {defineComponent, reactive, ref} from 'vue';
interface UserForm{
  // 只读属性
  readonly name:string,
  age:number
}
interface Alarm{
  alert(): any;
}
interface Light {
  lightOn(): void;
  lightOff(): void;
}
class Car implements Alarm {
  alert() {
    console.log('Car alert');
  }
}
export default defineComponent({
  name: '',
  setup() {
    let num=ref<number>(0);
    const test=()=>{
      num.value++;
      const car=new Car()
      car.alert();
    }
    return {
      test,
      num
    }
  }
})
</script>
<style scoped>
</style>
```
#### 类实现接口
```typescript
/* 
类类型: 实现接口
1. 一个类可以实现多个接口
2. 一个接口可以继承多个接口
*/
interface Alarm {
  alert(): any;
}
interface Light {
  lightOn(): void;
  lightOff(): void;
}
class Car implements Alarm {
  alert() {
      console.log('Car alert');
  }
}
```
#### 一个类实现多个接口
```typescript
class Car2 implements Alarm, Light {
  alert() {
    console.log('Car alert');
  }
  lightOn() {
    console.log('Car light on');
  }
  lightOff() {
    console.log('Car light off');
  }
}
```
#### 接口继承接口
```typescript
interface LightableAlarm extends Alarm, Light {
}
```
### 类
#### 修饰符
当成员被标记成 `private` 时，它就不能在声明它的类的外部访问
`protected` 修饰符与 `private` 修饰符的行为很相似，但有一点不同，`protected`成员在派生类中仍然可以访问
```typescript
/* 
访问修饰符: 用来描述类内部的属性/方法的可访问性
  public: 默认值, 公开的外部也可以访问
  private: 只能类内部可以访问
  protected: 类内部和子类可以访问
*/
class Animal {
  public name: string
  public constructor (name: string) {
    this.name = name
  }
  public run (distance: number=0) {
    console.log(`${this.name} run ${distance}m`)
  }
}
class Person extends Animal {
  private age: number = 18
  protected sex: string = '男'
  run (distance: number=5) {
    console.log('Person jumping...')
    super.run(distance)
  }
}
class Student extends Person {
  run (distance: number=6) {
    console.log('Student jumping...')
    console.log(this.sex) // 子类能看到父类中受保护的成员
    // console.log(this.age) //  子类看不到父类中私有的成员
    super.run(distance)
  }
}
console.log(new Person('abc').name) // 公开的可见
// console.log(new Person('abc').sex) // 受保护的不可见
// console.log(new Person('abc').age) //  私有的不可见
```
#### 静态属性
```typescript
/* 
静态属性, 是类对象的属性
非静态属性, 是类的实例对象的属性
*/
class Person {
  name1: string = 'A'
  static name2: string = 'B'
}
console.log(Person.name2)
console.log(new Person().name1)
```
#### 抽象类
```typescript
/* 
抽象类
  不能创建实例对象, 只有实现类才能创建实例
  可以包含未实现的抽象方法
*/
abstract class Animal {
  abstract cry ()
  run () {
    console.log('run()')
  }
}
class Dog extends Animal {
  cry () {
    console.log(' Dog cry()')
  }
}
const dog = new Dog()
dog.cry()
dog.run()
```
### 函数
#### 函数类型
```typescript
function add(x: number, y: number): number {
  return x + y
}
let myAdd = function(x: number, y: number): number { 
  return x + y
}
```
#### 可选参数和默认参数
> JavaScript 里，每个参数都是可选的，可传可不传。 没传参的时候，它的值就是 `undefined`。
>  在TypeScript 里我们可以在参数名旁使用 `?` 实现可选参数的功能。 比如，我们想让 `lastName` 是可选的
```typescript
function buildName(firstName: string='A', lastName?: string): string {
  if (lastName) {
    return firstName + '-' + lastName
  } else {
    return firstName
  }
}
console.log(buildName('C', 'D'))
console.log(buildName('C'))
console.log(buildName())
```
#### 函数重载
>`typeof`判断参数的类型
```typescript
/* 
函数重载: 函数名相同, 而形参不同的多个函数
需求: 我们有一个add函数，它可以接收2个string类型的参数进行拼接，也可以接收2个number类型的参数进行相加 
*/
// 重载函数声明
function add (x: string, y: string): string
function add (x: number, y: number): number
// 定义函数实现
function add(x: string | number, y: string | number): string | number {
  // 在实现上我们要注意严格判断两个参数的类型是否相等，而不能简单的写一个 x + y
  if (typeof x === 'string' && typeof y === 'string') {
    return x + y
  } else if (typeof x === 'number' && typeof y === 'number') {
    return x + y
  }
}
console.log(add(1, 2))
console.log(add('a', 'b'))
// console.log(add(1, 'a')) // error
```
### 泛型
> 基本使用
```typescript
function createArray(value: any, count: number): any[] {
  const arr: any[] = []
  for (let index = 0; index < count; index++) {
    arr.push(value)
  }
  return arr
}
const arr1 = createArray(11, 3)
const arr2 = createArray('aa', 3)
console.log(arr1[0].toFixed(), arr2[0].split(''))
```
#### 函数泛型
```typescript
function createArray2 <T> (value: T, count: number) {
  const arr: Array<T> = []
  for (let index = 0; index < count; index++) {
    arr.push(value)
  }
  return arr
}
const arr3 = createArray2<number>(11, 3)
console.log(arr3[0].toFixed())
// console.log(arr3[0].split('')) // error
const arr4 = createArray2<string>('aa', 3)
console.log(arr4[0].split(''))
// console.log(arr4[0].toFixed()) // error
```
#### 多个泛型参数的函数
```typescript
function swap <K, V> (a: K, b: V): [K, V] {
  return [a, b]
}
const result = swap<string, number>('abc', 123)
console.log(result[0].length, result[1].toFixed())
```
#### 泛型接口
```typescript
interface IbaseCRUD <T> {
  data: T[]
  add: (t: T) => void
  getById: (id: number) => T
}
class User {
  id?: number; //id主键自增
  name: string; //姓名
  age: number; //年龄
  constructor (name, age) {
    this.name = name
    this.age = age
  }
}
class UserCRUD implements IbaseCRUD <User> {
  data: User[] = []
  
  add(user: User): void {
    user = {...user, id: Date.now()}
    this.data.push(user)
    console.log('保存user', user.id)
  }
  getById(id: number): User {
    return this.data.find(item => item.id===id)
  }
}
const userCRUD = new UserCRUD()
userCRUD.add(new User('tom', 12))
userCRUD.add(new User('tom2', 13))
console.log(userCRUD.data)
```
#### 泛型类
```typescript
class GenericNumber<T> {
  zeroValue: T
  add: (x: T, y: T) => T
}
let myGenericNumber = new GenericNumber<number>()
myGenericNumber.zeroValue = 0
myGenericNumber.add = function(x, y) {
  return x + y 
}
let myGenericString = new GenericNumber<string>()
myGenericString.zeroValue = 'abc'
myGenericString.add = function(x, y) { 
  return x + y
}
console.log(myGenericString.add(myGenericString.zeroValue, 'test'))
console.log(myGenericNumber.add(myGenericNumber.zeroValue, 12))
```
#### 泛型约束
如果我们直接对一个泛型参数取 `length` 属性, 会报错, 因为这个泛型根本就不知道它有这个属性
```typescript
// 没有泛型约束
function fn <T>(x: T): void {
  // console.log(x.length)  // error
}
```
我们可以使用泛型约束来实现
```typescript
interface Lengthwise {
  length: number;
}
// 指定泛型约束
function fn2 <T extends Lengthwise>(x: T): void {
  console.log(x.length)
}
```
我们需要传入符合约束类型的值，必须包含必须 `length` 属性：
```typescript
fn2('abc')
// fn2(123) // error  number没有length属性
```
## vuex
```bash
npm install vuex --save-dev
```
> store/index.ts
```js
import { createStore } from 'vuex'
export default createStore({
  state: {
  },
  getters: {
  },
  mutations: {
  },
  actions: {
  },
  modules: {
  }
})
```
> main.ts
```js
import App from './App.vue'
const app = createApp(App)
app.use(store)
app.mount('#app')
```
> 在vue3中，vuex提供了一个`useStore`方法来获取全局的store
> 没有什么比直接看[官网](https://vuex.vuejs.org/zh/)更清楚了
### 访问 State 和 Getter
>为了访问 state 和 getter，需要创建 `computed` 引用以保留响应性，这与在选项式 API 中创建计算属性等效。
```js
import {createStore} from 'vuex'
export default createStore({
    state: {
        count:0,
    },
    getters: {
        double(state){
            return state.count;
        }
    },
    mutations: {
    },
    actions: {
    },
    modules: {
    },
    plugins:[
    ]
})
```
```vue
<template>
  <div>
    <el-button type="info" icon="Message" circle @click="testGreeter"/>
  </div>
</template>
<script lang="ts">
import {defineComponent, reactive, ref, computed} from 'vue';
import {useStore} from "vuex";
export default defineComponent({
  setup() {
    const store = useStore();
    let count = computed(() => {
      return store.state.count;
    })
    let double = computed(() => {
      return store.getters.double;
    })
    function testGreeter() {
      console.log("count的值:",count.value);
      console.log("double的值:",double.value);
    }
    return {
      testGreeter,
    }
  }
})
</script>
```
> 官网采用以下写法
```vue
<template>
  <div>
  </div>
</template>
<script lang="ts">
import {defineComponent, reactive, ref, computed} from 'vue';
import {useStore} from "vuex";
export default defineComponent({
  setup() {
    const store = useStore();
    return {
      count: computed(() => store.state.count),
      double: computed(() => store.getters.double)
    }
  }
})
</script>
```
### 访问 Mutation 和 Action
> 要使用 mutation 和 action 时，只需要在 `setup` 钩子函数中调用 `commit` 和 `dispatch` 函数
>
> [参考博客](https://blog.csdn.net/qq_45934504/article/details/123462736)
```js
import {createStore} from 'vuex'
import createPersistedstate  from 'vuex-persistedstate'
import user from './modules/user'
export default createStore({
    state: {
        count:0,
    },
    getters: {
    },
    mutations: {
        sum (state, num) {
            state.count += num
        }
    },
    actions: {
    },
    modules: {
    },
    plugins:[
    ]
})
```
```vue
<template>
  <div>
    <el-button type="info" icon="Message" circle @click="testVuex(10)"/>
  </div>
</template>
<script lang="ts">
import {defineComponent, reactive, ref, computed} from 'vue';
import {useStore} from "vuex";
export default defineComponent({
  setup() {
    const store = useStore();
    const testVuex=(num)=>{
      // 使用 store.commit('mution中函数名', '需要传递的参数' ) 在commit里添加参数的方式进行传递
      store.commit('sum', num)
      console.log(store.state.count);
    }
    return {
      testVuex,
    }
  }
})
</script>
```
> 也可以采用以下方式
```js
import {createStore} from 'vuex'
import createPersistedstate  from 'vuex-persistedstate'
import user from './modules/user'
export default createStore({
    state: {
        count:0,
    },
    getters: {
    },
    mutations: {
        sum (state, payload) {
            state.count += payload.num
        }
    },
    actions: {
    },
    modules: {
    },
    plugins:[
    ]
})
```
```vue
<template>
  <div>
    <el-button type="info" icon="Message" circle @click="testVuex(10)"/>
  </div>
</template>
<script lang="ts">
import {defineComponent, reactive, ref, computed} from 'vue';
import {useStore} from "vuex";
export default defineComponent({
  setup() {
    const store = useStore();
    const testVuex=(num)=>{
      // 前面提到了 mution 主要包含 type 和 回调函数 两部分, 和通过commit payload的方式进行参数传递（提交）,下面我们可以用这种方式进行 mution 的提交
      store.commit({
        type: 'sum',  // 类型就是mution中定义的方法名称
        num
      })
      console.log(store.state.count);
    }
    return {
      testVuex,
    }
  }
})
</script>
```
**`actions`**
```js
import {createStore} from 'vuex'
import createPersistedstate  from 'vuex-persistedstate'
import user from './modules/user'
export default createStore({
    state: {
        count:0,
    },
    getters: {
    },
    mutations: {
        sum (state, num) {
            state.count += num
        }
    },
    actions: {
        // context 上下文对象，可以理解为store
        sum_actions (context, num) {
            setTimeout(() => {
                context.commit('sum', num)  // 通过context去触发mutions中的sum
            }, 1000)
        }
    },
    modules: {
    },
    plugins:[
    ]
})
```
```vue
<template>
  <div>
    <el-button type="info" icon="Message" circle @click="testVuex(10)"/>
  </div>
</template>
<script lang="ts">
import {defineComponent, reactive, ref, computed} from 'vue';
import {useStore} from "vuex";
export default defineComponent({
  setup() {
    const store = useStore();
    const testVuex=(num)=>{
      store.dispatch('sum_actions', num)
      console.log(store.state.count);
    }
    return {
      testVuex,
    }
  }
})
</script>
```
> **通过 promise 实现异步操作完成，通知组件异步执行成功或是失败。**
```js
import {createStore} from 'vuex'
import createPersistedstate from 'vuex-persistedstate'
import user from './modules/user'
export default createStore({
    state: {
        count: 0,
    },
    getters: {
    },
    mutations: {
        sum(state, num) {
            state.count += num
        }
    },
    actions: {
        sum_actions(context, payload) {
            return new Promise((resolve, reject) => {
                setTimeout(() => {
                    // 通过 context 上下文对象拿到 count
                    if (context.state.count < 30) {
                        context.commit('sum', payload.num)
                        resolve('异步操作执行成功')
                    } else {
                        reject(new Error('异步操作执行错误'))
                    }
                }, 1000)
            })
        }
    },
    modules: {},
    plugins: [
    ]
})
```
```vue
<template>
  <div>
    <el-button type="info" icon="Message" circle @click="testVuex(10)"/>
  </div>
</template>
<script lang="ts">
import {defineComponent, reactive, ref, computed} from 'vue';
import {useStore} from "vuex";
export default defineComponent({
  setup() {
    const store = useStore();
    const testVuex=(num)=>{
      store.dispatch('sum_actions', {
        num
      }).then((res) => {
        console.log("success:",res)
      }).catch((err) => {
        console.log(err)
      })
    }
    return {
      testVuex,
    }
  }
})
</script>
```
**`getters`**
```js
import {createStore} from 'vuex'
import createPersistedstate from 'vuex-persistedstate'
import user from './modules/user'
export default createStore({
    state: {
        users: [{
            name: "小美",
            age: 12
        }, {
            name: "小红",
            age: 22
        }, {
            name: "小明",
            age: 23
        }]
    },
    getters: {
        filterUsersByAge(state) {
            return state.users.filter(item => item.age >= 20)
        }
    },
    mutations: {},
    actions: {},
    modules: {},
    plugins: [
    ]
})
```
```vue
<template>
  <div>
    <el-button type="info" icon="Message" circle @click="testVuex(10)"/>
    <br>
    <ul>
      <li>{{ store.getters.filterUsersByAge }}</li>
    </ul>
    <hr>
    <ul>
      <li v-for="item in store.getters.filterUsersByAge">{{ item }}</li>
    </ul>
  </div>
</template>
<script lang="ts">
import {defineComponent, reactive, ref, computed} from 'vue';
import {useStore} from "vuex";
export default defineComponent({
  setup() {
    const store = useStore();
    const testVuex = (num) => {
    }
    return {
      testVuex,
      store
    }
  }
})
</script>
```
**modules**
> user.js
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
