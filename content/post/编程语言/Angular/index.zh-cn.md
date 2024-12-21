---
title: Angular
description: Angular
date: 2024-12-08
slug: Angular
image: 202412211941020.png
categories:
    - Angular
---
# Angular

## 基本要点

### 基础

> [官方文档](https://angular.cn/overview)
>
> [中文文档](https://www.angularc.cn/introduce.html)
>
> 安装脚手架

```sh
npm install -g @angular/cli
```

> 查看命令

```sh
ng --help

Commands:
  ng add <collection>            Adds support for an external library to your project.
  ng analytics                   Configures the gathering of Angular CLI usage metrics.
  ng build [project]             Compiles an Angular application or library into an output directory named dist/ at the
                                 given output path.                                                         [aliases: b]
  ng cache                       Configure persistent disk cache and retrieve cache statistics.
  ng completion                  Set up Angular CLI autocompletion for your terminal.
  ng config [json-path] [value]  Retrieves or sets Angular configuration values in the angular.json file for the
                                 workspace.
  ng deploy [project]            Invokes the deploy builder for a specified project or for the default project in the
                                 workspace.
  ng e2e [project]               Builds and serves an Angular application, then runs end-to-end tests.      [aliases: e]
  ng extract-i18n [project]      Extracts i18n messages from source code.
  ng generate                    Generates and/or modifies files based on a schematic.                      [aliases: g]
  ng lint [project]              Runs linting tools on Angular application code in a given project folder.
  ng new [name]                  Creates a new Angular workspace.                                           [aliases: n]
  ng run <target>                Runs an Architect target with an optional custom builder configuration defined in your
                                 project.
  ng serve [project]             Builds and serves your application, rebuilding on file changes.       [aliases: dev, s]
  ng test [project]              Runs unit tests in a project.                                              [aliases: t]
  ng update [packages..]         Updates your workspace and its dependencies. See https://update.angular.dev/.
  ng version                     Outputs Angular CLI version.                                               [aliases: v]
Options:
  --help     Shows a help message for this command in the console.                                             [boolean]
  --version  Show Angular CLI version.                                                                         [boolean]

For more information, see https://angular.dev/cli/.
```

> 创建项目

```sh
ng new angular-learning
```

> 运行项目

```sh
npm run start
```

> [访问页面](http://localhost:4200/)
>
> `http://localhost:4200/`

### 项目结构解析

接下来介绍一下项目的结构

`app.routes.ts`是路由配置文件

`app.component.html`是模板文件

`app.component.scss`是css文件

`app.component.spec.ts`是`app.component.ts`的单元测试文件

`app.component.ts`是当前app组建的核心js文件

有些项目可能会有`app.module.ts`文件，是app组件的入口文件

没有`app.module.ts`文件是因为Angular CLI默认创建的是一个独立组件的应用程序，而不是传统的基于模块的应用程序

在独立组件模式下，组件直接在其类中定义模板和样式，不需要显式地在模块中声明它们

`app.config.server.ts` 文件主要用于配置 Angular 应用程序的服务器端渲染(SSR)

```typescript
import { mergeApplicationConfig, ApplicationConfig } from '@angular/core';
import { provideServerRendering } from '@angular/platform-server';
import { appConfig } from './app.config';

const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering()
  ]
};

export const config = mergeApplicationConfig(appConfig, serverConfig);
```

以下是 `app.config.server.ts` 文件的主要作用：

1. **合并配置**：
   - 使用 `mergeApplicationConfig` 函数将客户端配置与服务器端配置合并。
   - 这样可以确保服务器端和客户端共享相同的配置基础，同时允许特定于服务器的配置覆盖或扩展。
2. **提供服务器端渲染支持**：
   - `provideServerRendering()` 是一个提供者，它为服务器端渲染添加必要的支持。
   - 在 `serverConfig` 中，通过 `providers` 数组添加了 `provideServerRendering()` 提供者。
3. **导出最终配置**：
   - 最终的配置通过 `export const config` 导出，以便在其他地方（如主服务器文件）使用。



`app.config.ts` 文件主要用于配置 Angular 应用程序的基础配置，特别是针对客户端的初始化和运行时配置

```typescript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';

import { routes } from './app.routes';
import { provideClientHydration } from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [provideZoneChangeDetection({ eventCoalescing: true }), provideRouter(routes), provideClientHydration()]
};
```

**主要作用**

1. **配置全局提供者**：
   - 通过 `providers` 数组配置全局提供者，这些提供者在整个应用程序范围内有效。
2. **路由配置**：
   - 配置应用程序的路由，确保正确的路由功能。
3. **事件处理**：
   - 配置事件处理机制，如 Zone.js 的变更检测。
4. **客户端水合**：
   - 配置客户端水合（hydration），确保客户端能够正确接管服务器端渲染的内容。

**逐行解释**

1. **导入必要模块**：
   - `ApplicationConfig` 和 `provideZoneChangeDetection` 从 `@angular/core` 导入。
   - `provideRouter` 从 `@angular/router` 导入。
   - `provideClientHydration` 从 `@angular/platform-browser` 导入。
   - `routes` 从 `./app.routes` 导入。
2. **定义配置对象**：
   - `appConfig` 是一个 `ApplicationConfig` 类型的对象。
3. **配置提供者**：
   - **provideZoneChangeDetection({ eventCoalescing: true })**：配置 Zone.js 的变更检测机制，提高性能。
     - `eventCoalescing: true` 表示启用事件合并，减少不必要的事件处理。
   - **provideRouter(routes)**：配置应用程序的路由。
     - `routes` 是从 `./app.routes` 导入的路由配置。
   - **provideClientHydration()**：配置客户端水合，确保客户端能够接管服务器端渲染的内容。

`main.ts` 文件的主要作用是启动 Angular 应用程序，并将 `AppComponent` 渲染到 `index.html` 上

### 组件

每个组件由以下四个部分组成

1. 一个包含一些配置的`@Component`装饰器
2. 控制将渲染到 DOM 中的 `HTML模板`
3. 定义组件在 HTML 中如何使用的`CSS选择器`
4. 一个包含管理状态、处理用户输入或从服务器获取数据等行为的 `TypeScript类`

```typescript
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  title = 'angular-demo';
}
```

> 逐行解释以上配置

1. **selector: 'app-root'**
   - **作用**：定义组件的选择器（selector），即在模板中如何引用这个组件。
   - **示例**：在 HTML 中使用 `<app-root></app-root>` 标签来引用 `AppComponent`
2. **standalone: true**
   - **作用**：表示这是一个独立组件，不需要依赖其他模块。
   - **优点**：简化了组件的定义和使用，减少了对全局模块的依赖。
3. **imports: [RouterOutlet]**
   - **作用**：导入所需的其他组件或指令。
   - **示例**：这里导入了 `RouterOutlet`，用于路由导航。
   - **优点**：确保组件内部可以正常使用 `RouterOutlet` 组件。
4. **templateUrl: './app.component.html'**
   - **作用**：指定组件的模板文件路径。
   - **示例**：模板文件位于当前目录下的 `app.component.html` 文件中。
5. **styleUrls: ['./app.component.scss']**
   - **作用**：指定组件的样式文件路径。
   - **示例**：样式文件位于当前目录下的 `app.component.scss` 文件中。
6. **类定义**
   - **title**: 组件的一个属性，用于存储组件的标题。
   - **示例**：`title = 'angular-demo'`，表示组件的标题为 `'angular-demo'`。也可以在模板中使用该参数比如`<div>页面标题是{{title}}</div>`

如果希望在独立组件中使用其他组件

> 注意这里的定义的`selector`是`todo-list-item`，会作为模板里的选择器使用

```typescript
@Component({
  standalone: true,
  selector: 'todo-list-item',
  template:`<li>(TODO) Read Angular Essentials Guide</li>`
})
export class TodoListItem  {
  title = 'angular-demo';
}
```

> 在app组件中使用

```typescript
import { TodoListItem } from '../todo/todo.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet,TodoListItem],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  title = 'angular-demo';
}
```

>`todo-list-item`标签来引用子组件

```html
<main class="main">
  <div>当前页面是{{title}}</div>
  <button class="btn">首页</button>
  <todo-list-item></todo-list-item>
</main>

<router-outlet />
```

如果你不希望使用独立组件，其实在使用独立组件之前，Angular 代码使用 `NgModule` 作为导入和使用其他组件的机制

[参考文档](https://angular.cn/guide/ngmodules)

### 管理动态数据

#### 定义状态

要定义状态，你可以在组件内使用[类字段语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Public_class_fields)

例如，使用 `TodoListItem` 组件，创建两个你想要跟踪的属性：

1. `taskTitle` — 任务的标题
2. `isComplete` — 任务是否完成

```typescript
@Component({
  standalone: true,
  selector: 'todo-list-item',
  template:`这里是{{taskTitle}},您{{isComplete?'已经':'还没有'}}完成`
})
export class TodoListItem  {
  taskTitle = '代办事项';
  isComplete = false;
}
```

#### 更新状态

> 这里并没有绑定事件，在后续中会提到如何绑定事件

```typescript
@Component({
  standalone: true,
  selector: 'todo-list-item',
  template:`
  <div>这里是{{taskTitle}},您{{isComplete?'已经':'还没有'}}完成</div>
  `
})
export class TodoListItem  {
  taskTitle = '代办事项';
  isComplete = false;

  /**
   * 完成待办事项
   */
  completeTask() {
    this.isComplete = true;
  }

  /**
   * 更新待办事项
   */
  updateTitle(newTitle: string) {
    this.taskTitle = newTitle;
  }
}
```

### 渲染动态模板

#### 渲染动态数据

> 使用`双花括号`来渲染数据

```typescript
@Component({
  selector: 'todo-list-item',
  template: `
    <p>Title: {{ taskTitle }}</p>
  `,
})
export class TodoListItem {
  taskTitle = 'Read cup of coffee';
}
```

#### 动态属性

当你需要在 HTML 元素上动态设置标准 DOM 属性的值时，该属性会用方括号包裹，以通知 Angular 声明的值应被解释为类似 JavaScript 的语句，而不是纯字符串

```typescript
@Component({
  standalone: true,
  selector: 'todo-list-item',
  template:`
  <input type="text" [value]="taskTitle">
  `
})
export class TodoListItem  {
  taskTitle = '代办事项';
  isComplete = false;
}
```

如果你想动态绑定自定义的 HTML 属性（例如， `aria-`， `data-` 等），你可能会倾向于用相同的方括号包裹自定义属性

但需要**注意**的是，这样的实现无效

```typescript
@Component({
  standalone: true,
  selector: 'todo-list-item',
  template:`
  <button [data-test-title]="taskTitle">点击</button>
  `
})
export class TodoListItem  {
  taskTitle = '代办事项';
  isComplete = false;
}
```

> 应该使用如下方法设置前缀`attr.`来使用

```typescript
@Component({
  standalone: true,
  selector: 'todo-list-item',
  template:`
  <button [attr.data-test-title]="taskTitle">点击</button>
  `
})
export class TodoListItem  {
  taskTitle = '代办事项';
}
```

### 条件与循环

> 开发中经常需要根据条件进行dom渲染

#### `@if`控制块

类似于 JavaScript 的 `if` 语句，Angular 使用 `@if` 控制块有条件地隐藏和显示模板及其内容的部分

```typescript
@Component({
  standalone: true,
  selector: 'todo-list-item',
  template:`
  @if(showFlag){
    <button>隐藏</button>
  }
  `
})
export class TodoListItem  {
  showFlag=false;
}
```

只有当`showFlag`为true时，才会渲染`button`按钮

#### `@else`控制块

```typescript
@Component({
  standalone: true,
  selector: 'todo-list-item',
  template:`
  @if(showFlag){
    <button>隐藏</button>
  }@else {
    <button>显示</button>
  }
  
  `
})
export class TodoListItem  {
  showFlag=false;
}
```

#### `@else if`控制块

```typescript
@Component({
  standalone: true,
  selector: 'todo-list-item',
  template:`
  @if(showFlag1){
    <button>隐藏</button>
  }@else if(showFlag2) {
    <button>显示</button>
  }@else {
    <button>其他</button>
  }
  `
})
export class TodoListItem  {
  showFlag1=false;
  showFlag2=true;
}
```

#### `@for`控制块

类似于 JavaScript 的 `for...of` 循环，Angular 提供了 `@for` 控制块来渲染重复的元素

需要注意的是还有一个额外的 `track` 关键字

当 Angular 使用 `@for` 渲染元素列表时，这些条目可能会在后续更改或移动。Angular 需要通过任何重新排序跟踪每个元素，通常是将条目的某个属性作为唯一标识符或键

这确保了对列表的任何更新都能在 UI 中正确反映，并且在 Angular 内部得到正确的跟踪，特别是在有状态元素或动画的情况下

为此，我们可以使用 `track` 关键字为 Angular 提供唯一的键

```typescript
@Component({
  standalone: true,
  selector: 'todo-list-item',
  template: `
    @for(user of userList; track user.name){
      <div>
      <li>姓名:{{user.name}}</li>
      <li>年龄:{{user.age}}</li>
      <li>性别:{{user.gender}}</li>
      </div>
    }
  `
})
export class TodoListItem {
  userList = [
    {
      name: 'John',
      age: 25,
      gender: '男'
    },
    {
      name: 'Jane',
      age: 30,
      gender: '女'
    },
    {
      name: 'Jack',
      age: 40,
      gender: '男'
    }
  ]
}
```

### 用户交互

#### 事件处理

可以通过以下方式向元素添加事件处理器：

1. 在圆括号内添加一个带有事件名称的属性
2. 指定当事件触发时你想运行的 JavaScript 语句

```typescript
@Component({
  standalone: true,
  selector: 'todo-list-item',
  template: `
    <button (click)="testClick()">点击</button>
  `
})
export class TodoListItem {
  testClick() {
    console.log('click')
  }
}
```

> 如果需要访问`事件对象event`，Angular 提供了一个隐式的 `$event` 变量，你可以将其传递给一个函数

```html
<button (click)="createUser($event)">Submit</button>
```

### 共享逻辑

当你需要在组件之间共享逻辑时，Angular 利用 [依赖注入](https://angular.cn/guide/di)的设计模式，允许你创建一个“服务”，从而允许你将代码注入组件，并从单一事实之源管理它

#### 服务

服务是可重用的代码片段，可以被注入

类似于定义组件，服务由以下内容组成：

- 一个 **TypeScript 装饰器**，通过 `@Injectable` 声明该类为 Angular 服务，并允许你通过 `providedIn` 属性（通常为 `'root'`）定义应用程序中哪些部分可以访问该服务，从而允许服务在应用程序中的任何地方被访问
- 一个 **TypeScript 类**，定义当服务被注入时将可访问的代码

这是一个 `Calculator` 服务的示例

```typescript
import {Injectable} from '@angular/core';
@Injectable({
  providedIn: 'root',
})
export class CalculatorService {
  add(x: number, y: number) {
    return x + y;
  }
}
```

当你想在组件中使用服务时，你需要：

1. 导入服务
2. 声明一个类字段，服务被注入到其中。将类字段赋值为调用内置函数 `inject` 创建服务的结果

这是在 `Receipt`组件中的可能样子：

```typescript
import { Component, inject } from "@angular/core";
import { CalculatorService } from "../share/test.service";


@Component({
  standalone: true,
  selector: 'todo-list-item',
  styleUrl: './todo.component.scss',
  template: `
    <button class="btn btn-default" (click)="useShareService()">计算</button>
  `
})
export class TodoListItem {
  private calculatorService = inject(CalculatorService);
  useShareService() {
    let result = this.calculatorService.add(1, 2)
    console.log(`result:${result}`)
  }
}
```

> inject仅支持angular14以上的版本，否则可以使用传统方法进行注入

```typescript
import { Component } from "@angular/core";
import { CalculatorService } from "../share/test.service";


@Component({
  standalone: true,
  selector: 'todo-list-item',
  styleUrl: './todo.component.scss',
  template: `
    <button class="btn btn-default" (click)="useShareService()">计算</button>
  `
})
export class TodoListItem {
  constructor(private calculatorService: CalculatorService) { }
  useShareService() {
    let result = this.calculatorService.add(1, 2)
    console.log(`result:${result}`)
  }
}
```

## 深度指南

### 组件的剖析

> 如何导入以及使用其他`独立组件`在上方`基本要点`中已经了解到，如果希望使用`NgModule`的方式而非`独立组件`，可以参考[文档](https://angular.cn/guide/ngmodules)

#### 组件选择器

##### 类型选择器

每个组件都定义了一个css选择器用于使用该组件

```typescript
@Component({
  selector: 'profile-photo',
  ...
})
export class ProfilePhoto { }
```

**Angular在编译时静态匹配选择器**。在运行时改变DOM，无论是通过Angular绑定还是DOM API，都不会影响渲染的组件

**一个元素只能匹配一个组件选择器。**如果多个组件选择器匹配一个元素，Angular会报错

**`需要注意的是`**，**组件选择器是区分大小写的**

##### 选择器类型

>Angular 在组件选择器中支持一组 [基本 CSS 选择器类型](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors)

| 选择器类型 | 说明                                             | 例子                      |
| ---------- | ------------------------------------------------ | ------------------------- |
| 类型选择器 | 根据HTML标签名称或节点名称匹配元素。             | profile-photo             |
| 属性选择器 | 根据HTML属性的存在以及可选的属性确切值匹配元素。 | [dropzone] [type="reset"] |
| 类选择器   | 根据CSS类的存在匹配元素。                        | .menu-item                |

Angular 组件选择器不支持组合器，包括 [后代组合器](https://developer.mozilla.org/en-US/docs/Web/CSS/Descendant_combinator)或 [子组合器](https://developer.mozilla.org/en-US/docs/Web/CSS/Child_combinator)

Angular 组件选择器不支持指定 [命名空间](https://developer.mozilla.org/en-US/docs/Web/SVG/Namespaces_Crash_Course)

##### `:not`伪类

Angular 支持 [伪类 `:not`](https://developer.mozilla.org/en-US/docs/Web/CSS/:not)，可以将这个伪类附加到任何其他选择器上，以缩小组件选择器匹配的元素范围

可以定义一个 `[dropzone]`属性选择器并阻止匹配 `textarea`元素

```typescript
@Component({
  standalone: true,
  selector: '[dropzone]:not(textarea)',
  styleUrl: './todo.component.scss',
  template: `
    你好
  `
})
export class TodoListItem {
}
```

##### 组合选择器

可以通过它们来组合多个选择器。比如，可以匹配指定了 `type="reset"`的 `<button>`元素

```typescript
@Component({
  standalone: true,
  selector: 'button[type="reset"]',
  styleUrl: './todo.component.scss',
  template: `
    你好
  `
})
export class TodoListItem {
}
```

> 也可以使用逗号分隔的列表定义多个选择器

```typescript
@Component({
  standalone: true,
  selector: 'button[type="reset"],.btn-info',
  styleUrl: './todo.component.scss',
  template: `
    你好
  `
})
export class TodoListItem {
}
```

此时会选择`type`为`reset`的button元素`以及`类名为`btn-info`的元素

### 指定样式

#### 样式化组件

> 可以在`xxx.component.ts`中直接配置html与对应的css样式

```ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  template:`<button class="btn">点击</button>`,
  styles:`.btn{
    height: 100px;
    width: 200px;
  }`
})
export class AppComponent {
  title = 'angular-app';
}
```

> 不过将`html`和`css`抽出作为单独文件更加规范一点

```ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  title = 'angular-app';
}
```

#### 样式作用域

> 每个组件都有一个 **视图封装**设置，这个设置决定了框架如何限定组件的样式作用域。有三种视图封装模式： `Emulated`、 `ShadowDom` 和 `None`。你可以在 `@Component` 装饰器中指定模式：

使用如下命令创建两个模块

```sh
# g代表generate，c代表component
ng g c user
ng g c order
```

> 接下来可以自行使用这两个模块测试一下样式的作用域
>
> 使用`encapsulation`指定

```ts
@Component({
  selector: 'app-user',
  standalone: true,
  imports: [],
  templateUrl: './user.component.html',
  styleUrl: './user.component.scss',
  encapsulation: ViewEncapsulation.None,
})
export class UserComponent {
}
```

##### `ViewEncapsulation.Emulated`

> angular默认情况采用这种模拟封装，这样指定的样式只在模板中生效，不对其他模板造成干扰
>
> 框架会为每个组件实例生成唯一一个HTML属性，并且会将该属性添加到组件模板中的元素，以及将该属性插入到组件样式定义的css选择器中

在封装模式下，angular支持`:host`和`:host-context()`伪类，而无需使用 [Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM)。在编译期间，框架将这些伪类转换为属性，因此在运行时不遵循这些本地伪类的规则（例如浏览器兼容性、特异性）。Angular 的模拟封装模式不支持与 Shadow DOM 相关的任何其他伪类，例如 `::shadow` 或  `::part`。

##### `ViewEncapsulation.ShadowDom`

> 这种模式通过使用 [Web 标准 Shadow DOM API](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM) 来限定组件内的样式作用域。启用这种模式时，Angular 会将一个 shadow root 附着到组件的宿主元素，并将组件的模板和样式渲染到对应的 shadow 树中。
>
> 此模式严格保证 *只有*该组件的样式适用于组件模板中的元素。全局样式不能影响影子树中的元素，影子树中的样式也不能影响影子树外的元素。

然而，启用 `ShadowDom` 封装不仅仅影响样式作用域。在 shadow 树中渲染组件会影响事件传播、与 [`` API](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_templates_and_slots) 的交互，以及浏览器开发者工具显示元素的方式。在启用此选项之前，务必理解在你的应用中使用 Shadow DOM 的全部影响。

`ViewEncapsulation.None`

这种模式会禁用组件的所有样式封装。任何与组件关联的样式都会作为全局样式来处理。

##### 定义样式

你可以在组件的模板中使用 `<style>` 元素来定义额外样式。组件的视图封装模式会应用于这种方式定义的样式。

`Angular 不支持在样式元素内部进行绑定。`

这句话是啥意思呢，请看如下代码

> angular不支持

```ts
<style>
    .myClass {
        color: {{ someColor }};
    }
</style>

<!-- 模板部分 -->
<div class="myClass">Hello World!</div>
```

> 正确做法是使用`[ngClass]`或`[ngStyle]`

```ts
<div [ngStyle]="{'color': someColor}">Hello World!</div>

<!-- 或者使用类名 -->
<div [ngClass]="someClass">Hello World!</div>
```

> 可以使用全局或局部css变量

```typescript
<style>
  :host{
    --text-color:red;
  }
  .text{
    color: var(--text-color);
  }
</style>

<p class="text">user works!</p>
```

#### 接收输入属性中的数据





> 以上是官方文档内记载的内容

### 属性绑定

> 属性绑定使用中括号`[]`将属性包裹进行绑定
>
> 这里绑定的属性需要时dom已经存在的属性，比如如果希望绑定自定义的属性，就需要使用`[attr.自定义属性名]`

```html
<main class="main">
  <!-- 在component中定义name为`angular_name` -->
  <!-- 所以这里的class就是`angular_name` -->
  <div [class]="name">abc</div>

  <!-- 而如果希望使用字符串而不是变量，则需要使用单引号继续包裹对象 -->
  <!-- 这里的class就是`angular_name` -->
  <div [class]="'angular_name'">abc</div>

  <!-- 这里绑定的属性需要时dom已经存在的属性，比如如果希望绑定自定义的属性 -->
  <!-- 则需要使用`[attr.自定义属性名]` -->
  <div [attr.data-name]="name">abc</div>

  <!-- 如果希望可以动态控制是否显示某个属性 -->
  <!-- 显示class名称(angular_name) -->
  <div [class.angular_name]="true">abc</div>
  <!-- 不显示class名称(angular_name) -->
  <div [class.angular_name]="false">abc</div>

  <!-- 绑定多个属性值 -->
  <div [class]="'abc_classname angular_name'">abc</div>
  <!-- 动态绑定 -->
  <div [class]="['abc_classname',name]">abc</div>
  <!-- 这里最后一个空格不能省`abc_classname ` -->
  <div [class]="'abc_classname '+name">abc</div>
  <!-- 控制多个属性值是否显示 -->
  <!-- 这种方法了解即可,我更推介下面的三元表达式来控制是否显示 -->
  <div [class]="{'abc_classname':false,'angular_name':true}">abc</div>
  <!-- 推荐：三元表达式控制多个属性值是否显示,react也常用这种方式 -->
  <div [class]="true?['abc_classname',name]:'abc_classname'">abc</div>
</main>
```

#### 样式绑定

> component

```ts
import { Component } from '@angular/core';
// 导入`NgStyle`模块
import { NgStyle, NgClass } from '@angular/common';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  // 导入`NgStyle`模块
  imports: [RouterOutlet, NgStyle, NgClass],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
}
```

> html

```html
<main class="main">
  <!-- 单一样式绑定 -->
  <div [style.backgroundColor]="'red'">abc</div>

  <!-- 带单位的样式绑定 -->
  <div [style.height.px]="300" style="background-color: green;">abc</div>

  <!-- 多重样式绑定 -->
  <div [style]="'background-color:grey;height:200px'">abc</div>
  <div [style]="{'background-color':'grey','height':'200px'}">abc</div>

  <!-- 动态绑定样式 -->
  <!-- 这里需要在component导入`NgStyle`模块 -->
  <div [ngStyle]="{'color':'red'}">abc</div>
  <div [ngStyle]="{'color':false?'green':'grey'}">abc</div>
  <!-- NgClass也是同理 -->
  <div [ngClass]="{'class_name1':true,'class_name2':false,'class_name3':true}">abc</div>
</main>
```

### 条件判断

> 这种方法也需要导入`NgIf`模块

```html
<main class="main">
  <!-- 条件判断语法 -->
  <!-- 这种方式是直接控制是否渲染,而非`display:none`的显示隐藏 -->
  <div *ngIf="flag">aaa</div>
  <div *ngIf="!flag">bbb</div>

  <!-- 以上代码解析完之后,其实本质的模板如下 -->
  <ng-template [ngIf]="flag">
    <div *ngIf="flag">aaa</div>
  </ng-template>

  <!-- 那如果条件判断需要分支实现 -->
  <div *ngIf="flag;else otherTemplate">
    <div>ccc</div>
  </div>
  <ng-template #otherTemplate>
    <div>bbb</div>
  </ng-template>

  <!-- 其实还可以使用以下方式进行条件判断 -->
  <!-- `@if`和`ngIf`的区别是`ngIf`是指令语法,`@if`是流程控制语法(angular新特性) -->
  <!-- `@if`的性能更好,`ngIf`需要导入`NgIf`模块 -->
  <!-- 而`@if`是内置语法,无需额外导入,而且支持更复杂的条件分支 -->

  <!-- @if (condition1) {
    <div>content1</div>
  } @else if (condition2) {
    <div>content2</div>
  } @else {
    <div>content3</div>
  } -->
  @if(true){
  <div>显示</div>
  }@else{
  <div>隐藏</div>
  }
</main>
```

### 循环语句

> component

```ts
import { Component } from '@angular/core';
import { NgFor, NgForOf, NgSwitch, NgSwitchCase, NgSwitchDefault } from '@angular/common';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  // 导入`NgIf`模块
  imports: [RouterOutlet, NgFor, NgForOf, NgSwitch, NgSwitchCase, NgSwitchDefault],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  colorList = ['red', 'green', 'grey', 'pink'];

  type = 1;


  userList: User[] = [{
    id: 1,
    name: 'John',
    age: 25,
    gender: '男'
  },
  {
    id: 2,
    name: 'Jane',
    age: 30,
    gender: '女'
  },
  {
    id: 3,
    name: 'Jack',
    age: 40,
    gender: '男'
  }]
}

interface User {
  id: number;
  name: string;
  age: number;
  gender: string;
}
```

> html

```html
<main class="main">
  <!-- 循环语句 -->
  <!-- for -->
  <div *ngFor="let color of colorList let odd=odd let i=index">
    <!-- 当前索引是否是偶数 -->
    {{odd}}
    <!-- 当前索引,从0开始 -->
    {{i}}
    <!-- 当前索引指向的字符串 -->
    {{color}}
  </div>
  <!-- 以上代码解析完之后,模板内容如下 -->
  <ng-template ngFor let-color [ngForOf]="colorList" let-i="index" let-odd="odd">
    <div>
      {{odd}}
      {{i}}
      {{color}}
    </div>
  </ng-template>


  <!-- switch -->
  <div [ngSwitch]="type">
    <!-- `type`的值为1 -->
    <p *ngSwitchCase="1">1</p>
    <!-- `type`的值为2 -->
    <p *ngSwitchCase="2">2</p>
    <!-- `type`的值既不是1,也不是2 -->
    <p *ngSwitchDefault>3</p>
  </div>

  <!-- 除了以上的循环语句之后,在angular17中发布的新特性 -->
  <!-- `track`使用唯一id来提升遍历性能 -->
  @for(user of userList;track user.id){
  <div>
    {{user.name}}=={{user.age}}=={{user.gender}}
  </div>
  }

  @switch (type) {
  @case (1) {
  <p>1</p>
  }@case (2) {
  <p>2</p>
  }@default {
  <p>3</p>
  }
  }
</main>
```

### 事件绑定

> component

```ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  num = 1;
  incrementNum(event: Event): void {
    // 获取事件对象e
    console.log(event.target);
    this.num++;
  }

  inputValue(event: Event): void {
    // 打印当前input
    console.log(event.target);
  }

  getUserName(val: string): void {
    console.log(`当前input的值为${val}`);
  }

}
```

> html

```html
<main class="main">
  {{num}}
  <!-- click -->
  <button (click)="incrementNum($event)">点击增加</button>
  <!-- input -->
  <input type="text" (input)="inputValue($event)" #userName>

  <!-- 模板引用变量 -->
  <!-- 上面input定义了一个模板名称`userName` -->
  <!-- 可以通过这个input的模板名称来获取其值 -->
  <button (click)="getUserName(userName.value)">获取userName</button>
</main>
```

#### 数据双向绑定

> 双向绑定是应用中的组件共享数据的以种方式。使用双向绑定绑定来侦听事件并在父组件和子组件之间同步更新值
>
> ngModel指令**只对表单元素有效**,所以在使用使用之前需要导入`FormsModule`模块

```html
<main class="main">
  <!-- 双向绑定userName，双向绑定只对表单元素有效 -->
  <input type="text" [(ngModel)]="userName">
  <!-- 展示userName，判断变化双向绑定是否成功 -->
  <p>{{ userName }}</p>
</main>
```

### 表单控件

1. 在你的应用中注册响应式表单模块。该模块声明了一些你要用在响应式表单中的指令。
2. 生成一个新的`FormControl` 实例，并把它保存在组件中。
3. 在模板中注册这个`FormControl`

##### 注册响应式表单模块

> 导入`ReactiveFormsModule`

```ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { FormControl, FormsModule, ReactiveFormsModule } from '@angular/forms';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, FormsModule, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  userName = new FormControl('angular');

  changeUserName() {
    // 通过setValue方法修改表单对象的值
    this.userName.setValue(this.userName.value + "==");
  }
}
```

```html
<main class="main">
  <!-- 动态表单控件 -->
  <input type="text" [formControl]="userName">
  <p>{{ userName.value }}</p>

  <!-- 修改表单项 -->
  <button (click)="changeUserName()">修改</button>
</main>
```

### 表单控件分组

表单中通常会包含几个相互关联的控件。响应式表单提供了两种把多个相关控件分组到同一个输入表单中的方法

要将表单组添加到此组件中，请执行以下步骤。

1. 创建一个FormGroup实例
2. 把这个FormGroup 模型关联到视图
3. 保存表单数据

```ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { FormGroup, FormControl, FormsModule, ReactiveFormsModule } from '@angular/forms';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, FormsModule, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  loginForm = new FormGroup({
    username: new FormControl(''),
    password: new FormControl(''),
  });

  /**
   * 打印表单
   */
  logForm() {
    console.log(this.loginForm.value);
  }
}
```

```html
<main class="main">
  <!-- 使用`formGroup`绑定控件组 -->
  <div [formGroup]="loginForm">
    <div>
      <label for="username">账号:</label>
      <!-- 使用`[formControlName]`指定绑定的控件名称 -->
      <input type="text" name="username" placeholder="请输入账号" formControlName="username">
    </div>
    <div>
      <label for="password">密码:</label>
      <input type="password" name="password" placeholder="请输入密码" formControlName="password">
    </div>
    <button (click)="logForm()">打印表单</button>
  </div>
</main>
```

### 表单校验

表单元素添加`required`关键字表示必填、`minlength`表示输入的最小长度，通过绑定ngMode1的引用可以拿到到当前组件的信息，通过引用获取到验证的信息

```html
<main class="main">
  <div>
    <div>
      <label for="username">账号:</label>
      <input type="text" required name="username" placeholder="请输入账号" #usernameInput="ngModel"
        [(ngModel)]="loginForm.username">
      <p>username验证是否通过:验证{{usernameInput.invalid?'`不通过`':'`通过`'}}</p>
      <!-- 如果验证没有通过，而且`表单控件的值已经发生了变化` || `表单控件已经被用户输入或焦点` -->
      @if(usernameInput.invalid&&(usernameInput.dirty || usernameInput.touched)){
      <div class="alter">
        @if(usernameInput.errors?.['required']){
        <div>
          还未填写username
        </div>
        }
      </div>
      }
    </div>
    <div>
      <label for="password">密码:</label>
      <input type="password" required minlength="6" name="password" placeholder="请输入密码" #passwordInput="ngModel"
        [(ngModel)]="loginForm.password">
      <p>password验证是否通过:验证{{passwordInput.invalid?'`不通过`':'`通过`'}}</p>
      <!-- password表单校验 -->
      @if(passwordInput.invalid&&(passwordInput.dirty || passwordInput.touched)){
      <div class="alter">
        @if(passwordInput.errors?.['required']){
        <div>
          还未填写password
        </div>
        }
        @if(passwordInput.errors?.['minlength']){
        <div>
          password最小长度是6位
        </div>
        }
      </div>
      }
    </div>
    <button (click)="submitForm(usernameInput.value,passwordInput.value)">登录</button>
  </div>
</main>
```

```ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, FormsModule, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {

  loginForm = {
    username: '',
    password: ''
  }

  /**
   * 登录
   */
  submitForm(username: string, password: string) {
    console.log(`username:${username} password:${password}`);
  }
}
```

> 以上是内置校验器，接下来学习自定义校验器

1. 定义自定义验证器
2. 把自定义验证器添加到响应式表单中
3. 为模板驱动表单添加自定义验证器

**定义自定义验证器**

```ts
  /**
   * 密码正则校验器
   */
  passwordRegValidator(): ValidatorFn {
    // 正则表达式如下
    const passwordReg = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[$@,.])[a-zA-Z\d$@,.]{6,12}$/
    return (control: AbstractControl): ValidationErrors | null => {
      const forbidden = passwordReg.test(control.value);
      return forbidden ? null : { forbiddenName: { value: control.value } };
    };
  }
```

**把自定义验证器添加到响应式表单中**

```ts
  loginForm = new FormGroup({
    username: new FormControl('', [
      Validators.required
    ]),
    password: new FormControl('', [
      Validators.required,
      Validators.minLength(6),
      Validators.maxLength(12),
      this.passwordRegValidator()
    ])
  })
```

**为模板驱动表单添加自定义验证器**

```html
    <div>
      <!-- 密码如下 -->
      <!-- 1. 6~12位 -->
      <!-- 2. 必须包含数字、小写字母、大写字母、特殊字符($@,_.之一) -->
      <label for="password">密码:</label>
      <input type="password" required name="password" placeholder="请输入密码" formControlName="password">
    </div>
    <div class="alter">
      @if(checkPasswordIsValid){
      @if(loginForm.get('password')?.errors?.['required']){
      <span>密码不能为空</span>
      }@else if (loginForm.get('password')?.errors?.['minlength']) {
      <span>密码长度不能小于6位</span>
      }@else if (loginForm.get('password')?.errors?.['maxlength']) {
      <span>密码长度不能大于12位</span>
      }@else if (loginForm.get('password')?.errors?.['forbiddenName']) {
      <span>密码必须包含数字、小写字母、大写字母、特殊字符</span>
      }
      }
    </div>
```

完整代码如下

```html
<main class="main">
  <div [formGroup]="loginForm">
    <div>
      <label for="username">账号:</label>
      <input type="text" required name="username" placeholder="请输入账号" formControlName="username">
    </div>
    <div class="alter">
      @if(checkUsernameIsValid){
      <span>账号不能为空</span>
      }
    </div>
    <div>
      <!-- 密码如下 -->
      <!-- 1. 6~12位 -->
      <!-- 2. 必须包含数字、小写字母、大写字母、特殊字符($@,_.之一) -->
      <label for="password">密码:</label>
      <input type="password" required name="password" placeholder="请输入密码" formControlName="password">
    </div>
    <div class="alter">
      @if(checkPasswordIsValid){
      @if(loginForm.get('password')?.errors?.['required']){
      <span>密码不能为空</span>
      }@else if (loginForm.get('password')?.errors?.['minlength']) {
      <span>密码长度不能小于6位</span>
      }@else if (loginForm.get('password')?.errors?.['maxlength']) {
      <span>密码长度不能大于12位</span>
      }@else if (loginForm.get('password')?.errors?.['forbiddenName']) {
      <span>密码必须包含数字、小写字母、大写字母、特殊字符</span>
      }
      }
    </div>
    <button (click)="submitForm()">注册</button>
  </div>
</main>
```

```ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { AbstractControl, FormControl, FormGroup, FormsModule, ReactiveFormsModule, ValidationErrors, ValidatorFn, Validators } from '@angular/forms';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, FormsModule, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  loginForm = new FormGroup({
    username: new FormControl('', [
      Validators.required
    ]),
    password: new FormControl('', [
      Validators.required,
      Validators.minLength(6),
      Validators.maxLength(12),
      this.passwordRegValidator()
    ])
  })

  /**
   * 密码正则校验器
   */
  passwordRegValidator(): ValidatorFn {
    // 正则表达式如下
    const passwordReg = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[$@,.])[a-zA-Z\d$@,.]{6,12}$/
    return (control: AbstractControl): ValidationErrors | null => {
      const forbidden = passwordReg.test(control.value);
      return forbidden ? null : { forbiddenName: { value: control.value } };
    };
  }

  /**
   * 登录
   */
  submitForm() {
    if (this.loginForm.valid) {
      console.log(`username:${this.loginForm.value.username}  password:${this.loginForm.value.password}`);
    }
  }

  /**
   * 验证用户名是否合法
   */
  get checkUsernameIsValid(): boolean {
    return this.loginForm.get('username')?.errors?.['required'] &&
      (this.loginForm.get('username')?.dirty || this.loginForm.get('username')?.touched);
  }

  /**
   * 验证密码是否合法
   */
  get checkPasswordIsValid(): boolean {
    const requiredFlag = this.loginForm.get('password')?.errors?.['required'] &&
      (this.loginForm.get('password')?.dirty || this.loginForm.get('password')?.touched);
    const minLengthFlag = this.loginForm.get('password')?.errors?.['minlength'] &&
      (this.loginForm.get('password')?.dirty || this.loginForm.get('password')?.touched);
    const maxLengthFlag = this.loginForm.get('password')?.errors?.['maxlength'] &&
      (this.loginForm.get('password')?.dirty || this.loginForm.get('password')?.touched);
    const regExpFlag = this.loginForm.get('password')?.errors?.['forbiddenName'] &&
      (this.loginForm.get('password')?.dirty || this.loginForm.get('password')?.touched);
    console.log(`regExpFlag:${regExpFlag}`);

    return requiredFlag || minLengthFlag || maxLengthFlag || regExpFlag;
  }
}
```

### 管道

> 参考[博客](https://juejin.cn/post/7085762568223981605)

```ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { FormsModule, ReactiveFormsModule, } from '@angular/forms';
// 导入CommonModule模块
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, CommonModule, FormsModule, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  dateTime = new Date()
  money = 12345;
  lowerCaseStr = 'hello world';
  upperCaseStr = 'HELLO WORLD';
}
```

```html
<main class="main">
  <!-- 时间管道 -->
  <div>
    {{dateTime | date:'yyyy-MM-dd HH:mm:ss' }}
  </div>

  <!-- 人名币格式的管道 -->
  <div>
    {{money | currency }}
  </div>

  <!-- 字符串转大写 -->
  <div>
    {{lowerCaseStr | uppercase }}
  </div>

  <!-- 字符串转小写 -->
  <div>
    {{upperCaseStr | lowercase}}
  </div>
</main>

<router-outlet />
```

#### 自定义管道

> 快捷生成管道类

```sh
# ng g p pipes/pipe-name

ng g p pipes/strFilter
```

```ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'strFilter',
  standalone: true
})
export class StrFilterPipe implements PipeTransform {

  transform(value: string, ...args: string[]): string {
    // 打印结果:value:abc cba  args:a,c
    console.log(`value:${value}  args:${args}`);
    args.forEach((str) => {
      value = value.replaceAll(str, '')
    })
    // 返回结果是:b b
    return value;
  }

}
```

> 使用

```ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { FormsModule, ReactiveFormsModule, } from '@angular/forms';
import { CommonModule } from '@angular/common';
// 导入模块
import { StrFilterPipe } from './pipes/str-filter.pipe';

@Component({
  selector: 'app-root',
  standalone: true,
  // 导入模块
  imports: [RouterOutlet, CommonModule, StrFilterPipe, FormsModule, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  dateTime = new Date()
  money = 12345;
  lowerCaseStr = 'hello world';
  upperCaseStr = 'HELLO WORLD';
  strFilterPipeValue = 'abc cba';
}
```

```html
<main class="main">
  <!-- 自定义管道 -->
  <div>
    {{ strFilterPipeValue | strFilter :'a' :'c' }}
  </div>
</main>
```

### 组件通信

> 创建组件

```sh
# ng g c + 组件名
ng g c component_name
```

> 我们在引用组件时，主要关注`xxx.component.ts`这个文件

```ts
@Component({
  selector: 'app-user',
  standalone: true,
  imports: [],
  templateUrl: './user.component.html',
  styleUrl: './user.component.scss'
})
export class UserComponent {

}
```

这里我们定义的选择器是**app-user**

所以，如果要在其他组件中引用`UserComponent`组件，我们需要新建一个**app-user**标签

```html
<main class="main">
  <app-user></app-user>
</main>
```

除此之外还需要在`xxx.component.ts`中导入`UserComponent`

```html
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { FormsModule, ReactiveFormsModule, } from '@angular/forms';
import { CommonModule } from '@angular/common';
// 导入UserComponent
import { UserComponent } from './user/user.component';
@Component({
  selector: 'app-root',
  standalone: true,
  // 导入UserComponent
  imports: [RouterOutlet, CommonModule, UserComponent, FormsModule, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  dateTime = new Date()
  money = 12345;
  lowerCaseStr = 'hello world';
  upperCaseStr = 'HELLO WORLD';
  strFilterPipeValue = 'abc cba';
}
```

#### 生命周期

当你的应用通过调用构造函数来实例化一个组件或指令时，Angular 就会调用那个在该实例生命周期的适当位置实现了的那些钩子方法

Angular 会按以下顺序执行钩子方法。可以用它来执行以下类型的操作。

| 钩子方法                  | 用途                                                         | 时机                                                         |
| :------------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `ngOnChanges()`           | 当 Angular 设置或重新设置数据绑定的输入属性时响应。该方法接受当前和上一属性值的 `SimpleChanges` 对象 **注意**： 这发生得比较频繁，所以你在这里执行的任何操作都会显著影响性能。欲知详情，参阅本文档的[使用变更检测钩子](http://ubuntu-notebook:8015/guide/lifecycle-hooks#onchanges)。 | 如果组件绑定过输入属性，那么在 `ngOnInit()` 之前以及所绑定的一个或多个输入属性的值发生变化时都会调用。 **注意**： 如果你的组件没有输入属性，或者你使用它时没有提供任何输入属性，那么框架就不会调用 `ngOnChanges()`。 |
| `ngOnInit()`              | 在 Angular 第一次显示数据绑定和设置指令/组件的输入属性之后，初始化指令/组件。欲知详情，参阅本文档中的[初始化组件或指令](http://ubuntu-notebook:8015/guide/lifecycle-hooks#oninit)。 | 在第一轮 `ngOnChanges()` 完成之后调用，只调用**一次**。而且即使没有调用过 `ngOnChanges()`，也仍然会调用 `ngOnInit()`（比如当模板中没有绑定任何输入属性时）。 |
| `ngDoCheck()`             | 检测，并在发生 Angular 无法或不愿意自己检测的变化时作出反应。欲知详情和范例，参阅本文档中的[自定义变更检测](http://ubuntu-notebook:8015/guide/lifecycle-hooks#docheck)。 | 紧跟在每次执行变更检测时的 `ngOnChanges()` 和 首次执行变更检测时的 `ngOnInit()` 后调用。 |
| `ngAfterContentInit()`    | 当 Angular 把外部内容投影进组件视图或指令所在的视图之后调用。 欲知详情和范例，参阅本文档中的[响应内容中的变更](http://ubuntu-notebook:8015/guide/lifecycle-hooks#aftercontent)。 | 第一次 `ngDoCheck()` 之后调用，只调用一次。                  |
| `ngAfterContentChecked()` | 每当 Angular 检查完被投影到组件或指令中的内容之后调用。 欲知详情和范例，参阅本文档中的[响应被投影内容的变更](http://ubuntu-notebook:8015/guide/lifecycle-hooks#aftercontent)。 | `ngAfterContentInit()` 和每次 `ngDoCheck()` 之后调用。       |
| `ngAfterViewInit()`       | 当 Angular 初始化完组件视图及其子视图或包含该指令的视图之后调用。 欲知详情和范例，参阅本文档中的[响应视图变更](http://ubuntu-notebook:8015/guide/lifecycle-hooks#afterview)。 | 第一次 `ngAfterContentChecked()` 之后调用，只调用一次。      |
| `ngAfterViewChecked()`    | 每当 Angular 做完组件视图和子视图或包含该指令的视图的变更检测之后调用。 | `ngAfterViewInit()` 和每次 `ngAfterContentChecked()` 之后调用。 |
| `ngOnDestroy()`           | 每当 Angular 每次销毁指令/组件之前调用并清扫。在这儿反订阅可观察对象和分离事件处理器，以防内存泄漏。欲知详情，参阅本文档中的[在实例销毁时进行清理](http://ubuntu-notebook:8015/guide/lifecycle-hooks#ondestroy)。 | 在 Angular 销毁指令或组件之前立即调用。                      |

#### 父子组件传参

参考[博客](https://juejin.cn/post/7205841266906398781)

##### 父传子

`@Input()` 和 `@Output()` 为子组件提供了一种与其父组件通信的方法。`@Input()` 允许父组件更新子组件中的数据。相反，`@Output()` 允许子组件向父组件发送数据。

```ts
@Component({
  selector: 'app-root',
  standalone: true,
  // 导入UserComponent
  imports: [RouterOutlet, CommonModule, UserComponent, FormsModule, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent implements OnInit {
  message = '你好，初次见面';
  ngOnInit(): void {
    console.log('初始化...');
  }
```

> 在父组件中定义一个字符串`message`作为需要传递给`user`子组件的内容

```html
<main class="main">
  <app-user [msg]="message"></app-user>
</main>
```

> 像dom节点设置属性一样传递内容
>
> 接下来在子组件中接收

```ts
@Component({
  selector: 'app-user',
  standalone: true,
  imports: [],
  templateUrl: './user.component.html',
  styleUrl: './user.component.scss'
})
export class UserComponent {
  // 使用`@Input()`装饰器接收
  @Input() msg = '';
}
```

```html
<p>msg:{{msg}}</p>
```

##### 子传父

> 将输入框的内容传递给`addNewIten`函数

```html
<input type="text" #childMsg>
<button type="button" (click)="addNewIten(childMsg.value)">传递信息给父组件</button>
```

>使用`@Output()`装饰器标注信息，通过`xxx.emit('msg')`向父组件传递信息

```ts
@Component({
  selector: 'app-user',
  standalone: true,
  imports: [],
  templateUrl: './user.component.html',
  styleUrl: './user.component.scss'
})
export class UserComponent {
  @Output() msg = new EventEmitter<string>();

  addNewIten(childMsg: string): void {
    this.msg.emit(`向父组件传递信息:【${childMsg}】`);
  }
}
```

> 父组件通过`(msg)="getChildMsg($event)"`接收信息

```html
<main class="main">
  <app-user (msg)="getChildMsg($event)"></app-user>
  @if(childMsg!==null && childMsg!==''){
  父组件接收到值:{{childMsg}}
  }
</main>
```

```ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, CommonModule, UserComponent, FormsModule, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  childMsg = '';
  getChildMsg(childMsg: string) {
    this.childMsg = childMsg;
  }
}
```

##### 获取子组件实例

> 如果在父组件中，希望直接获取到子组件的实例，或者希望获取到子组件里的某个函数、属性

**以下是子组件中的代码**

```html
<p>{{title}}</p>
```

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-user',
  standalone: true,
  imports: [],
  templateUrl: './user.component.html',
  styleUrl: './user.component.scss'
})
export class UserComponent {
  title = '这里是子组件';
}
```

**以下是父组件中的代码**

```html
<main class="main">
  <app-user #childComponent></app-user>
  <button type="button" class="btn btn-success" (click)="logChildComponent()">点击获取子组件</button>
</main>

<router-outlet />
```

```ts
import { Component, ViewChild } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { FormsModule, ReactiveFormsModule, } from '@angular/forms';
import { CommonModule } from '@angular/common';
import { UserComponent } from './user/user.component';
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, CommonModule, UserComponent, FormsModule, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  @ViewChild('childComponent') child?: UserComponent;

  logChildComponent() {
    console.log(this.child?.title);
  }
}
```

这样就可以通过获取到`子组件实例`从而取得子组件里的属性

### 服务

Angular 把组件和服务区分开，以提高模块性和复用性。

理想情况下，组件的工作只管用户体验，而不用顾及其它。它应该提供用于数据绑定的属性和方法，以便作为视图和应用逻辑的中介者。视图就是模板所渲染的东西，而程序逻辑就是用于承载模型概念的东西。

```sh
# 生成服务(service目录下的list服务)
ng g s service/list
```

> 接下来演示一下用法

```ts
export interface User {
  id: number;
  name: string;
  nickName: string;
}
```

**这里通常是发请求向后端请求数据**，我这里暂时先把数据写死了

```ts
import { Injectable } from '@angular/core';
import { User } from '../entity/User';

@Injectable({
  // 作用域设定，`root`表示默认注入，注入到`AppModule`中
  providedIn: 'root'
})
export class ListService {

  listUser(): User[] {
    return [{
      id: 1,
      name: 'zhangsan',
      nickName: '张三'
    }, {
      id: 2,
      name: 'lisi',
      nickName: '李四'
    }, {
      id: 3,
      name: 'wangwu',
      nickName: '王五'
    }];
  }
}
```

> 导入服务使用

```ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { FormsModule, ReactiveFormsModule, } from '@angular/forms';
import { CommonModule } from '@angular/common';
import { ListService } from './service/list.service';
import { User } from './entity/User';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, CommonModule, FormsModule, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  list?: User[];
  // 注入服务
  constructor(private listService: ListService) { }
  showUsers() {
    this.list = this.listService.listUser();
  }
}
```

```html
<main class="main">
  @for(user of list;track user.id){
  <div>id:{{user.id}}===name:{{user.name}}==nickName:{{user.nickName}}</div>
  }
  <button type="button" class="btn btn-success" (click)="showUsers()">展示用户列表</button>
</main>
```

### 路由

> 参考[文档](https://angular.cn/guide/routing/router-tutorial)

**app.routes.ts**

```ts
import { Routes } from '@angular/router';
import { UserComponent } from './user/user.component';
import { OrdersComponent } from './orders/orders.component';

export const routes: Routes = [
  { path: 'user', component: UserComponent, pathMatch: 'full' },
  { path: 'orders', component: OrdersComponent, pathMatch: 'full' },
];
```

**app.config.ts**

```ts
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';

import { routes } from './app.routes';
import { provideClientHydration } from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    // 引入路由配置
    provideRouter(routes),
    provideClientHydration()
  ]
};
```

这样就实现了路由的配置，如果希望通过`link`进行路由跳转

```html
<main class="main">
  <p>
    <a class="button" routerLink="/user">user</a>
  </p>
  <p>
    <a class="button" routerLink="/orders">orders</a>
  </p>
</main>
```

在`app.component.html`中使用`routerLink`进行跳转，注意要导包

```ts
import { Component } from '@angular/core';
import { RouterOutlet, RouterLink } from '@angular/router';
import { FormsModule, ReactiveFormsModule, } from '@angular/forms';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    RouterOutlet,
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    // 导包
    RouterLink
  ],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
}
```

#### 标注出当前路由

如果我们希望在`RouterLink`的基础上进一步，对当前路由的`a`标签展示更加明显的`css`样式

可以使用`routerLinkActive`属性，当然，需要导入`RouterLinkActive`的包

```html
<main class="main">
  <nav>
    <a class="button" routerLink="user" routerLinkActive="activebutton">user</a> |
    <a class="button" routerLink="orders" routerLinkActive="activebutton">orders</a>
  </nav>
</main>

<router-outlet />
```

```css
.button {
  box-shadow: inset 0 1px 0 0 #ffffff;
  background: #ffffff linear-gradient(to bottom, #ffffff 5%, #f6f6f6 100%);
  border-radius: 6px;
  border: 1px solid #dcdcdc;
  display: inline-block;
  cursor: pointer;
  color: #666666;
  font-family: Arial, sans-serif;
  font-size: 15px;
  font-weight: bold;
  padding: 6px 24px;
  text-decoration: none;
  text-shadow: 0 1px 0 #ffffff;
  outline: 0;
}
.activebutton {
  box-shadow: inset 0 1px 0 0 #dcecfb;
  background: #bddbfa linear-gradient(to bottom, #bddbfa 5%, #80b5ea 100%);
  border: 1px solid #84bbf3;
  color: #ffffff;
  text-shadow: 0 1px 0 #528ecc;
}
```

这意味着，当前路由会使用`.activebutton`的样式

#### 重定向

```ts
export const routes: Routes = [
  { path: '', redirectTo: '/user', pathMatch: 'full' },
  { path: 'user', component: UserComponent, pathMatch: 'full' },
  { path: 'orders', component: OrdersComponent, pathMatch: 'full' },
];
```

> 这样配置之后，如果访问`http://localhost:4200`则会自动重定向到`http://localhost:4200/user`

#### 添加404页面

```html
<div id="root">
  <div class="box-404-wrap">
    <div class="box">
      <div class="d-flex flex-column align-items-center">
        <div class="text-wrap">
          <h1 data-t="404" class="h1">404</h1>
        </div>
        <div class="text-center mt-2">页面不存在或者您没有权限访问。</div>
        <div class="mt-4"><a routerLink="/" role="button" tabindex="0" class="btn btn-primary">回首页</a></div>
      </div>
    </div>
  </div>
</div>
```

```ts
import { Routes } from '@angular/router';
import { UserComponent } from './user/user.component';
import { OrdersComponent } from './orders/orders.component';
import { NotFoundComponent } from './not-found/not-found.component';

export const routes: Routes = [
  { path: '', redirectTo: '/user', pathMatch: 'full' },
  { path: 'user', component: UserComponent, pathMatch: 'full' },
  { path: 'orders', component: OrdersComponent, pathMatch: 'full' },
  // 使用通配符来匹配所有路径,如果没有匹配的路由将会返回404页面
  { path: '**', component: NotFoundComponent },
];
```

#### 路由嵌套

```ts
import { Routes } from '@angular/router';
import { UserComponent } from './user/user.component';
import { OrdersComponent } from './orders/orders.component';
import { NotFoundComponent } from './not-found/not-found.component';
import { ListComponent } from './list/list.component';

export const routes: Routes = [
  { path: '', redirectTo: '/user', pathMatch: 'full' },
  {
    path: 'user',
    component: UserComponent,
    children: [
      // 访问子路由`/user/list`
      {
        path: 'list',
        component: ListComponent,
        pathMatch: 'full',
      }
    ]
  },
  { path: 'orders', component: OrdersComponent, pathMatch: 'full' },
  { path: '**', component: NotFoundComponent },
];
```

**user.component.html**

```html
<p>这里是user子组件</p>

<!-- 渲染user下的子路由 -->
<router-outlet></router-outlet>
```

#### 路由传参

> 路由传参一般有两种方式，一种是`xxx/1`斜线分割，一种是`xxx?name=张三`问号分割

```html
<main class="main">
  <nav>
    <a class="button" routerLink="user" routerLinkActive="activebutton" [queryParams]="{name:'aaa'}">user</a> |
    <a class="button" [routerLink]="['orders','aaa']" routerLinkActive="activebutton">orders</a>
  </nav>
</main>

<router-outlet />
```

通过点击这两个按钮进行传参

#####   query

> 访问`http://localhost:4200/user?name=aaa`

```ts
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-user',
  standalone: true,
  imports: [],
  templateUrl: './user.component.html',
  styleUrl: './user.component.scss'
})
export class UserComponent implements OnInit {
  constructor(private activatedRoute: ActivatedRoute) { }
  ngOnInit(): void {
    console.log('name:', this.activatedRoute.snapshot.queryParams['name']);
  }
}
```



##### parms

> 访问`http://localhost:4200/orders/aaa`

`router`需要新增一个`:name`表示传递参数的名字

```ts
import { Routes } from '@angular/router';
import { UserComponent } from './user/user.component';
import { OrdersComponent } from './orders/orders.component';
import { NotFoundComponent } from './not-found/not-found.component';

export const routes: Routes = [
  { path: '', redirectTo: '/user', pathMatch: 'full' },
  {
    path: 'user',
    component: UserComponent,
  },
  {
    path: 'orders/:name',
    component: OrdersComponent
  },
  { path: '**', component: NotFoundComponent },
];
```

```ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Params } from '@angular/router';

@Component({
  selector: 'app-orders',
  standalone: true,
  imports: [],
  templateUrl: './orders.component.html',
  styleUrl: './orders.component.scss'
})
export class OrdersComponent implements OnInit {

  constructor(private activatedRoute: ActivatedRoute) { }

  ngOnInit(): void {
    this.activatedRoute.params.subscribe((parms: Params) => {
      console.log('name:', parms['name']);
    });
  }
}
```

