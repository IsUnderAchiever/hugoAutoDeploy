---
title: NestJs
description: NestJs
date: 2024-07-18
slug: NestJs
image: 202412212154588.png
categories:
    - NestJs
---
# NestJs

## 前置知识

### IOC`控制反转`和DI`依赖注入`

> 以下A、B、C三个类之间的关系是强耦合的，当A类发生改变后，B类和C类也需要发生改变
>
> 创建以下`IOC.ts`文件

```ts
class A {
  name: string
  constructor(name: string) {
    this.name = name
  }
}

class B {
  a: any
  constructor() {
    this.a = new A('').name
  }
}

class C {
  a: any
  constructor() { 
    this.a = new A('').name
  }
}
```

> 这并不是我们所希望的，所以我们将使用`控制反转`的方式解决这个问题
>
> 使用`ts-node IOC.ts`来运行代码

```ts
class A {
  name: string
  constructor(name: string) {
    this.name = name
  }
}

class Container {
  module: any
  constructor() {
    this.module = {}
  }

  provide(key: string, value: any) {
    this.module[key] = value
  }

  get(key: string) {
    return this.module[key]
  }
}

const container = new Container()
container.provide('a', new A('唱'))
container.provide('c', new A('跳'))

class B {
  a: any
  c: any
  constructor() {
    this.a = container.get('a')
    this.c = container.get('c')
  }
}

console.log(container.get('a'))
console.log(container.get('c'))
console.log(new B())
```

> 类似于发布订阅（在发布者里存入需要推送的订阅者信息），实际上就是借助第三方作为容器来存入信息以消除强耦合

### 装饰器

#### 类装饰器

> 使用**tsc --init**来生成`tsconfig.json`文件,并将如下代码`解开注释`,我们需要开放这段代码

```json
"experimentalDecorators": true
```


> 再新建`index.ts`文件

```ts
const doc: ClassDecorator = (param: any) => {
  console.log(param)
}

@doc
class MyDoc {
  constructor() {}
}
```

这段代码的本质其实是将`MyDoc`的构造函数传入`doc`,即`param`

我们可以通过这种方法给`MyDoc`来添加属性

```ts
const doc: ClassDecorator = (param: any) => {
  param.prototype.name = 'doc'
}

@doc
class MyDoc {
  constructor() {}
}

const myDoc: any = new MyDoc()
console.log(myDoc.name)
```

```sh
ts-node .\index.ts
```

#### 属性装饰器

```ts
const doc: PropertyDecorator = (param: any, key: string | symbol) => {
  console.log(param, key)
}

class MyDoc {
  @doc
  public name: string
  constructor() {
    this.name = '张三'
  }
}
```

> 打印如下内容

```
{} name
```

#### 方法装饰器

```ts
const doc: MethodDecorator = (
  param: any,
  key: string | symbol,
  descriptor: any
) => {
  console.log(param, key, descriptor)
}

class MyDoc {
  private name: string
  constructor() {
    this.name = '张三'
  }
  @doc
  getName() {
    return this.name
  }
}
```

> descriptor是一个描述符,这里暂写为any类型,打印内容如下

```
{} getName {
  value: [Function: getName],
  writable: true,
  enumerable: false,
  configurable: true
}
```

#### 参数装饰器

```ts
const doc: ParameterDecorator = (
  param: any,
  key: string | symbol | undefined,
  index: number
) => {
  console.log(param, key, index)
}

class MyDoc {
  private name: string
  constructor() {
    this.name = '张三'
  }

  getName(name: string, @doc age: number) {}
}
```

> 参数分别代表构造函数、方法名、参数索引(从0开始),打印如下

```
{} getName 1
```

#### 装饰器使用案例

> 使用axios、装饰器实现请求发送

```sh
npm install axios -S
```

```ts
import axios from 'axios'

const Get = () => {}

class SendReq{
  constructor(){}
  @Get('http://xxx.com/xxx/list')
  getList(){

  }
}
```

> 我们希望`@Get('http://xxx.com/xxx/list')`这样传入url来发起请求，但是装饰器默认会传入原型对象，所以不支持自定义参数
>
> 所以这时候需要使用到装饰器工厂，[查看资料](https://www.tslang.cn/docs/handbook/decorators.html)

如果我们要定制一个修饰器如何应用到一个声明上，我们得写一个装饰器工厂函数。 *装饰器工厂*就是一个简单的函数，它返回一个表达式，以供装饰器在运行时调用。

我们可以通过下面的方式来写一个装饰器工厂函数：

```ts
function color(value: string) { // 这是一个装饰器工厂
    return function (target) { //  这是装饰器
        // do something with "target" and "value"...
    }
}
```

> 注意  下面[方法装饰器](https://www.tslang.cn/docs/handbook/decorators.html#method-decorators)里有一个更加详细的例子。

下面是一个方法装饰器（`@enumerable`）的例子，应用于`Greeter`类的方法上

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }

    @enumerable(false)
    greet() {
        return "Hello, " + this.greeting;
    }
}
```

我们可以用下面的函数声明来定义`@enumerable`装饰器：

```ts
function enumerable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.enumerable = value;
    };
}
```

这里的`@enumerable(false)`是一个[装饰器工厂](https://www.tslang.cn/docs/handbook/decorators.html#decorator-factories)。 当装饰器 `@enumerable(false)`被调用时，它会修改属性描述符的`enumerable`属性。

> 回到我们的主题，继续编写代码

```jsx
import axios from 'axios'

const Get = (url: string) => {
  return (
    target: Object,
    key: string | symbol,
    descriptor: PropertyDescriptor
  ) => {
    const fnc = descriptor.value
    axios
      .get(url)
      .then((res) => {
        fnc(res, {
          status: 200,
          success: true,
        })
      })
      .catch((e) => {
        fnc(e, {
          status: 500,
          success: false,
        })
      })
  }
}

class SendReq {
  constructor() {}
  @Get('https://haokan.baidu.com/haokan/ui-web/subactauthor?num=5')
  getList(res: any, status: any) {
    console.log(res.data, status)
  }
}
```

`Get`这里就是用到了`装饰器工厂`，`return`出去的就是真正的装饰器逻辑

当装饰器应用于 `getList` 方法时，TypeScript 会执行如下操作：

1. **调用 `Get` 函数**：首先，`Get` 函数被调用，传入装饰器使用的参数（在这里是 `'https://haokan.baidu.com/haokan/ui-web/subactauthor?num=5'`）。
2. **返回装饰器逻辑**：`Get` 函数返回一个接收 `target`、`key`、`descriptor` 参数的匿名函数，这是装饰器的具体实现逻辑。
3. **执行装饰器逻辑**：TypeScript 编译器自动调用这个返回的函数，实际上执行了装饰器内部的逻辑，包括设置axios的GET请求等。

这里的`fnc`或者说`descriptor.value`指的是`getList`方法，当axios发起请求后，就会执行`getList`的代码，且接收到的参数就是`then`里传入的参数

Post方法同理，这里不再演示

## NestJs学习

### 配置

> 安装

```sh
npm install -g @nestjs/cli
```

> 创建项目

```sh
nest new nest-demo
```

#### 目录结构

1. main.ts  入口文件
2. app.module.ts  根模块用于处理其他类的引用与共享
3. app.controller.ts  常见功能是用来处理http请求以及调用service层的处理方法
4. app.service.ts  封装通用的业务逻辑、与数据层的交互(例如数据库)、 其他额外的一-些三方请求

> package.json

```json
"start:dev": "nest start --watch"
```

代表代码修改后自动重启，运行`npm run start:dev`即可

### 常用命令

```sh
# 查看帮助文档
nest --help

# 生成模块
nest g controller demo
# 简写
nest g co demo

# 生成module
nest g mo demo

# 生成service
nest g s demo

# 生成以上一套
nest g resource demo
```

### 版本控制

> 比如`http://localhost:10020/v1/user/1`代表v1版本

**main.ts**

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { VersioningType } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // 配置跨域
  app.enableCors();
  app.enableVersioning({
    type: VersioningType.URI,
  });
  await app.listen(10020);
}
bootstrap();
```

> 这样就开启了版本控制，主要代码是`app.enableVersioning`，里面接收一个`VersioningType`枚举类型

#### 类

开启完之后，再controller使用版本控制

```ts
@Controller({
  path: 'user',
  version: '1',
})
export class UserController {
	// 代码省略
}
```

> 在加上`version: '1'`之后，即可重新请求`http://localhost:10020/v1/user/1`（注意这里请求路径会自动添加一个`v`，即`1 -> v1`）

#### 方法

上面是在整个类上添加版本控制，其实也可以在单个方法上实现版本控制

```ts
@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get(':id')
  @Version('1')
  findOne(@Param('id') id: string) {
    return this.userService.findOne(+id);
  }
}
```

### 请求

#### Get

##### 问号传参

> 传入query参数 `/user?name=张三`
>
> `@Request()`可以简写为`@Req()`

```ts
@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  findOne(@Request() req) {
    console.log(req.query);

    return {
      code: 200,
      data: req.query.name,
    };
  }
}
```

> 使用`@Query()`简写

```ts
@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  findOne(@Query() req) {
    console.log(req);

    return {
      code: 200,
      data: req.name,
    };
  }
}
```

##### 路径传参

> `/user/{id}`

```ts
  @Get(':id')
  findOne(@Req() id) {
    console.log(id.params);

    return {
      code: 200,
      data: id.params,
    };
  }
```

> 返回的响应如下

```json
{
    "code": 200,
    "data": {
        "id": "123"
    }
}
```

> 可简写成`@Param()`或者`@Param('id')`   (注意这里没有s)

#### Post

> 请求传入Body的参数如下

```json
{
    "id":1,
    "name":"张三"
}
```



```ts
@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  findOne(@Request() req) {
    console.log(req.body);

    return {
      code: 200,
      data: req.body,
    };
  }
}
```

> 使用`@Body()`简写

```ts
@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  findOne(@Body() req) {
    console.log(req);

    return {
      code: 200,
      data: req,
    };
  }
}
```

> 也可以只读取`name`属性
>
> `@Body('name')`

#### Headers

```ts
  @Get()
  findOne(@Query('name') name, @Headers('token') token) {
    console.log(token);

    return {
      code: 200,
      data: name,
      token: token,
    };
  }
```

#### HttpCode

```ts
  @Get()
  @HttpCode(500)
  findOne(@Query('name') name, @Headers('token') token) {
    console.log(token);

    return {
      code: 200,
      data: name,
      token: token,
    };
  }
```

### Session

> 安装依赖

```sh
npm i express-session --save

# 代码提示
npm i @types/express-session -D
```

> 在main.ts中引入

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { VersioningType } from '@nestjs/common';
// 引入
import * as session from 'express-session';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // 配置跨域
  app.enableCors();
  app.enableVersioning({
    type: VersioningType.URI,
  });
  // 使用
  app.use(
    session({
      // 签名
      secret: 'Tong',
      // 在每次请求时强行设置cookie,这将重置cookie过期时间(默认:false)
      resave: true,
      // 生成客户端cookie的名字,默认connect.sid
      name: 'tong.sid',
      cookie: {
        // 过期时间,毫秒级,7天
        maxAge: 1000 * 60 * 60 * 24 * 7,
      },
    }),
  );
  await app.listen(10020);
}
bootstrap();
```

#### 案例实现

> 验证码

nest项目安装验证码对应依赖

```sh
npm i svg-captcha -S
```

> 编写nest代码

```ts
import {
  Controller,
  Get,
  Post,
  Body,
  Req,
  Res,
  Session,
} from '@nestjs/common';
import { UserService } from './user.service';
// 引入
import * as svgCaptcha from 'svg-captcha';

@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get('code/create')
  createCode(@Req() req, @Res() res, @Session() session) {
    const captcha = svgCaptcha.create({
      // 生成验证码位数
      size: 4,
      // 文字大小
      fontSize: 50,
      // 宽度
      width: 100,
      // 高度
      height: 34,
      // 背景颜色
      background: '#cc9966',
    });
    // 将生成的验证码存入session
    console.log(captcha.text);
    session.code = captcha.text;
    res.type('image/svg+xml');
    res.send(captcha.data);
  }

  // 校验验证码
  @Post('code/check')
  checkCode(@Body() body, @Session() session) {
    // 检验用户名、密码，判断用户是否存在
    // 此时没有使用到数据库，所以暂时省略该步骤
    if (body.code === session.code) {
      return {
        code: 200,
        msg: '验证码正确',
      };
    } else {
      return {
        code: 400,
        msg: '验证码错误',
      };
    }
  }
}
```

> 我这里后端端口是`10020`，请求`http://localhost:10020/user/code/create`即可获得验证码图片

前端vue项目配置`vite.config.ts`文件

```ts
import { fileURLToPath, URL } from 'node:url'

import { defineConfig, loadEnv } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'
import vueDevTools from 'vite-plugin-vue-devtools'

export default defineConfig(({ command, mode }) => {
  //获取各种环境下的对应的变量
  const env = loadEnv(mode, process.cwd())

  return {
    plugins: [vue(), vueJsx(), vueDevTools()],
    server: {
      port: 10010,
      // 设置代理，根据我们项目实际情况配置
      proxy: {
        [env.VITE_APP_BASE_API]: {
          //apiTest是自行设置的请求前缀，按照这个来匹配请求，有这个字段的请求，就会进到代理来
          // 需要代理的域名
          target: env.VITE_URL,
          // 是否跨域
          changeOrigin: true,
          //重写匹配的字段，如果不需要放在请求路径上，可以重写为""
          rewrite: (path) => path.replace('/api', '')
        }
      }
    },
    resolve: {
      alias: {
        '@': fileURLToPath(new URL('./src', import.meta.url))
      }
    }
  }
})
```

> `.env.local`文件如果不存在，则在根目录下新建

```properties
VITE_APP_BASE_API=/api
VITE_PORT=10020
VITE_URL=http://localhost:$VITE_PORT
```

> 前端表单页面实现验证码功能

```vue
<template>
  <el-form
    label-width="auto"
    :model="loginForm"
    style="max-width: 600px"
  >
    <el-form-item label="账号">
      <el-input v-model="loginForm.username"  style="width: 300px;"/>
    </el-form-item>
    <el-form-item label="密码">
      <el-input v-model="loginForm.password"  style="width: 300px;"/>
    </el-form-item>
    <el-form-item label="验证码">
      <el-input v-model="loginForm.code" style="width: 100px;" />
      <img :src="codeUrl" @click="getCode" style="margin-left: 30px;">
    </el-form-item>
    <el-form-item>
      <el-button @click="submit">登录</el-button>
    </el-form-item>
  </el-form>
</template>

<script lang="ts" setup>
import { ElMessage } from 'element-plus'
import { reactive, ref } from 'vue'

// 后端url
const url=import.meta.env.VITE_APP_BASE_API

const codeUrl=ref<string>(url+'/user/code/create');

const loginForm = reactive({
  username: '',
  password: '',
  code: '',
})

// 重新请求验证码
const getCode=()=>codeUrl.value=codeUrl.value+'?'+Math.random();

const submit=()=>{
  fetch(url+'/user/code/check',{
    method:'POST',
    body:JSON.stringify(loginForm),
    headers:{
      'Content-Type':'application/json'
    }
  }).then(res=>res.json()).then((res)=>{
    if(res.code===200){
      // 验证码正确
      ElMessage.success(res.msg)
    }else{
      // 验证码错误
      ElMessage.error(res.msg)
      // 重新请求验证码
      getCode()
    }
  })
}
</script>
```

### 提供者

#### 基本使用

使用`@Injectable()`修饰类

```ts
@Injectable()
export class UserService {
}
```

通过`Module`模块注入到`providers`

```ts
@Module({
  controllers: [UserController],
  providers: [UserService],
})
export class UserModule {}
```

然后就可以在controller里使用这个`UserService`服务了

```ts
@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}
}
```

#### 自定义名称

```ts
import { Module } from '@nestjs/common';
import { UserService } from './user.service';
import { UserController } from './user.controller';

@Module({
  controllers: [UserController],
  providers: [
    {
      // 自定义名称
      provide: 'userService1',
      useClass: UserService,
    },
  ],
})
export class UserModule {}
```

在controller中通过`@Inject('userService1')`来识别

```ts
@Controller('user')
export class UserController {
  constructor(
    @Inject('userService1') private readonly userService: UserService,
  ) {}
}
```

#### 自定义注入值

> 使用方法如下

```ts
import { Module } from '@nestjs/common';
import { UserService } from './user.service';
import { UserController } from './user.controller';

@Module({
  controllers: [UserController],
  providers: [
    {
      // 自定义名称
      provide: 'userService1',
      useClass: UserService,
    },
    {
      // 自定义值
      provide: 'shopService',
      useValue: ['PDD', 'JD', 'TaoBao'],
    },
  ],
})
export class UserModule {}
```

```ts
@Controller('user')
export class UserController {
  constructor(
    @Inject('userService1') private readonly userService: UserService,
    // 注入
    @Inject('shopService') private readonly shopService: string[],
  ) {}

  @Get()
  showShopList() {
    // 使用
    return this.shopService;
  }
}
```

#### 工厂模式

> 简单使用

```ts
import { Module } from '@nestjs/common';
import { UserService } from './user.service';
import { UserController } from './user.controller';

@Module({
  controllers: [UserController],
  providers: [
    {
      // 自定义名称
      provide: 'userService1',
      useClass: UserService,
    },
    {
      // 自定义值
      provide: 'shopService',
      useValue: ['PDD', 'JD', 'TaoBao'],
    },
    {
      // 工厂模式
      provide: 'testService1',
      useFactory() {
        // 在这里可以写一些业务逻辑
        let flag = true;
        if (flag) {
          return 'yes';
        } else {
          return 'no';
        }
      },
    },
  ],
})
export class UserModule {}
```

```ts
@Controller('user')
export class UserController {
  constructor(
    @Inject('testService1') private readonly testService1: string[],
  ) {}

  @Get()
  showShopList() {
    return this.testService1;
  }
}
```

> 如果这个工厂仅仅编写一些逻辑已经不太够了，它依赖于其他service，参考如下写法

新建一个service，`user.service2.ts`

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class UserService2 {
  sayHello(): string {
    return 'Hello';
  }
}
```

```ts
import { Module } from '@nestjs/common';
import { UserService } from './user.service';
import { UserController } from './user.controller';
import { UserService2 } from './user.service2';

@Module({
  controllers: [UserController],
  providers: [
    UserService2,
    {
      // 工厂模式
      provide: 'testService1',
      // 注入
      inject: [UserService2],
      // 接收到的参数
      useFactory(userService2: UserService2) {
        // 调用其他service
        let str = userService2.sayHello();
        return str + ',tong';
      },
    },
  ],
})
export class UserModule {}
```

```ts
@Controller('user')
export class UserController {
  constructor(
    @Inject('testService1') private readonly testService1: string[],
  ) {}

  @Get()
  showShopList() {
    return this.testService1;
  }
}
```

#### 异步

```ts
import { Module } from '@nestjs/common';
import { UserService } from './user.service';
import { UserController } from './user.controller';
import { UserService2 } from './user.service2';

@Module({
  controllers: [UserController],
  providers: [
    UserService2,
    {
      // 工厂模式
      provide: 'testService1',
      inject: [UserService2],
      async useFactory(userService2: UserService2) {
        return await new Promise((r) => {
          setTimeout(() => {
            r(userService2.sayHello());
          }, 3000);
        });
      },
    },
  ],
})
export class UserModule {}
```

### 模块

#### 基本用法

> 创建两个模块

```sh
# 创建list模块
nest g res list

# 创建user模块
nest g res user
```

> 此时在`app.module.ts`下已经将这两个模块自动引入了

```ts
@Module({
  imports: [UserModule, ListModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

> 如果我希望在`AppController`下使用`UserService`模块

```ts
import { UserService } from './user/user.service';

@Controller()
export class AppController {
  constructor(
    private readonly appService: AppService,
    private readonly userService: UserService,
  ) {}

  @Get()
  getHello(): string {
    return this.userService.test();
  }
}
```

还没完，不然会报错找不到`UserService`，因为它现在并不是一个共享模块，它现在只能在自己的`User`模块里使用，所以还需进行配置

```ts
@Module({
  controllers: [UserController],
  providers: [UserService],
  // 导出模块，设置为共享模块
  exports: [UserService],
})
export class UserModule {}
```

#### 全局模块

将模块注册到全局，下发到下面的子模块使用

> src下新建`config/config.module.ts`，并使用`@Global`装饰器将模块标记为全局模块

```ts
import { Global, Module } from '@nestjs/common';

// 使用@Global装饰器将模块标记为全局模块
@Global()
@Module({
  // 注入
  providers: [
    {
      provide: 'configModule',
      useValue: {
        baseUrl: '/api',
      },
    },
  ],
  // 导出
  exports: [
    {
      provide: 'configModule',
      useValue: {
        baseUrl: '/api',
      },
    },
  ],
})
export class ConfigModule {}
```

> 然后在`app.module.ts`注册该模块

```ts
// 引入
import { ConfigModule } from './config/config.module';

@Module({
  imports: [UserModule, ListModule, ConfigModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

> 在controller里使用

```ts
@Controller()
export class AppController {
  constructor(
    private readonly appService: AppService,
    private readonly userService: UserService,
    @Inject('configModule') private readonly configModule: ConfigModule,
  ) {}

  @Get()
  getHello(): ConfigModule {
    return this.configModule;
  }
}
```

#### 动态模块

> 比如说我希望给`ConfigModule`模块传入参数，但现在是不支持的，代码如下

```ts
@Module({
  // 不支持传参
  imports: [UserModule, ListModule, ConfigModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

可以使用静态方法来实现

> 动态模块可以配合`@Module`结合来学习，这里就直接把`Module`装饰器里的代码copy到return里了
>
> return要求需要有一个`module`属性，名字与class保持一致即可，所以是`module: ConfigModule`

```ts
import { DynamicModule, Global, Module } from '@nestjs/common';

interface Options {
  path: string;
}

// 使用@Global装饰器将模块标记为全局模块
@Global()
@Module({})
export class ConfigModule {
  // 定义形参
  static initModule(options: Options): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: 'configModule',
          useValue: {
            // 使用参数
            baseUrl: `/api${options.path}`,
          },
        },
      ],
      exports: [
        {
          provide: 'configModule',
          useValue: {
            // 使用参数
            baseUrl: `/api${options.path}`,
          },
        },
      ],
    };
  }
}
```

> 在`AppModule`里传入参数

```ts
@Module({
  imports: [
    UserModule,
    ListModule,
    // 传参
    ConfigModule.initModule({
      path: '/tong',
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

### 中间件

#### 基本使用

中间件是在路由处理程序之前调用的函数。中间件函数可以访问请求和响应对象

感觉有些像是`vue`里的路由守卫

```sh
# 查看文档
nest --help

      ┌───────────────┬─────────────┬──────────────────────────────────────────────┐
      │ name          │ alias       │ description                                  │
      │ application   │ application │ Generate a new application workspace         │
      │ class         │ cl          │ Generate a new class                         │
      │ configuration │ config      │ Generate a CLI configuration file            │
      │ controller    │ co          │ Generate a controller declaration            │
      │ decorator     │ d           │ Generate a custom decorator                  │
      │ filter        │ f           │ Generate a filter declaration                │
      │ gateway       │ ga          │ Generate a gateway declaration               │
      │ guard         │ gu          │ Generate a guard declaration                 │
      │ interceptor   │ itc         │ Generate an interceptor declaration          │
      │ interface     │ itf         │ Generate an interface                        │
      │ library       │ lib         │ Generate a new library within a monorepo     │
      │ middleware    │ mi          │ Generate a middleware declaration            │
      │ module        │ mo          │ Generate a module declaration                │
      │ pipe          │ pi          │ Generate a pipe declaration                  │
      │ provider      │ pr          │ Generate a provider declaration              │
      │ resolver      │ r           │ Generate a GraphQL resolver declaration      │
      │ resource      │ res         │ Generate a new CRUD resource                 │
      │ service       │ s           │ Generate a service declaration               │
      │ sub-app       │ app         │ Generate a new application within a monorepo │
      └───────────────┴─────────────┴──────────────────────────────────────────────┘

# 选择中间件，middleware，生成
nest g mi logger
```

> 但是我们接下来不使用命令创建模板
>
> src下新建`middleware/index.ts`

```ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { log } from 'console';
import { NextFunction, Request, Response } from 'express';

@Injectable()
export class Logger implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    log('hello');
    next();
  }
}
```

`app.module.ts`

> 实现了`NestModule`接口中的`configure`方法，并且指定了`Logger`生效的路由是`/`

```ts
import { MiddlewareConsumer, Module, NestModule } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { UserModule } from './user/user.module';
import { ListModule } from './list/list.module';
import { ConfigModule } from './config/config.module';
import { Logger } from './middleware';

@Module({
  imports: [
    UserModule,
    ListModule,
    // 传参
    ConfigModule.initModule({
      path: '/tong',
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(Logger).forRoutes('/');
  }
}
```

> 将`middleware/index.ts`的`next`注释掉之后就会挂起
>
> 也可以直接通过`response`来拦截
>
> 可以通过这种特性实现白名单的效果

```ts
@Injectable()
export class Logger implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    log('hello');
    res.send('已被拦截');
    // next();
  }
}
```

> 指定路由除了使用字符串的形式之外，还有其他两种形式

```ts
import {RequestMethod} from '@nestjs/common';

@Module({
  imports: [
    UserModule,
    ListModule,
    // 传参
    ConfigModule.initModule({
      path: '/tong',
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(Logger)
      .forRoutes({ path: 'list', method: RequestMethod.GET });
  }
}
```

或者直接写控制器即可

```ts
@Module({
  imports: [
    UserModule,
    ListModule,
    // 传参
    ConfigModule.initModule({
      path: '/tong',
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(Logger).forRoutes(UserController);
  }
}
```

#### 全局中间件

全局中间件需要在`main.ts`里注册，而且必须是函数，不能是类

`main.ts`

```ts
function globalMiddleware(req: Request, res: Response, next: NextFunction) {
  log(req.originalUrl);
  // 实现白名单
  const whiteList = ['/user'];
  if (whiteList.includes(req.originalUrl)) {
    next();
  } else {
    res.send('没有权限');
  }
}

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // 使用
  app.use(globalMiddleware);
  await app.listen(10020);
}
bootstrap();
```

#### 第三方中间件

安装

```sh
npm install cors

npm install @types/cors -D
```

> 使用

```ts
import * as cors from 'cors';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // 使用
  app.use(cors());
  await app.listen(10020);
}
bootstrap();
```

### 上传文件

#### 安装依赖

```sh
npm install multer -S

npm install @types/multer -D

# 生成一个upload目录
nest g res upload
```

#### 基本使用

> 配置`upload.module.ts`模块
>
> 下方使用到的多个nodejs里的函数，比如`join`、`extname`

```ts
import { Module } from '@nestjs/common';
import { UploadService } from './upload.service';
import { UploadController } from './upload.controller';
import { MulterModule } from '@nestjs/platform-express';
import { diskStorage } from 'multer';
import { extname, join } from 'path';

@Module({
  imports: [
    MulterModule.register({
      // 配置图片存放路径
      storage: diskStorage({
        // 配置路径，这里使用join拼接
        destination: join(__dirname, '../images'),
        // request暂时用不到，就使用_表示不获取
        filename: (_, file, callback) => {
          // 拼接时间戳，使用extname函数截取后缀
          const filename =
            `${new Date().getTime()}` + extname(file.originalname);
          return callback(null, filename);
        },
      }),
    }),
  ],
  controllers: [UploadController],
  providers: [UploadService],
})
export class UploadModule {}
```

> 上传文件，编写`upload.controller.ts`

```ts
import {
  Controller,
  Post,
  UseInterceptors,
  UploadedFile,
} from '@nestjs/common';
import { UploadService } from './upload.service';
import { FileInterceptor, FilesInterceptor } from '@nestjs/platform-express';
import { log } from 'console';

@Controller('upload')
export class UploadController {
  constructor(private readonly uploadService: UploadService) {}

  @Post('image')
  // 方法装饰器处理文件
  // 上传单个文件是FileInterceptor，多个是FilesInterceptor
  // 字段名称是file
  @UseInterceptors(FileInterceptor('file'))
  // 使用参数装饰器读取文件
  upload(@UploadedFile() file) {
    log('file:', file);
    return {
      code: 200,
    };
  }
}
```

在使用`apifox`或者`postman`发送请求后，`dist/images`下就已经存在了该文件

![image-20240714210707281](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412212047568.png)

> 访问文件
>
> 现在是无法访问到这张图片的，因为它是一个静态资源，所以还需要配置一个静态资源的访问目录
>
> 编写`main.ts`

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { VersioningType } from '@nestjs/common';
import { NestExpressApplication } from '@nestjs/platform-express';
import { join } from 'path';

async function bootstrap() {
  // 导入NestExpressApplication类型支持,否则没有对应的代码提示
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  // 配置静态资源访问目录
  app.useStaticAssets(join(__dirname, 'images'));
  await app.listen(10020);
}
bootstrap();
```

> 访问`http://localhost:10020/1720962299048.png`即可访问到该图片
>
> 但如果我们希望在前面加一个自定义路径

```ts
async function bootstrap() {
  // 导入NestExpressApplication类型支持,否则没有对应的代码提示
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  // 配置静态资源访问目录
  app.useStaticAssets(join(__dirname, 'images'), {
    // 配置前缀
    prefix: '/files',
  });
  await app.listen(10020);
}
bootstrap();
```

> 访问`http://localhost:10020/files/1720962299048.png`即可

### 下载文件

1. download写法
2. 文件流

先演示`download`写法

```ts
@Controller('upload')
export class UploadController {
  constructor(private readonly uploadService: UploadService) {}

  @Get('export')
  download(@Res() res) {
    // 暂时写死地址，正常应该存储在数据库中
    const url = join(__dirname, '../images/1720962299048.png');
    res.download(url);
  }
}
```

前端代码

```vue
<template>
  <el-button @click='download'>下载</el-button>
</template>

<script lang="ts" setup>

const download=()=>{
  window.open('http://localhost:10020/upload/export')
}
</script>
```

再看一下流的方式

```sh
# 安装依赖
npm install compressing -S
```

```ts
import { zip } from 'compressing';
import { Response } from 'express';

@Controller('upload')
export class UploadController {
  constructor(private readonly uploadService: UploadService) {}

  @Get('stream')
  async stream(@Res() res: Response) {
    const filename = '1720962299048.png';
    const url = join(__dirname, '../images', '/', filename);
    const tarStream = new zip.Stream();
    // 由于是异步的，所以需要等待处理完成再返回，否则返回的文件流是有问题的
    await tarStream.addEntry(url);
    // 设置流的格式
    res.setHeader('Content-Type', 'application/octet-stream');
    // 设置文件名称
    res.setHeader('Content-Disposition', `attachment; filename=${filename}`);
    //在此处开放Content-Disposition权限，前端代码才能获取到
    res.setHeader('Access-Control-Expose-Headers', 'Content-Disposition');
    tarStream.pipe(res);
  }
}
```

> 前端怎么解析流文件

前端代码

```vue
<template>
  <el-button @click='download'>下载</el-button>
</template>

<script lang="ts" setup>
// 如果使用的是axios，那么responseType需要设置为ArrayBuffer或者Blob，而不再是json了
// 下面使用fetch进行演示

const useFetch=async (url:string)=>{
  const res=await fetch(url)
  // 获取请求头里的content-disposition的内容
  const header=res.headers.get('content-disposition') as string
  // 获取文件名
  const filename=header.split('filename=')[1]
  const arrayBuffer =await res.arrayBuffer()
  const blob=new Blob([arrayBuffer])
  const fileUrl=URL.createObjectURL(blob)
  // 创建a标签
  const a=document.createElement('a')
  a.href=fileUrl
  // 这是一个zip包,下载
  a.download=`${filename}.zip`
  a.click()  
}

const download=()=>{
  useFetch('http://localhost:10020/upload/stream')
}
</script>
```

### RxJs

RxJs使用的是观察者模式，用来编写异步队列和事件处理。

Observable 可观察的物件
Subscription 监听Observable
Operators 纯函数可以处理管道的数据 如 map filter concat reduce 等

感觉用法像lambda表达式

> 这里新建一个空的`demo`目录，使用命令初始化

```sh
npm install ts-node -g

npm install rxjs
```

#### 基本使用

新建`index.ts`

```ts
import { Observable } from 'rxjs'

// 接收一个回调函数
const observable = new Observable((subscribe) => {
  subscribe.next(1)
  subscribe.next(2)
  subscribe.next(3)
  setTimeout(() => {
    subscribe.next(4)
    // 完成
    subscribe.complete()
  }, 2000)
})

// 接收一个对象,对象里存在一个next函数
observable.subscribe({
  next: (num) => {
    console.log(num)
  },
})
```

运行

```sh
ts-node index.ts
```

#### 常用api

```ts
import { of, Observable, interval, take } from 'rxjs'
import { map, filter, findIndex, reduce } from 'rxjs/operators'

// 将产生一个序列，每500毫秒发射一次值。初始值为0，之后每次发射的值比前一次多1
interval(500)
  // pipe是管道,到5的时候停止
  .pipe(take(5))
  .subscribe((e) => {
    console.log(e)
  })
```

打印结果如下

```
0
1
2
3
4
```

> map

```ts
import { of, Observable, interval, take } from 'rxjs'
import { map, filter, findIndex, reduce } from 'rxjs/operators'

interval(500)
  .pipe(
    // 封装成一个对象
    map((v) => ({
      msg: '这是一个对象',
      data: v,
    }))
  )
  .subscribe((e) => {
    console.log(e)
  })
```

现在是无限打印如下内容

```
{ msg: '这是一个对象', data: 0 }
{ msg: '这是一个对象', data: 1 }
{ msg: '这是一个对象', data: 2 }
{ msg: '这是一个对象', data: 3 }
{ msg: '这是一个对象', data: 4 }
{ msg: '这是一个对象', data: 5 }
{ msg: '这是一个对象', data: 6 }
{ msg: '这是一个对象', data: 7 }
{ msg: '这是一个对象', data: 8 }
{ msg: '这是一个对象', data: 9 }
{ msg: '这是一个对象', data: 10 }
```

我们需要做一个停止

```ts
import { of, Observable, interval, take } from 'rxjs'
import { map, filter, findIndex, reduce } from 'rxjs/operators'

// 用一个变量接收
const subs = interval(500)
  .pipe(
    map((v) => ({
      msg: '这是一个对象',
      data: v,
    }))
  )
  .subscribe((e) => {
    console.log(e)
    if (e.data === 5) {
      subs.unsubscribe()
    }
  })
```

> filter

```ts
import { of, Observable, interval, take } from 'rxjs'
import { map, filter, findIndex, reduce } from 'rxjs/operators'

const subs = interval(500)
  .pipe(
    map((v) => ({
      msg: '这是一个对象',
      data: v,
    })),
    // 过滤偶数
    filter((v) => v.data % 2 === 0)
  )
  .subscribe((e) => {
    console.log(e)
    if (e.data >= 5) {
      subs.unsubscribe()
    }
  })

```

打印如下

```
{ msg: '这是一个对象', data: 0 }
{ msg: '这是一个对象', data: 2 }
{ msg: '这是一个对象', data: 4 }
{ msg: '这是一个对象', data: 6 }
```

> of
>
> 我们希望使用自定义数据，而不是`interval`

```ts
const subs = of(1, 2, 3, 4, 5, 6)
  .pipe(
    map((v) => ({
      msg: '这是一个对象',
      data: v,
    })),
    // 过滤偶数
    filter((v) => v.data % 2 !== 0)
  )
  .subscribe((e) => {
    console.log(e)
  })
```

> retry

```ts
import { of, Observable, interval, take, retry } from 'rxjs'
import { map, filter, findIndex, reduce } from 'rxjs/operators'

// 假设of是一个后台接口
const subs = of(1, 2, 3, 4, 5, 6)
  .pipe(
    // 如果后台接口报错了,会进行失败重试3次
    retry(3),
    map((v) => ({
      msg: '这是一个对象',
      data: v,
    })),
    filter((v) => v.data % 2 !== 0)
  )
  .subscribe((e) => {
    console.log(e)
  })
```

### 响应拦截器

> 自动生成的模块都是自动返回一个字符串，而我们希望返回给前端的是一个`json`数据。现在需要对响应统一处理
>
> 在`src`下新建`common/response.ts`

response.ts

```ts
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from '@nestjs/common';
import { Observable, map } from 'rxjs';

interface Data<T> {
  data: T;
}

@Injectable()
export class Response<T> implements NestInterceptor {
  intercept(
    context: ExecutionContext,
    next: CallHandler<any>,
  ): Observable<Data<T>> {
    return next.handle().pipe(
      map((data) => {
        return {
          data,
          success: true,
          code: 200,
          message: '成功',
        };
      }),
    );
  }
}
```

main.ts

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { VersioningType } from '@nestjs/common';
import { NestExpressApplication } from '@nestjs/platform-express';
import { join } from 'path';
import * as cros from 'cors';
// 引入
import { Response } from './common/response';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.useStaticAssets(join(__dirname, 'images'), {
    prefix: '/files',
  });
  app.use(cros());
  // 注入响应拦截器
  app.useGlobalInterceptors(new Response());
  await app.listen(10020);
}
bootstrap();
```

### 异常拦截器

>在`src`下新建`common/filter.ts`

```ts
import {
  ArgumentsHost,
  Catch,
  ExceptionFilter,
  HttpException,
} from '@nestjs/common';
import { Request, Response } from 'express';

@Catch()
export class HttpFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest<Request>();
    const response = ctx.getResponse<Response>();
    const status = exception.getStatus();
    response.status(status).json({
      success: false,
      time: new Date(),
      data: exception.message,
      status,
      // 需要知道哪个接口报的错误
      path: request.url,
    });
  }
}
```

> main.ts

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { VersioningType } from '@nestjs/common';
import { NestExpressApplication } from '@nestjs/platform-express';
import { join } from 'path';
import * as cros from 'cors';
// 引入
import { HttpFilter } from './common/filter';
import { Response } from './common/response';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.useStaticAssets(join(__dirname, 'images'), {
    prefix: '/files',
  });
  app.use(cros());
  app.useGlobalInterceptors(new Response());
  // 注入异常拦截器
  app.useGlobalFilters(new HttpFilter());
  await app.listen(10020);
}
bootstrap();
```

> 随便访问一个不存在的url即可查看到404的效果

### 管道转换

管道可以做两件事

1. 转换，可以将前端传入的数据转成成我们需要的数据
2. 验证类似于前端的rules配置验证规则

#### 类型转换

```ts
@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get(':id')
  showShopList(@Param('id') id: string) {
    log(typeof id);
    return this.userService.findAll();
  }
}
```

> 访问`http://localhost:10020/user/123`，打印的id类型为`string`，我们希望转成`number`

```ts
import {ParseIntPipe} from '@nestjs/common';

@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get(':id')
  // 使用ParseIntPipe转成number
  showShopList(@Param('id', ParseIntPipe) id: string) {
    log(typeof id);
    return this.userService.findAll();
  }
}
```

> 常用的还可以转`Float`、`Int`、`Bool`、`Array`、`UUID`

#### 验证

> 执行以下命令

```sh
# 创建login模块
nest g res login

# 创建login管道
nest g pi login

# 安装校验库
npm install --save class-validator class-transformer
```

login.pipe.ts

```ts
import { ArgumentMetadata, Injectable, PipeTransform } from '@nestjs/common';
import { log } from 'console';

@Injectable()
export class LoginPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    log('value:', value, 'metadata:', metadata);
    return value;
  }
}
```

login.controller.ts

```ts
// 导入管道
import { LoginPipe } from './login.pipe';

@Controller('login')
export class LoginController {
  constructor(private readonly loginService: LoginService) {}

  @Post()
  // 使用管道
  create(@Body(LoginPipe) createLoginDto: CreateLoginDto) {
    return this.loginService.create(createLoginDto);
  }
}
```

> post请求携带的body参数为

```json
{
    "name":"张三",
    "age":23
}
```

> 打印的内容如下

```
value: { name: '张三', age: 23 } metadata: { metatype: [class CreateLoginDto], type: 'body', data: undefined }
```

> 这个`data: undefined`是啥
>
> 其实就是下面代码的`name`

```ts
create(@Body('name',LoginPipe) createLoginDto: CreateLoginDto)
```

> 打印内容如下

```
value: 张三 metadata: { metatype: [class CreateLoginDto], type: 'body', data: 'name' }
```

了解到以上内容以后，开始编写校验代码

> 模块中自动生成的`dto`目录下`create-login.dto.ts`

```ts
import { IsNotEmpty, IsNumber, Length } from 'class-validator';

export class CreateLoginDto {
  @IsNotEmpty({
    message: '名字不能为空',
  })
  @Length(5, 10, {
    message: '名字的长度应该在5-10之间',
  })
  name: string;
  @IsNumber(
    {},
    {
      message: '年龄必须是数字',
    },
  )
  age: number;
}
```

login.controller.ts

```ts
@Controller('login')
export class LoginController {
  constructor(private readonly loginService: LoginService) {}

  @Post()
  // 使用管道
  create(@Body(LoginPipe) createLoginDto: CreateLoginDto) {
    return this.loginService.create(createLoginDto);
  }
}
```

login.pipe.ts

> 使用`plainToInstance`反射创建对象

```ts
import { ArgumentMetadata, Injectable, PipeTransform } from '@nestjs/common';
import { plainToInstance } from 'class-transformer';
import { log } from 'console';

@Injectable()
export class LoginPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    // 反射创建对象
    const dto = plainToInstance(metadata.metatype, value);
    log('dto:', dto);
    return value;
  }
}
```

打印如下

```
dto: CreateLoginDto { name: '张三', age: 23 }
```

>`validate`返回的是`promise`，所以加一个`async`和`await`

```ts
import { validate } from 'class-validator';

@Injectable()
export class LoginPipe implements PipeTransform {
  async transform(value: any, metadata: ArgumentMetadata) {
    // 反射创建对象
    const dto = plainToInstance(metadata.metatype, value);
    // 校验dto
    const errors = await validate(dto);
    log('errors:', errors);
    return value;
  }
}
```

> 打印内容如下

```
errors: [
  ValidationError {
    target: CreateLoginDto { name: '张三', age: 23 },
    value: '张三',
    property: 'name',
    children: [],
    constraints: { isLength: '名字的长度应该在5-10之间' }
  }
]
```

> 如果数据啥也不传

```
errors: [
  ValidationError {
    target: CreateLoginDto {},
    value: undefined,
    property: 'name',
    children: [],
    constraints: { isLength: '名字的长度应该在5-10之间', isNotEmpty: '名字不能为空' }
  },
  ValidationError {
    target: CreateLoginDto {},
    value: undefined,
    property: 'age',
    children: [],
    constraints: { isNumber: '年龄必须是数字' }
  }
]
```

这里可以选择使用`管道`来格式化一下错误信息

> 返回错误信息

```ts
@Injectable()
export class LoginPipe implements PipeTransform {
  async transform(value: any, metadata: ArgumentMetadata) {
    // 反射创建对象
    const dto = plainToInstance(metadata.metatype, value);
    // 校验dto
    const errors = await validate(dto);
    if (errors.length > 0) {
      throw new HttpException(errors, HttpStatus.BAD_REQUEST);
    }
    return value;
  }
}
```

> 如果返回的错误信息不对，请查看是否配置了全局的`异常拦截器`

除了以上方法之外，还可以使用nest自带的校验方法

```tsx
@Controller('login')
export class LoginController {
  constructor(private readonly loginService: LoginService) {}

  @Post()
  // 不再需要使用管道
  create(@Body() createLoginDto: CreateLoginDto) {
    return this.loginService.create(createLoginDto);
  }
}
```

> dto照样如上配置即可，无需改动

main.ts

```ts
// 引入
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.useStaticAssets(join(__dirname, 'images'), {
    prefix: '/files',
  });
  app.use(cros());
  app.useGlobalInterceptors(new Response());
  app.useGlobalFilters(new HttpFilter());
  // 使用
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(10020);
}
bootstrap();
```

### 爬虫

```sh
# 类似于JQuery，操作dom
npm install cheerio -S

npm install axios -S

nest g res reptile
```

> 目标`https://love.19lou.com/recommend?sex=0`

```ts
import {
  Controller,
  Get,
  Post,
  Body,
  Patch,
  Param,
  Delete,
} from '@nestjs/common';
import { ReptileService } from './reptile.service';
import { CreateReptileDto } from './dto/create-reptile.dto';
import { UpdateReptileDto } from './dto/update-reptile.dto';
import axios from 'axios';
import { log } from 'console';
import * as cheerio from 'cheerio';
import * as fs from 'fs';
import * as path from 'path';

@Controller('reptile')
export class ReptileController {
  constructor(private readonly reptileService: ReptileService) {}

  baseUrl: string = 'https://love.19lou.com/recommend?sex=0';

  @Get()
  async findAll() {
    const downloadUrls: string[] = [];

    const getImgUrls = async (url: string) => {
      // 获取到html内容
      const body = await axios.get(url).then((res) => res.data);
      const $ = cheerio.load(body);
      $('.list-main .love-item img').each(function () {
        downloadUrls.push($(this).attr('src'));
      });
    };
    // 获取到图片链接
    await getImgUrls(this.baseUrl);
    // 下载图片
    this.write(downloadUrls);
    return downloadUrls;
  }

  /**
   * 下载文件
   * @param urls 链接
   */
  write(urls: string[]) {
    urls.forEach(async (url) => {
      const buffer = await axios
        .get(url, { responseType: 'arraybuffer' })
        .then((res) => res.data);
      const ws = fs.createWriteStream(
        path.join(__dirname, '../images', new Date().getTime() + '.jpg'),
      );
      ws.write(buffer);
    });
  }
}
```

> 请求`http://localhost:10020/reptile`即可

### 守卫

守卫有一个单独的责任。
它们根据运行时出现的某些条件(例如权限、角色、访问控制列表等)来确定给定的请求是否由路由处理程序理。
这通常称为授权。在传统的Express应用程序中，通常由中间件处理授权(以及认证)。
中间件是身份验证的良好选择，因为诸如token验证或添加属性到request对象上与特定路由(及元数据)没有强关联。

**`守卫在每个中间件之后执行,但在任何拦截器或管道之前执行。`**

创建一个守卫

```sh
nest g gu [name]

nest g res guard

cd src/guard

# 在src下新建guard模块后
nest g gu role
```

#### 局部使用

role.guard.ts

```ts
@Injectable()
export class RoleGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    log('经过了守卫');
    return true;
  }
}
```

guard.controller.ts

> 通过`@UseGuards`装饰器来使用守卫

```ts
import {
  Controller,
  Get,
  UseGuards,
} from '@nestjs/common';
// 引入守卫
import { RoleGuard } from './role/role.guard';

@Controller('guard')
// 使用
@UseGuards(RoleGuard)
export class GuardController {
  constructor(private readonly guardService: GuardService) {}

  @Get()
  findAll() {
    return this.guardService.findAll();
  }
}
```

#### 全局使用

main.ts

```ts
// 引入
import { RoleGuard } from './guard/role/role.guard';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  // 使用
  app.useGlobalGuards(new RoleGuard());
  await app.listen(10020);
}
bootstrap();
```

> 接下来还是以`局部使用`为例

```ts
@Controller('guard')
@UseGuards(RoleGuard)
export class GuardController {
  constructor(private readonly guardService: GuardService) {}

  @Get()
  // 设置元信息
  @SetMetadata('roles', ['admin'])
  findAll() {
    return this.guardService.findAll();
  }
}
```

role.guard.ts

```ts
@Injectable()
export class RoleGuard implements CanActivate {
  // 注入Reflector
  constructor(private readonly reflector: Reflector) {}
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    // roles需要与controller的key对应上
    // 使用context.getHandler()来获取函数体
    const admin = this.reflector.get<string[]>('roles', context.getHandler());
    log('经过了守卫:', admin);
    return true;
  }
}
```

打印如下

```
经过了守卫: [ 'admin' ]
```

已经获取到需要的角色了
这里可以去数据库查询是否具有相应角色，但由于还没有学习到`orm`框架，暂时从请求url里获取角色
`http://localhost:10020/guard?roles=admin`

```ts
@Injectable()
export class RoleGuard implements CanActivate {
  // 注入Reflector
  constructor(private readonly reflector: Reflector) {}
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    // roles需要与controller的key对应上
    // 使用context.getHandler()来获取函数体
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const req = context.switchToHttp().getRequest<Request>();
    log('经过了守卫:', req.query.roles, '需要角色:', roles);
    if (roles.includes(req.query.roles as string)) {
      return true;
    }
    return false;
  }
}
```

### 自定义装饰器

> 将上一节使用的`@SetMetadata('roles', ['admin'])`封装成自定义装饰器

```sh
# 建立装饰器
nest g d role
```

```ts
export const Role = (...args: string[]) => SetMetadata('role', args);
```

> 引入使用

```ts
@Controller('guard')
@UseGuards(RoleGuard)
export class GuardController {
  constructor(private readonly guardService: GuardService) {}

  @Get()
  @Role('admin')
  findAll() {
    return this.guardService.findAll();
  }
}
```

接下来自定义封装一个参数装饰器

role.decorator.ts

```ts
import {
  ExecutionContext,
  SetMetadata,
  createParamDecorator,
} from '@nestjs/common';
import { log } from 'console';
import { Request } from 'express';

export const Role = (...args: string[]) => SetMetadata('role', args);

export const ReqUrl = createParamDecorator(
  (data: string, context: ExecutionContext) => {
    const request = context.switchToHttp().getRequest<Request>();
    log('data:', data);
    return request.url;
  },
);
```

```ts
@Controller('guard')
@UseGuards(RoleGuard)
export class GuardController {
  constructor(private readonly guardService: GuardService) {}

  @Get()
  @Role('admin')
  // 使用参数装饰器
  findAll(@ReqUrl('url:test') url: string) {
    log('url:', url);
    return this.guardService.findAll();
  }
}
```

> 打印如下

```ts
data: url:test
url: /guard?roles=123
```

这里的`data`指`@ReqUrl`里的内容

### swagger接口文档

```sh
# 安装依赖
npm i @nestjs/swagger swagger-ui-express
```

main.ts

```ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.useStaticAssets(join(__dirname, 'images'), {
    prefix: '/files',
  });
  app.use(cros());
  app.useGlobalInterceptors(new Response());
  app.useGlobalFilters(new HttpFilter());
  app.useGlobalPipes(new ValidationPipe());
  // 设置接口文档信息
  const options = new DocumentBuilder()
    // 设置JWT权限校验
    .addBearerAuth()
    .setTitle('xxx后台管理系统')
    .setDescription('后台系统使用nestJS、React构建')
    .setVersion('1')
    .build();
  const document = SwaggerModule.createDocument(app, options);
  // 接口文档路径(http://localhost:10020/api-docs)
  SwaggerModule.setup('api-docs', app, document);
  await app.listen(10020);
}
bootstrap();
```

> 但是这样看起来很多很乱，我们需要对接口做`分组`

```ts
import {
  ApiBearerAuth,
  ApiOperation,
  ApiParam,
  ApiQuery,
  ApiResponse,
  ApiTags,
} from '@nestjs/swagger';

// 权限校验
@ApiBearerAuth()
@Controller('guard')
@UseGuards(RoleGuard)
// 用于分组
@ApiTags('守卫接口')
export class GuardController {
  constructor(private readonly guardService: GuardService) {}

  @Post()
  // 对参数createLoginDto进行描述,@ApiProperty装饰器写在`create-guard.dto.ts`文件里
  create(@Body() createLoginDto: CreateGuardDto) {
    return this.guardService.create(createLoginDto);
  }

  @Get()
  @Role('admin')
  // @ApiQuery和@ApiParam类似的，区别是@ApiQuery是请求参数，@ApiParam是路径参数
  // 设置参数信息
  @ApiParam({
    name: 'url',
    description: '这是当前访问的url',
    required: true,
    type: 'string',
  })
  // 返回状态描述
  @ApiResponse({ status: 403, description: '没有权限' })
  // 描述和详细描述
  @ApiOperation({ summary: 'get接口', description: '主要作用是xxx' })
  findAll(@ReqUrl('url:test') url: string) {
    log('url:', url);
    return this.guardService.findAll();
  }
}
```

create-guard.dto.ts

```ts
import { ApiProperty } from '@nestjs/swagger';

export class CreateGuardDto {
  // 对Body参数进行描述
  @ApiProperty({ example: '张三', enum: ['李四', '王五'], type: 'string' })
  name: string;
  @ApiProperty({ example: 23 })
  age: number;
}
```



### 连接数据库

> orm框架这里使用`typeOrm`

```sh
npm install --save @nestjs/typeorm typeorm mysql2
```

配置app.module.ts

```ts
@Module({
  imports: [
    UserModule,
    TypeOrmModule.forRoot({
      //数据库类型
      type: 'mysql',
      //账号
      username: 'root',
      //密码
      password: '123456',
      //host
      host: 'localhost',
      //端口
      port: 3306,
      //库名
      database: 'book',
      //实体文件
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      //synchronize字段代表是否自动将实体类同步到数据库
      synchronize: true,
      //重试连接数据库间隔
      retryDelay: 500,
      //重试连接数据库的次数
      retryAttempts: 10,
      //如果为true,将自动加载实体 forFeature()方法注册的每个实体都将自动添加到配置对象的实体数组中
      autoLoadEntities: true,
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(Logger).forRoutes(UserController);
  }
}
```

> 推介使用自动加载实体`autoLoadEntities: true`

user.entity.ts

```ts
import { Column, Entity, PrimaryGeneratedColumn } from 'typeorm';

@Entity({ name: 'user' })
export class User {
  // 主键自增
  @PrimaryGeneratedColumn('increment')
  id: number;
  @Column()
  username: string;
  @Column()
  password: string;
  @Column()
  nickName: string;
}
```

create-user.dto.ts

```ts
export class CreateUserDto {
  username: string;
  password: string;
  nickName: string;
}
```

update-user.dto.ts

```ts
import { PartialType } from '@nestjs/mapped-types';
import { CreateUserDto } from './create-user.dto';

export class UpdateUserDto extends PartialType(CreateUserDto) {
  id: number;
  userrname: string;
  password: string;
  nickName: string;
}
```

user.module.ts

```ts
import { User } from './entities/user.entity';

@Module({
  // 关联
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UserController],
  providers: [UserService],
  exports: [UserService],
})
export class UserModule {}
```

### CRUD

> 这里使用`React`和`NestJS`实现一个`CRUD`

![image-20240718234211256](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412212047227.png)

[前端代码](https://www.123pan.com/s/tMU0Vv-OL6Ud.html)

后端代码

> user.controller.ts

```tsx
@ApiTags('用户接口')
@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  findOne(@Query('username') username: string) {
    return this.userService.findOneByUsername(username);
  }

  @Get('list')
  list() {
    return this.userService.findAll();
  }

  @Post()
  saveUser(@Body() createUser: CreateUserDto) {
    return this.userService.create(createUser);
  }

  @Put()
  update(@Body() updateUser: UpdateUserDto) {
    return this.userService.update(updateUser);
  }

  @Delete(':id')
  removeOne(@Param('id', ParseIntPipe) id: number) {
    return this.userService.remove(id);
  }
}
```

>user.service.ts

```tsx
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User) private readonly userMapper: Repository<User>,
  ) {}

  create(createUserDto: CreateUserDto) {
    const user = new User();
    user.username = createUserDto.username;
    user.password = createUserDto.password;
    user.nickName = createUserDto.nickName;
    return this.userMapper.save(user);
  }
  findAll() {
    return this.userMapper.find();
  }

  findOneByUsername(username: string) {
    return this.userMapper.find({
      where: {
        username: Like(`%${username}%`),
      },
    });
  }

  update(updateUserDto: UpdateUserDto) {
    return this.userMapper.update(updateUserDto.id, updateUserDto);
  }

  remove(id: number) {
    return this.userMapper.delete(id);
  }
}
```

