# 概述

> TypeScript是微软公司开发的用于代替Javascript的一门编程语言，它以Javascript为基础构建，扩展了JavaScript。Typescript可以在任何支持Javascript的环境中运行，但是Typescript不能被Javascript引擎直接解析，它在运行前需要编译为JavaScript代码

- 相比js增加了什么
  1. 变量可以确定类型，且增加了一些数据类型
  2. 增加了接口，抽象类等新特性
  3. 具有丰富的配置选项，可以配置语言语法的松散与严格程度

> 注意:
>
> - TS是完全兼容JS的，所以可以再TS代码中任意书写JS代码

## TypeScript编译器

> Typescript解析器依赖于Node.js环境，可以通过包管理工具安装

- **安装编译器**

```
npm install typescript -g
```

- **编译ts文件**

```
tsc ts文件
```

### 编译选项

> 在使用命令编译ts文件时，通过命令的选项配置编译动作

- --init

  > 在当前目录下初始化tsc的配置文件,名为tsconfig.json

  ```
  tsc --init
  ```

- -w

  > 在编译时加入-w选项，编译器会监视ts文件的变化，实时编译。

  ```
  tsc app.ts -w
  ```

  > 注意:
  >
  > - 此时只能实时编译某一个ts文件,如果需要实时编译某个文件夹下的所有ts文件，需要在当前文件夹加入tsc配置文件，然后执行命令`tsc -w`，不指定文件路径，会编译当前目录下的所有文件

## 常用tsconfig.json配置选项

> 正常情况下，json文件中无法写注释，但是tsconfig.json中可以写注释

- include

  > 默认tsc会编译目录下的所有ts文件，可以通过include选项控制编译的文件

  ```
  {
  "include": ["./src/**/*"]
  }
  ```

  > 注意:
  >
  > - `**`匹配任意目录,`*`匹配任意文件
  > - 参数值接收一个数组
  > - 默认值为`"**/*"`

- exclude

  > 不编译哪些文件，配置同include,默认值为`["node_modules","bower_components","jspm_packages"]`

- extends

  > 定义被继承的配置文件,继承某个配置文件的配置

  ```
  {"extends":"./configs/base"}
  ```

- files

  > 指定被编译文件的列表，作用同include

  ```
  {"files":[
  "core.ts",
  "sys.ts"
  ]}
  ```

- compilerOptions

  > compilerOptions中具有许多子选项，这些子选项可以配置编译器的行为。

  - target

    > 指定ts被编译为js的语法版本

    ```
    {"complierOptions":"target":"ES6"|"ES5"|"ESNEXT"}
    ```

  - module

    > 使用的模块化规范

    ```
    "module": "ES6"
    ```

  - lib

    > 指定项目中用到的库

  - outDir

    > 指定编译后文件的生成目录

  - outFile

    > 将编译的所有ts文件合并为一个js文件

  - allowJs

    > 是否对js文件进行编译，默认为false

  - checkJs

    > 是否检查js文件中的语法，默认为false

  - removeComments

    > 是否编译后移除注释，默认为false

  - noEmit

    > 是否生成编译后软件，默认为false

  - noEmitOnError

    > 有错误时是否编译文件，默认为false

  - alwaysStrict
  
    > 开启js严格模式，默认为true
  
  - noImplicitAny
  
    > 不允许隐式any
  
  - noImplicitThis
  
    > 不允许不明确类型的this
  
  - strictNullChecks
  
    > 严格检查空值
  
  - strict
  
    > 开启所有严格检查，默认为false

# 类型声明

> JS是一个动态类型的语言，动态类型虽然使用方便，但是会带来极大的安全隐患；TS相比于JS最大的优点，就是加入了类型声明，可以为变量指定类型，使得TS成为了一个静态类型的语言。

- **声明变量并指定变量类型**

  ```
  let a: number
  声明关键字 变量名 : 变量类型
  ```

  > 注意:
  >
  > - 如果在声明变量的同时对变量进行初始化，如`let a = true`,此时会触发ts编译器的类型推断机制，即使没有显式指定变量的类型，仍然会根据赋值的类型自动推断出变量类型，相当于`let a:boolean = true`。

  

- **声明函数参数与返回值类型**

  ```
  function sum(参数1:参数类型,参数2:参数类型):返回值类型 {
  	return 
  }
  ```


## 数据类型

| 类型    | 例子            | 描述                                                     |
| ------- | --------------- | -------------------------------------------------------- |
| number  | 1,-33,2.5       | 任意数字                                                 |
| string  | 'hi',  "hi", hi | 任意字符串                                               |
| boolean | true、  false   | 布尔值true或false                                        |
| 字面量  | 其本身          | 限制变量的值就是该字面量的值                             |
| any     | *               | 任意类型                                                 |
| unknown | *               | 类型安全的any                                            |
| void    | 空值(undefined) | 没有值(或undefined),常用来表示函数没有返回值             |
| never   | 没有值          | 不能是任何值，常用来表示函数没有返回值                   |
| object  | {name:'孙悟空'} | 任意的JS对象                                             |
| array   | [1,2,3]         | 任意JS数组,通过数据类型[]声明或Array<数据类型>           |
| tuple   | [4,5]           | 元素,TS新增类型,固定长度的数组, 通过[数据类型······]声明 |
| enum    | enum{A,  B}     | 枚举  ,TS中新增类型                                      |

> 注意:
>
> - 声明类型时，不仅可以使用参数类型的关键字，还可以使用对应类型的**字面量**,如`leta:10`,此时该变量的值被限制为字面量值，不是字面量的类型。
> - 类型声明还可以使用逻辑表达式为变量指定多个合法值,`|`表示或，`&`表示且，如`let a:'male'|'female'`，称为联合类型。&常用于连接两个对象，如`let a : {name : string}& {age:number}`,表示a需要同时具有两个对象的属性
> - any类型表示任意类型，如果显示为变量加上any类型，则称为显式any；如果没有给变量声明类型，解析器会自动为变量加入any类型，称为隐式any
> - 如果不确定一个变量的类型，可以指定类型为unknown，与any的区别是:any类型的变量可以赋值给任何其他变量，但是unknown类型不能直接给其他类型赋值(any除外）。如果想要使用unknown给其他类型赋值，必须先对unknown类型的变量做类型检查或者类型断言,否则会报错
> - void可以接收函数的返回值为`null`,`undefined`或无返回值,never不能有返回值
> - 一般不使用object，而是使用字面量的形式指定对象的属性，如:`let a : {name:string,age?:number,[propName:string]:any}`,属性名后加`?`表示属性可选,中括号表示此时可以有任意个属性，中括号中的值表示属性名必须是string类型，any表示属性值可以是any。
> - 在ts中，函数也是对象，如果变量是一个函数，可以通过箭头函数的方式指定函数的结构，如`let a : (a:number,b:string) => number`
> - 枚举需要自己定义枚举类，通过enum{枚举属性名}，然后讲枚举类赋值给需要的变量，此时变量就可以赋值为枚举类中的属性，枚举类不需要写属性值，默认值从0递增

```typescript
let a : unknown
a = 'hello'
let b : string
if (typeof a === 'string')  {b = a }//类型检查
s = e as string //类型断言
s = <string> e //类型断言
```

- 类型别名

  > 可以自定义联合属性，通过type关键字为其起别名方便多次使用

  ```
  type myType = string | number
  ```


# 打包工具

## webpack

> webpack打包ts代码

```
npm install -D webpack webpack-cli typescript ts-loader
```

- 配置文件

  > 默认名字为webpack.config.js

  ```
  const path = require('path')
  //webpackz中的所有配置信息都需要通过module.exports导出
  module.exports = {
      //指定入口文件
      entry:'./src/index.ts',
      //指定打包文件目录
      output: {
          path: path.resolve(__dirname,'dist'),
          filename:'bundle.js'
      },
      //需要打包的模块
      module: {
          //指定加载的规则
          rules: [
              {   //表示规则匹配的文件，可以使用正则表达式匹配
                  test:/\.ts$/,
                  //使用的loader
                  use:'ts-loader',
                  exclude:/node-modules/
          }]
      }
  
  }
  ```

> 配置完成后，可以使用webpack命令进行打包，也可以配置成npm脚本

- htmlcss资源相关插件

  > 会自动创建html页面，并将js文件引入

  ```
  npm i -D html-webpack-plugin
  ```

  - 使用

    > 在webpack配置文件中配置插件

    ```
    const HTMLWebpackPlugin = require('html-webpack-plugin')
    module.exports = {
        plugins: [new HTMLWebpackPlugin(
        { 	//设置生成网页的标题
        	title: '你好', 
        	//设置生成网页的模板
        	template: './src/index.html'}
        )]
    }
    ```

- 服务器插件

  > 启动命令为webpack serve  --open chrome.exe

  ```
  npm i -D webpack-dev-server
  ```

- 清空插件

  > 使用之前需要在webpack中配置，方法同html插件

  ```
  npm i -D clean-webpack-plugin
  ```

## babel

> 可以进行语法转换，增强代码在浏览器的兼容性

```
npm i -D @babel/core @babel/preset-env babel-loader core-js
```



# 小知识

## JS严格模式

> 开启严格模式后，Js运行性能更好，通过在js文件开头加入"use strict"开启，如果使用了es6模块，则默认开启严格模式

## js问号运算符

```
box?.addEventListener()
```

