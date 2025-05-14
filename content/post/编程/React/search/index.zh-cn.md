---
title: React
description: React
date: 2024-07-07
slug: React
image: 202412212156937.png
categories:
    - React
---
# React

> 参考文档
>
> [React基础](https://www.yuque.com/fechaichai/qeamqf/xbai87)
>
> [React&Mobx](https://www.yuque.com/fechaichai/qeamqf/apomum)

## 基础

使用脚手架创建项目

```sh
npx create-react-app react-basic

cd react-basic

yarn start
```

删除`src`下除了`App.js`和`index.js`之外的其他文件

![image-20240630212110955](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412212048722.png)

> index.js

```js
import React from 'react';
import ReactDOM from 'react-dom/client';
// 引入根组件App
import App from './App';

// 通过调用ReactDOM的render方法渲染App根组件到id为root的dom节点上
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

> App.js

```js
function App() {
  return (
    <div className="App">
      <header className="App-header">
      </header>
    </div>
  );
}

export default App;
```

### 什么是JSX

> JSX是 JavaScript XML（HTML）的缩写，表示在 JS 代码中书写 HTML 结构

需要注意的是JSX 并不是标准的 JS 语法，是 JS 的语法扩展，浏览器默认是不识别的

### JSX中使用js表达式

```js
function App() {
  const name = "张三";
  return <div className="App">我的名字是{name}</div>;
}

export default App;
```

注意:`if/变量声明语句`是语句，并非表达式，不能出现在`{}`中

### JSX列表渲染

```jsx
function App() {
  const nameList = ["张三", "李四", "王五"];
  return (
    <div className="App">
      <ul>
        {nameList.map((item) => (
          <li>{item}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

> 我们一步步来分析，为什么这么写
>
> 我们希望重复`li`标签

```jsx
function App() {
  const nameList = ["张三", "李四", "王五"];
  return (
    <div className="App">
      <ul>
        <li>张三</li>
        <li>李四</li>
        <li>王五</li>
      </ul>
    </div>
  );
}

export default App;
```

> 我们希望`return li标签`

```jsx
function App() {
  const nameList = ["张三", "李四", "王五"];
  return (
    <div className="App">
      <ul>
        {nameList.map(item=><li>张三</li>)}
      </ul>
    </div>
  );
}

export default App;
```

最后将文本内容替换成列表即可

对象列表也是同理

```jsx
function App() {
  const userList = [
    {
      name: "张三",
      age: 12,
    },
    {
      name: "李四",
      age: 13,
    },
    {
      name: "王五",
      age: 14,
    },
  ];
  return (
    <div className="App">
      <ul>
        {userList.map((item) => (
          <li>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

> 在渲染的节点上添加`key`属性可提高diff性能

    <div className="App">
      <ul>
        {userList.map((item) => (
          <li key={item.name}>{item.name}</li>
        ))}
      </ul>
    </div>

### JSX条件渲染

> 实现条件渲染有两种方式，`三元运算`和`逻辑运算符(&&)`

```jsx
function App() {
  const flag = true;
  return (
    <div className="App">
      <span>{flag ? "yes" : "no"}</span>
      {flag && <span>显示</span>}
    </div>
  );
}

export default App;
```

### JSX样式处理

> 样式处理主要有以下几种写法

**行内样式**

```jsx
function App() {
  return (
    <div className="App">
      <span style={{ color: "red" }}>React从入门到入土</span>
    </div>
  );
}

export default App;
```

为啥这里要`套两层{}`？

其实这种写法中内部的`{}`表示`{ color: "red" }`是一个对象，效果和下面一样

```jsx
function App() {
  const fontColor={
    color: "red"
  }
  return (
    <div className="App">
      <span style={fontColor}>React从入门到入土</span>
    </div>
  );
}

export default App;
```

> className

新建一个`App.css`

```css
.font-red {
  color: red;
}
.font-green {
  color: green;
}
```

这里可以使用动态类名来控制样式

```jsx
import './App.css'

function App() {
  const flag = true;
  return (
    <div className="App">
      <span className={flag ? "font-red" : "font-green"}>
        React从入门到入土
      </span>
    </div>
  );
}

export default App;
```

### JSX注意事项

1. JSX必须有一个根节点，如果没有根节点，可以使用`<></>`（幽灵节点）替代
2. 所有标签必须形成闭合，成对闭合或者自闭合都可以
3. JSX中的语法更加贴近JS语法，属性名采用驼峰命名法  `class -> className`  `for -> htmlFor`
4. JSX支持多行（换行），如果需要换行，需使用`()` 包裹，防止bug出现

### 阶段练习

[项目demo](https://gitee.com/react-course-series/react-jsx-demo )

完成 `评论数据渲染` 、 `tab内容渲染`  、`评论列表点赞和点踩`的功能

```jsx
import './index.css'
import avatar from './images/avatar.png'
// 依赖的数据
const state = {
    // hot: 热度排序  time: 时间排序
    tabs: [
        {
            id: 1,
            name: '热度',
            type: 'hot'
        },
        {
            id: 2,
            name: '时间',
            type: 'time'
        }
    ],
    active: 'hot',
    list: [
        {
            id: 1,
            author: '刘德华',
            comment: '给我一杯忘情水',
            time: new Date('2021-10-10 09:09:00'),
            // 1: 点赞 0：无态度 -1:踩
            attitude: 1
        },
        {
            id: 2,
            author: '周杰伦',
            comment: '哎哟，不错哦',
            time: new Date('2021-10-11 09:09:00'),
            // 1: 点赞 0：无态度 -1:踩
            attitude: 0
        },
        {
            id: 3,
            author: '五月天',
            comment: '不打扰，是我的温柔',
            time: new Date('2021-10-11 10:09:00'),
            // 1: 点赞 0：无态度 -1:踩
            attitude: -1
        }
    ]
}

function ShowComment() {
    return <>
        {state.list.map(item => <div className="list-item" key={item.id}>
            <div className="user-face">
                <img className="user-head" src={avatar} alt=""/>
            </div>
            <div className="comment">
                <div className="user">{item.author}</div>
                <p className="text">{item.comment}</p>
                <div className="info">
                    <span className="time">
                        {item.time.toLocaleString()}
                    </span>
                    <span className={`like ${item.attitude === 1 ? 'liked' : ''}`}>
                        <i className="icon"/>
                    </span>
                    <span className={`hate ${item.attitude === -1 ? 'hated' : ''}`}>
                        <i className="icon"/>
                    </span>
                    <span className="reply btn-hover">删除</span>
                </div>
            </div>
        </div>)}
    </>
}

function App() {
    return (
        <div className="App">
            <div className="comment-container">
                {/* 评论数 */}
                <div className="comment-head">
                    <span>{state.list.length} 评论</span>
                </div>
                {/* 排序 */}
                <div className="tabs-order">
                    <ul className="sort-container">
                        {state.tabs.map(item => <li key={item.id}
                                                    className={item.type === state.active ? 'on' : ''}>按{item.name}排序</li>)}
                    </ul>
                </div>

                {/* 添加评论 */}
                <div className="comment-send">
                    <div className="user-face">
                        <img className="user-head" src={avatar} alt=""/>
                    </div>
                    <div className="textarea-container">
            <textarea
                cols="80"
                rows="5"
                placeholder="发条友善的评论"
                className="ipt-txt"
            />
                        <button className="comment-submit">发表评论</button>
                    </div>
                    <div className="comment-emoji">
                        <i className="face"></i>
                        <span className="text">表情</span>
                    </div>
                </div>

                {/* 评论列表 */}
                <div className="comment-list">
                    <ShowComment/>
                </div>
            </div>
        </div>
    )
}

export default App
```



## 组件

能够独立使用函数完成react组件的创建和渲染

### 函数组件

```jsx
// 定义函数组件
function SayHello () {
  return <span>Hello</span>
}

function App () {

  return (
    <div className="App">
      {/* 渲染函数组件 */}
      <SayHello />
    </div>
  )
}

export default App
```

**约定说明**

1. 组件的名称**必须首字母大写**，react内部会根据这个来判断是组件还是普通的HTML标签
2. 函数组件**必须有返回值**，表示该组件的 UI 结构；如果不需要渲染任何内容，则返回 null
3. 组件就像 HTML 标签一样可以被渲染到页面中。组件表示的是一段结构内容，对于函数组件来说，渲染的内容是函数的**返回值**就是对应的内容
4. 使用函数名称作为组件标签名称，可以成对出现也可以自闭合

### 类组件

```jsx
import React from "react"

class SayHello extends React.Component {
  render () {
    return <span>Hello</span>
  }
}

function App () {

  return (
    <div className="App">
      {/* 渲染函数组件 */}
      <SayHello />
    </div>
  )
}

export default App
```

**约定说明**

1. **类名称也必须以大写字母开头**
2. 类组件应该继承 React.Component 父类，从而使用父类中提供的方法或属性
3. 类组件必须提供 render 方法**render 方法必须有返回值，表示该组件的 UI 结构**

### 函数组件事件绑定

新建`components/Home.js`，以后再`Home`中编辑内容，就不直接更改App.js了

![image-20240630221126521](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412212049244.png)

```jsx
import Home from "./components/Home"

function App () {

  return (
    <div className="App">
      <Home />
    </div>
  )
}

export default App
```

```jsx
function Home () {
  // 定义事件
  const clickLog = () => {
    console.log('点击')
  }
  // 绑定事件
  return <button onClick={clickLog}>点击</button>
}

export default Home
```

> 获取事件对象

```jsx
function Home () {
  // 定义事件
  const clickLog = (e) => {
    console.log('点击:', e)
  }
  // 绑定事件
  return <button onClick={clickLog}>点击</button>
}

export default Home
```

> 传递额外参数
>
> 如果及需要获取事件对象，又需要传递参数怎么办

解决思路: 改造事件绑定为箭头函数 在箭头函数中完成参数的传递

```jsx
function Home () {
  // 定义事件
  const clickLog = (e, flag) => {
    if (flag) {
      console.log('点击:', e)
    }
  }
  // 绑定事件
  return <button onClick={(e) => clickLog(e, false)}>点击</button>
}

export default Home
```

### 类组件的事件绑定

类组件中的事件绑定，整体的方式和函数组件差别不大

唯一需要注意的 因为处于class类语境下 所以定义事件回调函数以及定它写法上有不同

1. 定义的时候: class Fields语法  

2. 使用的时候: 需要借助this关键词获取

```jsx
import React from "react"

class Home extends React.Component {
  // class Fields
  clickLog = (e) => {
    console.log(e)
  }
  render () {
    return <button onClick={this.clickLog}>点击</button>
  }
}

export default Home
```

> 注意这里的写法

```jsx
  clickLog = (e) => {
    console.log(e)
  }
```

你可能会写成

```
  clickLog (e) {
    console.log(e)
  }
```

这样可能会引发this的指向问题

### 组件状态

> 一个前提：在React hook出来之前，函数式组件是没有自己的状态的，所以我们统一通过类组件来讲解

1. 初始化状态
2. 视图读取初始状态
3. 修改状态
4. 视图使用新状态自动更新

#### 初始化状态

-  通过class的实例属性state来初始化 
-  state的值是一个对象结构，表示一个组件可以有多个数据状态 

```jsx
import React from "react"

class Home extends React.Component {
  state = {
    count: 1
  }
  render () {
    return <button>点击</button>
  }
}

export default Home
```

#### 读取状态

-  通过this.state来获取状态 

```jsx
import React from "react"

class Home extends React.Component {
  state = {
    count: 1
  }
  render () {
    return <button>{this.state.count}</button>
  }
}

export default Home
```

#### 修改状态

-  语法
  `this.setState({ 要修改的部分数据 })` 
-  setState方法作用 

1. 1. 修改state中的数据状态
   2. 更新UI

-  思想
    	数据驱动视图，也就是只要修改数据状态，那么页面就会自动刷新，无需手动操作dom 
-  注意事项
    	**不要直接修改state中的值，必须通过setState方法进行修改** 

```jsx
import React from "react"

class Home extends React.Component {
  state = {
    count: 1
  }
  addCount = () => {
    this.setState({
      count: this.state.count + 1
    })
  }
  render () {
    return <>
      <span>{this.state.count}</span>
      <button onClick={this.addCount}>+</button>
    </>
  }
}

export default Home
```

### this指向问题

```jsx
import React from "react"

class Home extends React.Component {
  constructor() {
    super()
    // 使用bind修正clickLog1中this的指向问题
    // 相当于在类组件初始化阶段来修正回调函数的this指向
    // 永远指向当前组件实例对象
    this.clickLog = this.clickLog.bind(this)
  }
  clickLog () {
    // 此时打印为 undefined，那么修改state数据就会报错
    console.log(this)
  }

  render () {
    // 这里的this指的是当前组件的实例对象
    console.log('render:', this)
    return <>
      <button onClick={this.clickLog}>点击</button>
    </>
  }
}

export default Home
```

> 推介写法

```jsx
import React from "react"

class Home extends React.Component {
  clickLog = () => {
    console.log(this)
  }

  render () {
    return <>
      <button onClick={this.clickLog}>点击1</button>
      <button onClick={() => this.clickLog()}>点击2</button>
      <button onClick={() => this.clickLog(true)}>点击3</button>
      <button onClick={(e) => this.clickLog(e, true)}>点击4</button>
    </>
  }
}

export default Home
```

### React的状态不可变

> 不要直接修改状态的值，而是基于当前状态创建新的状态值

**`错误的写法`**

```jsx
state = {
  count : 0,
  list: [1,2,3],
  person: {
     name:'jack',
     age:18
  }
}
// 直接修改简单类型Number
this.state.count++
++this.state.count
this.state.count += 1
this.state.count = 1

// 直接修改数组
this.state.list.push(123)
this.state.list.spice(1,1)

// 直接修改对象
this.state.person.name = 'rose'
```

**`正确的写法`**

```jsx
this.setState({
    count: this.state.count + 1
    list: [...this.state.list, 4],
    person: {
       ...this.state.person,
       // 覆盖原来的属性 就可以达到修改对象中属性的目的
       name: 'rose'
    }
})
```

### 表单处理

使用React处理表单元素，一般有两种方式：

1. 受控组件 （推荐使用）
2. 非受控组件 （了解）

#### 受控组件

> 什么是受控组件？  `input框自己的状态被React组件状态控制`
> React组件的状态的地方是在state中，input表单元素也有自己的状态是在value中
> React将state与表单元素的值（value）绑定到一起，由state的值来控制表单元素的值，从而保证单一数据源特性

**实现步骤**

1. 在组件的state中声明一个组件的状态数据
2. 将状态数据设置为input标签元素的value属性的值
3. 为input添加change事件，在事件处理程序中，通过事件对象e获取到当前文本框的值（`即用户当前输入的值`）
4. 调用setState方法，将文本框的值作为state状态的最新值

```jsx
import React from "react"

class Home extends React.Component {
  state = {
    text: ''
  }
  changeText = (e) => {
    this.setState({
      // 通过事件对象e来获取用户当前输入的值
      text: e.target.value
    })
  }
  render () {
    return <>
      <input type="text" value={this.state.text} onChange={this.changeText} />
    </>
  }
}

export default Home
```

#### 非受控表单组件

>非受控组件就是通过手动操作dom的方式获取文本框的值，文本框的状态不受react组件的state中的状态控制，直接通过原生dom获取输入框的值

**实现步骤**

1. 导入`createRef` 函数
2. 调用createRef函数，创建一个ref对象，存储到名为`msgRef`的实例属性中
3. 为input添加ref属性，值为`msgRef`
4. 在按钮的事件处理程序中，通过`msgRef.current`即可拿到input对应的dom元素，而其中`msgRef.current.value`拿到的就是文本框的值

```jsx
import React, { createRef } from "react"

class Home extends React.Component {
  msgRef = createRef()
  changeText = () => {
    console.log('value:', this.msgRef.current.value)
  }
  render () {
    return <>
      <input type="text" ref={this.msgRef} />
      <button onClick={this.changeText}>点击</button>
    </>
  }
}

export default Home
```

### 阶段小练习

[项目demo](https://gitee.com/react-course-series/react-component-demo )

1. 完成tab点击切换激活状态交互 
2. 完成发表评论功能
3. 完成删除评论功能 

```jsx
import './index.css'
import avatar from './images/avatar.png'
import React from 'react'

// 时间格式化
function formatDate(time) {
    return `${time.getFullYear()}-${time.getMonth()}-${time.getDate()}`
}

class App extends React.Component {
    state = {
        author: '战斗暴龙兽',
        // hot: 热度排序  time: 时间排序
        tabs: [
            {
                id: 1,
                name: '热度',
                type: 'hot'
            },
            {
                id: 2,
                name: '时间',
                type: 'time'
            }
        ],
        active: 'hot',
        list: [
            {
                id: 1,
                author: '刘德华',
                comment: '给我一杯忘情水',
                time: new Date('2021-10-10 09:09:00'),
                // 1: 点赞 0：无态度 -1:踩
                attitude: 1
            },
            {
                id: 2,
                author: '周杰伦',
                comment: '哎哟，不错哦',
                time: new Date('2021-10-11 09:09:00'),
                // 1: 点赞 0：无态度 -1:踩
                attitude: 0
            },
            {
                id: 3,
                author: '五月天',
                comment: '不打扰，是我的温柔',
                time: new Date('2021-10-11 10:09:00'),
                // 1: 点赞 0：无态度 -1:踩
                attitude: -1
            }
        ],
        comment: ''
    }
    // 切换tab页
    changeTab = (type) => {
        this.setState({
            active: type
        })
    }
    // 输入评论
    changeComment = (e) => {
        this.setState({
            comment: e.target.value
        })
    }
    // 发表评论
    addComment = (e) => {
        this.setState({
            // 插入评论列表
            list: [
                ...this.state.list,
                {
                    id: this.state.list.length + 1,
                    author: this.state.author,
                    comment: e.target.value,
                    time: new Date(),
                    attitude: 0
                }
            ],
            // 清空评论
            comment: ''
        })
    }
    // 删除评论
    deleteComment = (comment) => {
        this.setState({
            // 插入评论列表
            list: this.state.list.filter(item => {
                // 判断删除的评论是否是自己发布
                // 是则删除
                if (comment.author === this.state.author) {
                    return item.id !== comment.id
                } else {
                    // 否则不删除
                    return item
                }
            })
        })
    }

    render() {
        return (
            <div className="App">
                <div className="comment-container">
                    {/* 评论数 */}
                    <div className="comment-head">
                        <span>{this.state.list.length} 评论</span>
                    </div>
                    {/* 排序 */}
                    <div className="tabs-order">
                        <ul className="sort-container">
                            {
                                this.state.tabs.map(tab => (
                                    <li
                                        key={tab.id}
                                        className={tab.type === this.state.active ? 'on' : ''}
                                        onClick={() => this.changeTab(tab.type)}
                                    >按{tab.name}排序</li>
                                ))
                            }
                        </ul>
                    </div>

                    {/* 添加评论 */}
                    <div className="comment-send">
                        <div className="user-face">
                            <img className="user-head" src={avatar} alt=""/>
                        </div>
                        <div className="textarea-container">
              <textarea
                  cols="80"
                  rows="5"
                  placeholder="发条友善的评论"
                  className="ipt-txt"
                  value={this.state.comment}
                  onChange={this.changeComment}
              />
                            <button className="comment-submit" onClick={this.addComment}>发表评论</button>
                        </div>
                        <div className="comment-emoji">
                            <i className="face"></i>
                            <span className="text">表情</span>
                        </div>
                    </div>

                    {/* 评论列表 */}
                    <div className="comment-list">
                        {
                            this.state.list.map(item => (
                                <div className="list-item" key={item.id}>
                                    <div className="user-face">
                                        <img className="user-head" src={avatar} alt=""/>
                                    </div>
                                    <div className="comment">
                                        <div className="user">{item.author}</div>
                                        <p className="text">{item.comment}</p>
                                        <div className="info">
                                            <span className="time">{formatDate(item.time)}</span>
                                            <span className={item.attitude === 1 ? 'like liked' : 'like'}>
                        <i className="icon"/>
                      </span>
                                            <span className={item.attitude === -1 ? 'hate hated' : 'hate'}>
                        <i className="icon"/>
                      </span>
                                            <span className="reply btn-hover"
                                                  onClick={() => this.deleteComment(item)}>删除</span>
                                        </div>
                                    </div>
                                </div>
                            ))
                        }
                    </div>
                </div>
            </div>)
    }
}


export default App
```



## 组件通信

为了能让各组件之间可以进行互相沟通，数据传递，这个过程就是组件通信

1. 父子关系 -  **最重要的**
2. 兄弟关系 -  自定义事件模式产生技术方法 eventBus  /  通过共同的父组件通信
3. 其它关系 -  **mobx / redux / zustand**

### 父传子

**实现步骤**

1.  父组件提供要传递的数据  -  `state` 
2.  给子组件标签`添加属性`值为 state中的数据 
3.  子组件中通过 `props` 接收父组件中传过来的数据 
    - 类组件使用this.props获取props对象
    - 函数式组件直接通过参数获取props对象


```jsx
import React from "react"

// 子组件1(函数式子组件)
function ChildrenComponentFunction (props) {
  return <>
    <div>{props.msg}</div>
  </>
}

// 子组件2(类子组件)
class ChildrenComponentClass extends React.Component {
  render () {
    return <>
      <div>
        {this.props.msg}
      </div>
    </>
  }
}


// 父组件
class Home extends React.Component {
  state = {
    message: '你好'
  }
  render () {
    return <>
      <ChildrenComponentFunction msg={this.state.message} />
      <ChildrenComponentClass msg={this.state.message} />
    </>
  }
}

export default Home
```

### props说明

**1.  props是只读对象（readonly）**

根据单项数据流的要求，子组件只能读取props中的数据，不能进行修改

**2. props可以传递任意数据**

数字、字符串、布尔值、数组、对象、`函数、JSX`

### 子传父

**口诀：** `父组件给子组件传递回调函数，子组件调用`

**实现步骤**

1. 父组件提供一个回调函数 - 用于接收数据
2. 将函数作为属性的值，传给子组件
3. 子组件通过props调用 回调函数
4. 将子组件中的数据作为参数传递给回调函数

```jsx
import React from "react"

// 子组件1(函数式子组件)
function ChildrenComponentFunction (props) {
  const chickFuncComponent = () => {
    props.changeMsg('函数式子组件')
  }
  return <>
    <div onClick={chickFuncComponent}>{props.msg}</div>
  </>
}

// 子组件2(类子组件)
class ChildrenComponentClass extends React.Component {
  chickClassComponent = () => {
    this.props.changeMsg('类子组件')
  }
  render () {
    return <>
      <div onClick={this.chickClassComponent}>
        {this.props.msg}
      </div>
    </>
  }
}


// 父组件
class Home extends React.Component {
  state = {
    message: '你好'
  }
  // 提供回调函数
  changeMsg = (newMsg) => {
    this.setState({
      message: newMsg
    })
  }
  render () {
    return <>
      <ChildrenComponentFunction msg={this.state.message} changeMsg={this.changeMsg} />
      <ChildrenComponentClass msg={this.state.message} changeMsg={this.changeMsg} />
    </>
  }
}

export default Home
```

### 兄弟组件传参

> 思路: `SonB`->`Father`->`SonA`

**实现步骤**

1. 将共享状态提升到最近的公共父组件中，由公共父组件管理这个状态 

   - 提供共享状态
   - 提供操作共享状态的方法
2. 要接收数据状态的子组件通过 props 接收数据
3. 要传递数据状态的子组件通过props接收方法，调用方法传递数据

**子组件1**和**子组件2**互为兄弟组件

```jsx
import React from "react"

// 子组件1
function ChildrenComponentFunction1 ({ msg, changeMsg }) {
  const sendMessage = () => {
    changeMsg(msg + '==')
  }
  return <>
    <div onClick={sendMessage}>组件1</div>
  </>
}

// 子组件2
function ChildrenComponentFunction2 ({ msg }) {
  return <>
    <div>组件2</div><span>{msg}</span>
  </>
}


// 父组件
class Home extends React.Component {
  state = {
    message: '初始值'
  }
  changeMsg = (msg) => {
    this.setState({
      message: msg
    })
  }
  render () {
    return <>
      <ChildrenComponentFunction1 msg={this.state.message} changeMsg={this.changeMsg} />
      <ChildrenComponentFunction2 msg={this.state.message} />
    </>
  }
}

export default Home
```



### 跨组件传参

> 如果采用一层一层通过props往下传递就会很繁琐，有时候自己都不知道接收到的props里有什么，所以不推介这种做法

**实现步骤**

1. 创建Context对象 导出 Provider 和 Consumer对象 

   ```jsx
   const { Provider, Consumer } = createContext()
   ```

   

2. 使用Provider包裹上层组件提供数据 

   ```jsx
   <Provider value={this.state.message}>
       {/* 根组件 */}
   </Provider>
   ```

   

3. 需要用到数据的组件使用Consumer包裹获取数据 

   ```jsx
   <Consumer >
       {value => /* 基于 context 值进行渲染*/}
   </Consumer>
   ```


```jsx
import React, { createContext } from "react"

const { Provider, Consumer } = createContext()

// 子组件1
function ChildrenComponentFunction1 () {

  return <>
    <div>
      <ChildrenComponentFunction2 />
    </div>
  </>
}

// 子组件2
function ChildrenComponentFunction2 () {
  return <>
    <div>
      <ChildrenComponentFunction3 />
    </div>
  </>
}

// 子组件3
function ChildrenComponentFunction3 () {
  return <>
    <Consumer>
      {value => <div>组件3{value}</div>}
    </Consumer>
  </>
}


// 父组件
class Home extends React.Component {
  state = {
    message: '初始值'
  }
  changeMsg = (msg) => {
    this.setState({
      message: msg
    })
  }
  render () {
    return (
      <div>
        <Provider value={this.state.message}>
          <ChildrenComponentFunction1 />
        </Provider>
      </div>
    )
  }
}

export default Home
```

> 期间遇到过一个问题，错误写法如下

```jsx
// 子组件3
function ChildrenComponentFunction3 () {
  return <>
    <Consumer>
      <div>组件3</div>{value => <div>{value}</div>}
    </Consumer>
  </>
}
```

> 实际上，`Consumer`期望一个函数作为子元素，此函数接收上下文值作为参数并返回一个React节点。



### 阶段小练习

要求：App为父组件用来提供列表数据 ，ListItem为子组件用来渲染列表数据

![组件通信练习](https://cdn.nlark.com/yuque/0/2022/png/274425/1654490603983-9e535a08-84ca-4a16-850b-570586801ea3.png?x-oss-process=image%2Fformat%2Cwebp)

```jsx
import React from "react"

// 子组件
function ChildrenComponentFunction ({ messageList, deleteMsg }) {
  const deleteParentMsg = (id) => {
    deleteMsg(id)
  }
  return <>
    {messageList.map(item => <div key={item.id}>{item.message}<button onClick={() => deleteParentMsg(item.id)}>删除</button></div>)}
  </>
}


// 父组件
class Home extends React.Component {
  state = {
    messageList: [
      {
        id: 1,
        message: '信息1'
      },
      {
        id: 2,
        message: '信息2'
      },
      {
        id: 3,
        message: '信息3'
      }
    ]
  }
  deleteMsg = (id) => {
    this.setState({
      messageList: this.state.messageList.filter(item => item.id !== id)
    })
  }
  render () {
    return (
      <div>
        <ChildrenComponentFunction messageList={this.state.messageList} deleteMsg={this.deleteMsg} />
      </div>
    )
  }
}

export default Home
```

## Hooks基础

> 上面使用的主要是类组件来实现数据状态存储以及更新，接下来我们将改为函数式组件
>
> Hooks的本质：**一套能够使函数组件更强大，更灵活的“钩子”**

### useState

> useState为函数组件提供状态（state）

**使用步骤**

1. 导入 `useState` 函数
2. 调用 `useState` 函数，并传入状态的初始值
3. 从`useState`函数的返回值中，拿到状态和修改状态的方法
4. 在JSX中展示状态
5. 调用修改状态的方法更新状态

```jsx
import React, { useState } from "react"

function Home () {
  // 参数：状态初始值比如,传入 0 表示该状态的初始值为 0
  // 返回值：数组,包含两个值：1 状态值（state） 2 修改该状态的函数（setState）
  const [count, updateCount] = useState(0)
  return <>
    <div>
      <button onClick={() => updateCount(count - 1)}>-</button>
      <span> {count}</span>
      <button onClick={() => updateCount(count + 1)}>+</button>
    </div >
  </>
}

export default Home
```

#### 状态的读取和修改

**读取状态**

该方式提供的状态，是函数内部的局部变量，可以在函数内的任意位置使用

**修改状态**

1. `updateCount`是一个函数，参数表示`最新的状态值`
2. 调用该函数后，将使用新值替换旧值
3. 修改状态后，由于状态发生变化，会引起视图变化

#### 组件更新的过程

函数组件使用 **useState** hook 后的执行过程，以及状态值的变化



- 组件第一次渲染 

1. 1. 从头开始执行该组件中的代码逻辑
   2. 调用 `useState(0)` 将传入的参数作为状态初始值，即：0
   3. 渲染组件，此时，获取到的状态 count 值为： 0

- 组件第二次渲染 

1. 1. 点击按钮，调用 `updateCount(count + 1)` 修改状态，因为状态发生改变，所以，该组件会重新渲染
   2. 组件重新渲染时，会再次执行该组件中的代码逻辑
   3. 再次调用 `useState(0)`，此时 **React 内部会拿到最新的状态值而非初始值**，比如，该案例中最新的状态值为 1
   4. 再次渲染组件，此时，获取到的状态 count 值为：1

注意：**useState 的初始值(参数)只会在组件第一次渲染时生效**。也就是说，以后的每次渲染，useState 获取到都是最新的状态值，React 组件会记住每次最新的状态值

#### 使用规则

1. `useState` 函数可以执行多次，每次执行互相独立，每调用一次为函数组件提供一个状态 
2. 注意事项
   - 只能出现在函数组件或者其他hook函数中 
   - 不能嵌套在if/for/其它函数中（react按照hooks的调用顺序识别每一个hook） 
   - 可以通过开发者工具查看hooks状态 

> 错误写法

```jsx
let num = 1
function List(){
  num++
  if(num / 2 === 0){
     const [name, setName] = useState('cp') 
  }
  const [list,setList] = useState([])
}
// 俩个hook的顺序不是固定的，这是不可以的！！！
```

### useEffect

**什么是副作用**

> 副作用是相对于主作用来说的，一个函数除了主作用，其他的作用就是副作用。对于 React 组件来说，**主作用就是根据数据（state/props）渲染 UI**，除此之外都是副作用（比如，手动修改 DOM）

**常见的副作用**

1. 数据请求 ajax发送
2. 手动修改dom
3. localStorage操作

useEffect函数的作用就是为react函数组件提供副作用处理的！

> `useEffect`为react函数组件提供副作用处理

**使用步骤**

1. 导入 `useEffect` 函数
2. 调用 `useEffect` 函数，并传入回调函数
3. 在回调函数中编写副作用处理（dom操作）
4. 修改数据状态
5. 检测副作用是否生效

```jsx
import React, { useState, useEffect } from "react"

function Home () {
  const [count, updateCount] = useState(0)
  useEffect(() => {
    // 渲染dom
    document.title = count
  })
  return <>
    <div onClick={() => updateCount(count + 1)}>
      已经点击了{count}次
    </div >
  </>
}

export default Home
```

**不添加依赖项**

> 组件首次渲染执行一次，以及不管是哪个状态更改引起组件更新时都会重新执行

1. 组件初始渲染
2. useState里的数据发生更新

`useEffect`是基于React组件的渲染生命周期来工作的。当组件的props或state发生变化，触发了重新渲染，之后`useEffect`会根据其依赖数组来决定是否执行

**添加空数组**

> 组件只在首次渲染时执行一次

```jsx
import React, { useState, useEffect } from "react"

function Home () {
  const [count, updateCount] = useState(0)
  useEffect(() => {
    // 渲染dom
    document.title = count
  }, [])
  return <>
    <div onClick={() => updateCount(count + 1)}>
      已经点击了{count}次
    </div >
  </>
}

export default Home
```

**添加特定依赖项**

```jsx
import React, { useState, useEffect } from "react"

function Home () {
  const [count, updateCount] = useState(0)
  const [name, setName] = useState('张三')
  useEffect(() => {
    console.log('重新执行')
  }, [count])
  return <>
    <button onClick={() => updateCount(count + 1)}>
      已经点击了{count}次
    </button >
    <button onClick={() => setName(name + '==')}>
      名字:{name}
    </button >
  </>
}

export default Home
```

#### 清理副作用

如果想要清理副作用 可以在副作用函数中的末尾return一个新的函数，在新的函数中编写清理副作用的逻辑

注意执行时机为：

1. 组件卸载时自动执行
2. 组件更新时，下一个useEffect副作用函数执行之前自动执行

```jsx
import React, { useState, useEffect } from "react"

function Home () {
  const [count, setCount] = useState(0)
  useEffect(() => {
    const timerId = setInterval(() => {
      setCount(count + 1)
    }, 1000)
    return () => {
      // 用来清理副作用的事情
      clearInterval(timerId)
    }
  }, [count])
  return (
    <div>
      {count}
    </div>
  )
}

export default Home
```



### 阶段小练习

**需求描述：** 自定义hook函数，可以自动同步到本地LocalStorage

```jsx
function useStorage (key, value) {
  const [message, setMessage] = useState(value)
  useEffect(() => {
    localStorage.setItem(key, message)
  }, [key, message])
  return [message, setMessage]
}
```

## Hook进阶

### useState

> 回调函数的参数

**使用场景**

参数只会在组件的初始渲染中起作用，后续渲染时会被忽略。如果初始 state 需要通过计算才能获得，则可以传入一个函数，在函数中计算并返回初始的 state，此函数只在初始渲染时被调用

**语法**

```jsx
const [name, setName] = useState(()=>{   
  // 编写计算逻辑    return '计算之后的初始值'
})
```

**语法规则**

1. 回调函数return出去的值将作为 `name` 的初始值
2. 回调函数中的逻辑只会在组件初始化的时候执行一次

**语法选择**

1. 如果就是初始化一个普通的数据 直接使用 `useState(普通数据)` 即可
2. 如果要初始化的数据无法直接得到需要通过计算才能获取到，使用`useState(()=>{})`

```jsx
import React, { useState, useEffect } from "react"

function ChildrenComponent (props) {
  const [count, setCount] = useState(() => {
    // 经过计算
    return props.count * 2 + 9
  })
  return <>
    <div>{count}</div>
  </>
}

function Home () {
  return (
    <>
      <ChildrenComponent count={2} />
    </>
  )
}

export default Home
```

### useEffect

> 发送网络请求

**使用场景**

如何在useEffect中发送网络请求，并且封装同步 async await操作

**语法要求**

不可以直接在useEffect的回调函数外层直接包裹 await ，因为**异步会导致清理函数无法立即返回**

**`错误写法`**

```jsx
useEffect(async ()=>{    
    const res = await axios.get('http://geek.itheima.net/v1_0/channels')   
    console.log(res)
},[])
```

**`正确写法`**

在内部单独定义一个函数，然后把这个函数包装成同步

需要配置`json格式的数据`，这里未展示

```jsx
useEffect(()=>{   
    async function fetchData(){
       const res = await axios.get('http://localhost:10020/user/list')
       console.log(res)   
    } 
    fetchData()
},[])
```

或者

```jsx
  useEffect(() => {
    async function getUserList() {
      const res = await fetch('http://localhost:10020/user/list')
      console.log('userList:', await res.json())
    }
    getUserList()
  }, [])
```

### useRef

**使用场景**

在函数组件中获取真实的dom元素对象或者是组件对象

**使用步骤**

1. 导入 `useRef` 函数
2. 执行 `useRef` 函数并传入null，返回值为一个对象 内部有一个current属性存放拿到的dom对象（组件实例）
3. 通过ref 绑定 要获取的元素或者组件

**获取dom**

```jsx
'use client'
import { log } from 'console'
import React, { useEffect, useRef } from 'react'

function Home() {
  const btn = useRef(null)
  useEffect(() => {
    setInterval(() => {
      console.log(btn.current)
    }, 1000)
  }, [])

  return (
    <button
      type="button"
      ref={btn}>
      Default
    </button>
  )
}

export default Home
```

这里和前面的非受控组件非常类似

**获取组件实例**

函数组件由于没有实例，不能使用ref获取，如果想获取组件实例，必须是类组件

```jsx
'use client'
import React, { useEffect, useRef } from 'react'

class HelloComponent extends React.Component {
  render() {
    return (
      <button
        type="button">
        Default
      </button>
    )
  }
}

function Home() {
  const btn = useRef(null)
  useEffect(() => {
    setInterval(() => {
      console.log(btn.current)
    }, 1000)
  }, [])

  return <HelloComponent ref={btn} />
}

export default Home
```

### useContext

**实现步骤**

1. 使用`createContext` 创建Context对象
2. 在顶层组件通过`Provider` 提供数据
3. 在底层组件通过`useContext`函数获取数据



```jsx
'use client'
import React, { createContext, useContext } from 'react'

const Context = createContext('')

// 子组件1
function ChildComponent1() {
  return <ChildComponent2 />
}

// 子组件2
function ChildComponent2() {
  const msg = useContext(Context)
  return (
    <>
      <button
        type="button"
        className="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800">
        {msg}
      </button>
    </>
  )
}

// 父组件
function Home() {
  return (
    <Context.Provider value="这是传递的信息">
      <ChildComponent1 />
    </Context.Provider>
  )
}

export default Home
```

效果等同于

```jsx
'use client'
import React, { createContext } from 'react'

const Context = createContext('')

// 子组件1
function ChildComponent1() {
  return <ChildComponent2 />
}

// 子组件2
function ChildComponent2() {
  return (
    <Context.Consumer>
      {(value) => (
        <>
          <button
            type="button">
            {value}
          </button>
        </>
      )}
    </Context.Consumer>
  )
}

// 父组件
function Home() {
  return (
    <Context.Provider value="这是传递的信息">
      <ChildComponent1 />
    </Context.Provider>
  )
}

export default Home
```

## ReactRouter

### 环境配置

```sh
# 创建react项目
yarn create vite react-router --template react

# 安装所有依赖包
yarn

# 启动项目
yarn dev

# 安装react-router包
yarn add react-router-dom@6
```

### 基本使用

需求:  准备俩个按钮，点击不同按钮切换不同组件内容的显示

实现步骤：

1. 导入必要的路由router内置组件
2. 准备俩个React组件
3. 按照路由的规则进行路由配置

> 这里以`Home`和`About`为例

```jsx
// 引入必要的内置组件
import { BrowserRouter, Routes, Route, Link, Router } from 'react-router-dom'
import Home from "./components/Home"
import { About } from './components/About'

function App () {

  return (
    <div className="App">
      {/* 配置路由规则 */}
      <BrowserRouter>
        <Link to={'/home'} >首页</Link>
        <Link to={'/about'} >关于</Link>
        <Routes>
          {/* 首页路由 */}
          <Route path='/home' element={<Home />} />
          {/* 关于页路由 */}
          <Route path='/about' element={<About />} />
        </Routes>
      </BrowserRouter>
    </div>
  )
}

export default App
```

### 核心内置组件

#### BrowserRouter

作用: 包裹整个应用，一个React应用只需要使用一次

**模式**

1. HashRouter
   - 实现方式:`监听url hash值实现`
   - 路由url表现:`http://localhost:3000/#/about`
2. BrowserRouter
   - 实现方式:`h5的 history.pushState API实现`
   - 路由url表现:`http://localhost:3000/about`

#### Link

作用: 用于指定导航链接，完成声明式的路由跳转  类似于 `<router-link/>`

```jsx
<Link to={'/home'} >首页</Link>
```

这里to属性用于指定路由地址，表示要跳转到哪里去，Link组件最终会被渲染为原生的a链接

#### Routes

作用: 提供一个路由出口，组件内部会存在多个内置的Route组件，满足条件的路由会被渲染到组件内部，类比  `router-view`

#### Route

作用: 用于定义路由路径和渲染组件的对应关系  [element：因为react体系内 把组件叫做react element]

```jsx
    <Routes>
      {/* 首页路由 */}
      <Route path='/home' element={<Home />} />
      {/* 关于页路由 */}
      <Route path='/about' element={<About />} />
    </Routes>
```

其中path属性用来指定匹配的路径地址，element属性指定要渲染的组件，图中配置的意思为: 当url上访问的地址为 /about 时，当前路由发生匹配，对应的About组件渲染

### 编程式导航

概念:  通过js编程的方式进行路由页面跳转，比如说从首页跳转到关于页

实现步骤：

1. 导入一个 useNavigate 钩子函数
2. 执行 useNavigate 函数 得到 跳转函数
3. 在事件中执行跳转函数完成路由跳转

```jsx
import { useNavigate } from "react-router-dom"

function Home () {
  const navigate = useNavigate()
  return (
    <div>
      <button onClick={() => navigate('/about')}>跳转About页面</button>
    </div>
  )
}

export default Home
```

注: 如果在跳转时不想添加历史记录，可以添加额外参数replace 为true

```jsx
navigate('/about', { replace: true }
```

### 路由传参

场景：跳转路由的同时，有时候要需要传递参数  

#### searchParams传参

> 就是问号传参

```jsx
import { useNavigate } from "react-router-dom"

function Home () {
  const navigate = useNavigate()
  return (
    <div>
      <button onClick={() => navigate('/about?name=张三')}>跳转About页面</button>
    </div>
  )
}

export default Home
```

**获取参数**

```jsx
import { useEffect } from "react"
import { useSearchParams } from "react-router-dom"

export function About () {
  let [searchParams] = useSearchParams()
  useEffect(() => {
    const name = searchParams.get('name')
    console.log(name)
  }, [])
  return (
    <div>
      这里是About页面
    </div>
  )
}
```

#### params传参

> 就是路径传参

首先需要修改如下配置，否则`/about/1001`匹配不上`/about`，会报错`No routes matched location "/about/1001"`

```jsx
<Route path='/about/:id' element={<About />} />
```

```jsx
import { useNavigate } from "react-router-dom"

function Home () {
  const navigate = useNavigate()
  return (
    <div>
      <button onClick={() => navigate('/about/1001')}>跳转About页面</button>
    </div>
  )
}

export default Home
```

**获取参数**

```jsx
import { useEffect } from "react"
import { useParams } from "react-router-dom"

export function About () {
  let params = useParams()
  useEffect(() => {
    console.log(params.id)
  }, [])
  return (
    <div>
      这里是About页面
    </div>
  )
}
```

### 嵌套路由

嵌套路由非常常用，在我们做的很多的管理后台系统中，通常我们都会设计一个Layout组件，在它内部实现嵌套路由

实现步骤：

1. App.js中定义嵌套路由声明
2. Layout组件内部通过 <Outlet/> 指定二级路由出口

![嵌套路由](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412212048104.webp)

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import Home from "./components/Home"
import './App.css'
import LeftComponent from './components/Left'
import ContentComponent from './components/Content'

function App () {

  return (
    <div className="App" >
      {/* 配置路由规则 */}
      <BrowserRouter>
        <Routes>
          {/* 首页路由 */}
          <Route path='/home' element={<Home />} >
            <Route path='left' element={<LeftComponent />} />
            <Route path='content' element={<ContentComponent />} />
          </Route>
        </Routes>
      </BrowserRouter>
    </div>
  )
}

export default App
```

> 配置路由出口

```jsx
import { Link, Outlet } from "react-router-dom"

const bgColor = {
  height: '100vh',
  width: '100%',
  backgroundColor: 'grey'
}

function Home () {
  return (
    <div style={bgColor}>
      <Link to='/home/left' >左侧</Link>
      <Link to='/home/content' >正文</Link>
      <Outlet />
    </div >
  )
}

export default Home
```

### 默认二级路由

场景: 应用首次渲染完毕就需要显示的二级路由

实现步骤:

1. 给默认二级路由标记index属性
2. 把原本的路径path属性去掉

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import Home from "./components/Home"
import './App.css'
import LeftComponent from './components/Left'
import ContentComponent from './components/Content'

function App () {

  return (
    <div className="App" >
      {/* 配置路由规则 */}
      <BrowserRouter>
        <Routes>
          {/* 首页路由 */}
          <Route path='/home' element={<Home />} >
            <Route path='left' element={<LeftComponent />} />
            <Route index element={<ContentComponent />} />
          </Route>
        </Routes>
      </BrowserRouter>
    </div>
  )
}

export default App
```

```jsx
import { Link, Outlet } from "react-router-dom"

const bgColor = {
  height: '100vh',
  width: '100%',
  backgroundColor: 'grey'
}

function Home () {
  return (
    <div style={bgColor}>
      <Link to='/home/left' >左侧</Link>
      <Link to='/home/content' >正文</Link>
      <Outlet />
    </div >
  )
}

export default Home
```

### 404路由配置

场景：当url的路径在整个路由配置中都找不到对应的path，使用404兜底组件进行渲染

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import Home from "./components/Home"
import './App.css'
import LeftComponent from './components/Left'
import ContentComponent from './components/Content'
import NotFoundComponent from './components/NotFound'

function App () {

  return (
    <div className="App" >
      {/* 配置路由规则 */}
      <BrowserRouter>
        <Routes>
          {/* 首页路由 */}
          <Route path='/home' element={<Home />} >
            <Route path='left' element={<LeftComponent />} />
            <Route index element={<ContentComponent />} />
          </Route>
          {/* 兜底路由 404 */}
          <Route path='*' element={<NotFoundComponent />} />
        </Routes>
      </BrowserRouter>
    </div>
  )
}

export default App
```

```jsx
function NotFoundComponent () {
  return (
    <>
      <div>
        资源已经走丢了......
      </div>
    </>
  )
}

export default NotFoundComponent
```

### 集中式路由配置

场景: 当我们需要路由权限控制点时候, 对路由数组做一些权限的筛选过滤，所谓的集中式路由配置就是用一个数组统一把所有的路由对应关系写好替换 本来的Roues组件

```jsx
import { BrowserRouter, Routes, Route, useRoutes } from 'react-router-dom'
import Home from "./components/Home"
import './App.css'
import LeftComponent from './components/Left'
import ContentComponent from './components/Content'
import NotFoundComponent from './components/NotFound'

// 假设这是服务器响应的路由信息
const routeList = [{
  path: '/home',
  element: <Home />,
  children: [
    {
      element: <ContentComponent />,
      // index设置为true 变成默认的二级路由
      index: true,
    },
    {
      path: 'left',
      element: <LeftComponent />,
    },
  ],
},
// 增加n个路由对应关系
{
  path: '*',
  element: <NotFoundComponent />,
}]

// 2. 使用useRoutes方法传入routesList生成Routes组件
function WrapperRoutes () {
  let element = useRoutes(routeList)
  return element
}

function App () {

  return (
    <div className="App" >
      {/* 配置路由规则 */}
      <BrowserRouter>
        {/* 替换之前的Routes组件 */}
        <WrapperRoutes />
      </BrowserRouter>
    </div>
  )
}

export default App
```

### 阶段小练习

实现一个`layout`布局页面，这里我就不在乎什么样式了

**App.css**

```css
* {
  margin: 0;
  padding: none;
}

.App {
  height: 100vh;
  width: 100%;
}

.left {
  height: 100vh;
  width: 20%;
  /* background-color: green; */
  float: left;
}

.content {
  height: 100vh;
  width: 80%;
  /* background-color: red; */
  float: left;
}
```

**App.js**

```jsx
import { BrowserRouter, Routes, Route, useRoutes } from 'react-router-dom'
import Home from "./components/Home"
import './App.css'
import LeftComponent from './components/Left'
import ContentComponent from './components/Content'
import NotFoundComponent from './components/NotFound'

// 假设这是服务器响应的路由信息
const routeList = [{
  path: '/home',
  element: <Home />,
  children: [
    {
      path: 'content/:id',
      element: <ContentComponent />,
      // index设置为true 变成默认的二级路由
      index: true,
    },
    {
      path: 'left',
      element: <LeftComponent />,
    },
  ],
},
// 增加n个路由对应关系
{
  path: '*',
  element: <NotFoundComponent />,
}]

// 2. 使用useRoutes方法传入routesList生成Routes组件
function WrapperRoutes () {
  let element = useRoutes(routeList)
  return element
}

function App () {

  return (
    <div className="App" >
      {/* 配置路由规则 */}
      <BrowserRouter>
        {/* 替换之前的Routes组件 */}
        <WrapperRoutes />
      </BrowserRouter>
    </div>
  )
}

export default App
```

**Left.js**

```jsx
import { useNavigate } from "react-router-dom"

function LeftComponent () {
  const navigate = useNavigate()
  return (
    <>
      <button onClick={() => navigate('/home/content/1')}>跳转1</button>
      <button onClick={() => navigate('/home/content/2')}>跳转2</button>
    </>
  )
}

export default LeftComponent
```

**Home.js**

```jsx
import LeftComponent from './Left'
import ContentComponent from './Content'

function Home () {
  return (
    <>
      <div className='left' >
        <LeftComponent />
      </div >
      <div className='content'>
        <ContentComponent />
      </div>
    </>
  )
}

export default Home
```

**Content**

```jsx
import { useParams } from "react-router-dom"

function ContentComponent () {
  const params = useParams()
  return (
    <>
      这些是正文{params.id}
    </>
  )
}

export default ContentComponent
```

在React中，当有多个`input`元素时，可以通过创建一个通用的事件处理器来提高代码的复用性和可维护性。这种做法通常涉及到使用事件对象的`target`属性，该属性包含了触发事件的元素以及其相关的信息，如元素的`name`属性和当前的`value`。

下面是一个示例代码，展示了如何使用一个共享的`handleChange`函数来处理多个`input`元素的状态更新：

```tsx
import React, { useState } from 'react';

function LoginForm() {
  const [formState, setFormState] = useState({ username: '', password: '' });

  const handleChange = (event) => {
    const { name, value } = event.target;
    setFormState({
      ...formState,
      [name]: value,
    });
  };

  return (
    <form>
      <div>
        <label htmlFor="username">Username:</label>
        <input
          type="text"
          id="username"
          name="username"
          value={formState.username}
          onChange={handleChange}
        />
      </div>
      <div>
        <label htmlFor="password">Password:</label>
        <input
          type="password"
          id="password"
          name="password"
          value={formState.password}
          onChange={handleChange}
        />
      </div>
      {/* 其他表单元素和提交按钮 */}
    </form>
  );
}

export default LoginForm;
```

在这个例子中，`formState`是一个对象，包含了`username`和`password`的值。`handleChange`函数接收`event`参数，从中提取`name`和`value`属性，然后使用解构赋值和计算属性名来更新`formState`。

每个`input`元素都有一个`name`属性，该属性的值与`formState`对象中的键相对应。同时，每个`input`元素的`value`属性被设置为`formState`中相应的值，以保持组件受控状态。`onChange`属性则指向`handleChange`函数，确保每次输入变化时都会调用此函数并更新状态。

这种方法不仅提高了代码的复用性，还使得添加更多的`input`字段变得更加容易，因为只需要增加`input`标签并指定`name`属性即可。
