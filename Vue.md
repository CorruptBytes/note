# 概述

> Vue 是一套用于构架用户界面的渐进式框架

- 特点

1. 免除原生JavaScript中的DOM操作，简化书写‘
2. 基于MVVM(Model-View-ViewModel)思想，实现数据的双向绑定，将编程的关注点放在数据上（响应式）。

## MVVM模型

> View是视图层，Model是数据模型，当数据模型发生变化时，视图层会自动更新展示数据；当视图层数据发生变化时，数据模型中的数据也会自动更新，这就是双向绑定。其中，视图层与数据模型直接的连接是通过ViewModel实现的

![edac5999-384d-430c-93ac-b90019bf9981](D:\笔记\图片\edac5999-384d-430c-93ac-b90019bf9981.png)

## Vue组成

> Vue渐进式的特点便体现在其组成，Vue的核心开发只需要使用Vue核心包即可实现，和其他插件完全独立，可以渐进式的学习引入其他插件进行开发

![image-20250422213757246](D:\笔记\图片\image-20250422213757246.png)

> 基于Vue渐进式的特点，Vue有两种开发方式:
>
> ①**直接引入 Vue 核心库**：基于已有的 HTML、CSS、JavaScript 文件，在项目局部按需引入 Vue，为部分模块增加动态交互能力，适合渐进式改造。
>
> ②**基于构建工具的工程化开发**：借助构建工具（如 Webpack、Vite 等），整合各种插件和自动化流程，采用单文件组件（`.vue`）形式进行模块化开发，实现完整的工程化管理。

- Vue核心包

> Vue核心包具有开发版和生产板两个版本，开发版包含完整的警告和调试模式，生产板删除了警告

```html
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
<!-- 生产环境版本，优化了尺寸和速度 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2"></script>
```

> 也可也在官网下载核心包通过文件路径引入

## 插值表达式

> Vue中的模板语法，用于在页面中输出动态数据。Vue 会自动解析其中的表达式，并将结果渲染到对应的位置。

```
{{表达式}}
```

> 注意:
>
> - 在插值表达式中，可以直接通过属性名访问Vue实例中通过配置项`data`挂载的数据
> - 插值表达式只能用于文本内容中，标签属性中不能使用插值表达式
> - 表达式就是一切可以被求值的代码，如变量，三元运算符，数组，函数等
> - 在插值表达式中填入一个html标签，渲染引擎并不会解析标签，会将其当成字符串处理

# Vue实例

> Vue实例是Vue中的核心对象，用来管理数据、处理逻辑、控制页面视图的更新与交互，是整个 Vue 应用的起点和“大脑”。

- Vue实例创建

```javascript
//引入Vue核心包后，可以获得Vue的构造函数
const app = new Vue({
    el:'',
    data:{},
    methods:{},
    ...
})
```

> 注意:
>
> - 传入的对象参数叫做配置对象，也叫做 选项对象（Options Object），这个对象定义了Vue实例的挂载点、数据、方法、模板、生命周期等核心选项。

## Vue实例配置对象常用属性

> 配置对象是创建 Vue 实例时传入的参数对象，它的属性定义了一系列初始化实例的选项。通过这些选项，可以指定 Vue 实例的挂载目标、初始数据、操作方法、生命周期函数等功能配置。

- el

> el属性定义该Vue实例的挂载点，Vue实例将会管理这个元素，属性值为CSS选择器,通过该CSS选择器选择挂载点

```javascript
el:'#app'
```

- data

> `data`用于定义响应式数据。在 `data` 属性中定义响应式数据时，可以将数据组织为一个对象，并通过该对象的属性来定义所需的数据。

```javascript
data: {
	msg:'Hello,World!'
}
```

> 注意:
>
> - data对象的属性会被直接挂载到Vue实例上，所以在实例中可以通过this直接访问数据
> - 在组件中，data必须是一个函数，通过返回值定义数据，返回值使用一个对象封装组装所需数据

- methods

> 在methods中定义方法处理事件或业务逻辑。属性值为一个对象，对象中的每个属性对应一个方法

```javascript
methods : {
	fn : function() {

		}
}
```

- computed(计算属性)

> 基于现有的数据，提供一个计算逻辑，计算出一个新的属性。数据变化，属性值自动变化

```
//简写
computed : {
	计算属性名(){
		求值逻辑
		return 结果
	}
}

/完整写法
computed : {
	计算属性名 : {
		get() {
		求值逻辑
		return 结果
		}
		set(修改的值) {
			操作逻辑
		}
	}
}
```

> 注意:
>
> - 计算属性最终也会变成挂载到vue实例上的一个属性，属性名即为计算属性名
> - 想要在模板中使用计算属性只需要使用插值表达式包裹计算属性名
> - 计算属性简写方式只提供了访问计算属性的路径，如果想要对计算属性进行修改，需要完整写法，且set方法中可以获得所修改的值

|          |                           computed                           |             methods             |
| :------: | :----------------------------------------------------------: | :-----------------------------: |
|   作用   |              封装对数据处理的逻辑，返回一个结果              | 为vue实例提供处理业务逻辑的方法 |
|   语法   | 最终会变成一个挂载到vue实例上的属性，可以在模板中直接通过插值表达式访问或在实例方法中通过this访问 |          需要手动调用           |
| 缓存特性 | 计算属性会对计算结果进行缓存，再次使用直接读取缓存，依赖项变化后:自动重新调用计算->再次缓存 |             无缓存              |

- watch(监听器)

> 这个配置项用于监视数据变化，当数据变化时，自动执行一些业务逻辑或异步操作

```javascript
简单写法:监视简单数据类型，直接监视
//提供一个方法，方法名要与data中监视的属性名保持一致
watch : {
	数据名(newValue,oldValue) {
	
	},
	'对象.属性名'(newValue,oldValue){
	
	}
}
完整写法:添加额外配置项
watch : {
	数据属性名: {
	deep : true //深度监视，监视对象中的所有属性
	immediate : true //初始化立刻执行一次handler方法
	handler(newValue,oldValue) {
	业务逻辑
	}
	}
}
```

## 生命周期

> 生命周期是一个Vue实例从创建到销魂的整个过程分为四个阶段：

①创建

> 在此阶段Vue实例会创建并挂载定义的响应式数据。在此阶段后，可以进行发送初始化渲染请求

②挂载

> 此时Vue实例已挂载完成，开始根据模板进行数据渲染。此阶段后，浏览器元素渲染完成，可以进行dom操作

③更新

> 在此阶段Vue不断检测数据变化，更新视图，不断循环

④销毁

> 在此阶段Vue实例被销毁

![lifecycle_zh-CN.W0MNXI0C](D:\笔记\图片\lifecycle_zh-CN.W0MNXI0C.png)

### Vue生命周期函数(钩子函数)

> 在生命周期过程中，自动执行的一些函数，又被称为生命周期钩子,Vue提供了8个钩子函数供使用

![image](D:\笔记\图片\image-1745647731840.png)

> 注意:
>
> - 配置生命周期函数只要在配置项中提供同名方法即可。

# Vue指令

> Vue中带有v-前缀的特殊标签属性，Vue会根据不同的指令，针对标签实现不同的功能

- v-html

> 设置元素的innerHTML属性，内容需要为表达式

- v-show

> 通过display属性控制元素的显示与隐藏,值为以表达式，表达式结果为true，元素显示；为false，元素隐藏

```
v-show = "表达式"
```

> 注意:
>
> - v-show指令适用于频繁切换显示隐藏的情景

- v-if，v-else-if，v-else

> 与Java中的条件语句相同，通过条件判断是否创建或移除该元素(条件渲染)

```
v-if = "表达式"
```

> 注意:
>
> - v-if适用于不频繁变换的场景
> - 这三个指令需要紧挨着使用，才能构成完整的判断逻辑

- v-on

> 快速给元素注册事件

```vue
v-on:事件名 = "内联语句或methods中的函数"
//简写
@事件名 = "内联语句或methods中的函数"
```

> 注意:
>
> - 注册事件 = 添加事件监听 +  提供处理逻辑
>
> - 调用函数时，既可以写成`fn`,也可写成`fn()`，效果一样
> - 如果需要对函数传递参数，直接fn(a,b)

- v-bind

> 动态设置html元素的属性

```javascript
v-bind:属性名 = "表达式"
//简写
:属性名 = "表达式"

//操作class属性
//使用对象，如果对象属性值为true，则添加这个类名，反之不添加
<div class="box" :class="{ 类名1: 布尔值, 类名2: 布尔值 }"></div>
//使用数组，所有类名都会被添加
<div class="box" :class="[ 类名1, 类名2, 类名3 ]"></div>

//操作style样式，style = "样式对象
//css属性值要用引号引起来，属性名要使用驼峰命名或者用引号引起来
<div class="box" :style="{ CSS属性名1: CSS属性值, CSS属性名2: CSS属性值 }"></div>
```

> 注意:

- v-for

> 基于数据循环，多次渲染元素，可以遍历数组，对象，数字

```vue
v-for = "(item,index) in items" :key  = "item.id"

//如果无需使用index，简化为
v-for = "item in items" :key  = "item.id"
```

> 注意:
>
> - key属性 = "唯一标识"，给列表项添加的唯一标识。便于Vue进行列表项的正确排序复用。
>
>   如果不加key，Vue会尝试原地修改元素 ,此时会原地复用，而不是修改想要修改的元素
>
> - key 的值只能是字符串或数字类型,key 的值必须具有 唯一性, 推荐使用 id 作为 key（唯一），不推荐使用 index 作为 key（会变化，不对应）

- v-model

	> 用于对表单元素与变量双向绑定，将表单元素的值与一个变量绑定在一起，两者同步更新

	```
	v-model = "变量"
	```

	> - vue会根据表单元素类型，自动绑定表单元素对应属性
	> - 下拉框比较特殊，`<select>`元素的value，关联了其下被选中的`option`的value
	
	- v-model原理
	
	  > 本质是一个语法糖，将表单元素的值属性与表单事件和写
	
	  ```html
	  //以input举例，利用value属性的动态绑定与input事件将value值与变量双向绑定
	  <input type="text" :value="name" @input="name = $event.target.value">
	  ```
	
	  > - v-model作用在不同元素上，vue会根据元素自动绑定对应的value属性与表单时间
	  > - 在vue模板,使用$event可以获得事件传递的第一个参数，如果事件未指定传递的参数，则vue会自动将当前事件对象注入为事件的参数，此时$event可以获得当前事件对象；如果为事件指定了参数，也可以通过$event占位符，将事件对象注入

## 指令修饰符

> 通过"."指明一些指令后缀，不同后缀封装了不同的处理操作，目的是简化代码

- 按键修饰符

> 监听键盘事件的某些按键

```
//监听回车按下
@keyup.enter = "内联语句或函数"
```

- v-model修饰符

```
v-model.trim = "变量" ->去空格
v-model.number = "变量" ->转数字
```

- 事件修饰符

```
@事件名.stop   -> 阻止冒泡
@事件名.prevent -> 阻止默认行为
```

## 自定义指令

> 自定义一些指令，封装一些dom操作，扩展额外功能,使用指令时，使用`v-指令名`

- 全局注册

  ```
  //第一个参数为指令名，第二个参数为指令配置项，可以配置指令的钩子函数
  Vue.directive('指令名',{
  	inserted(){
  	el.focus
  	}
  //inserted为指令的钩子函数，意为在元素被渲染到页面后自动调用，可直接通过形参获得当前元素
  })
  ```

- 局部注册

  > 写在当前组件的vue实例配置项，只能在当前组件内使用

  ```
  directives : {
  	指令名 : {
  	inserted() {
  	
  	}
  	}
  }
  ```

- 指令的值

  > 自定义指令与官方指令相同，也可以通过`=`为指令绑定具体的参数值

  ```
  //指令值修改会触发指令的update钩子函数，钩子函数会自动注入两个形参，第一个为当前绑定的元素，第二个为binding,这是一个对象，通过binding.value可以拿到指令值，
   directives : {
      color : {
        inserted(el,binding) {
          el.style.color = bind.value
        },
        update(el,binding) {
          el.style.color = binding.value
        }
      }
    }
  
  ```

  

# Vue工程化开发

> 工程化开发是指，基于一套标准化的项目结构，结合完整的自动化工具链进行开发，通过规范化的开发流程与模块化管理，提升项目的开发效率、代码质量和维护可靠性。
> 在工程化开发过程中，通常借助构建工具（如 Webpack、Vite 等）配合各类插件进行开发。构建工具能够自动处理、编译、压缩并整合代码，将开发者编写的 Vue 组件、样式和脚本，最终打包生成浏览器可以直接识别和运行的 HTML、CSS 和 JavaScript 文件。

## Vue CLI

> Vue CLI 是官方脚手架工具，基于 Node.js 环境，帮助开发者快速搭建一个标准化、规范化的 Vue 项目基础架构(集成webpack)。

- 优点
  - 零配置
  - 标准化
  - 内置辅助开发的一些工具
- 安装

```bash
npm install @vue/cli -g
```

- 创建项目

```
vue create project-name
```

- 启动项目

```
npm run serve
```

> 注意:
>
> - 必须在工程目录中执行此命令

- 项目结构

![image (1)](D:\笔记\图片\image (1).png)

- main.js文件

> Vue项目的入口文件

```js
import Vue from 'vue'  //导入vue核心包
import App from './App.vue' //导入App.vue根组件

//配置当前处于什么环境(生产环境/开发环境),false为生产环境
Vue.config.productionTip = false

//创建Vue实例，提供render方法，基于App.vue创建结构渲染index.html
new Vue({
  render: createElement => {
      //基于组件创建HTML结构
      	return createElement(App)
  }           	         
}).$mount('#app')
```

> 注意:
>
> - 在工程化开发中，index.html中不会写模板结构，通过App.vue组件注入

## 组件化开发

> 将页面拆分为一个个组件，每个组件有自己独立的结构，样式和行为

- 优点

  > 便于维护，复用

- 组件组成

  > 组件与普通的html页面一样，包含结构，样式和行为

  ```vue
  <!-- template即模板，可以在其中使用vue模板语法，用于提供结构和实现数据渲染 -->
  <template>
    <div id="app">
      <img alt="Vue logo" src="./assets/logo.png">
      <HelloWorld msg="Welcome to Your Vue.js App"/>
    </div>
  </template>
  <!-- 行为和数据 -->
  <script>
  import HelloWorld from './components/HelloWorld.vue'
  
  export default {
    name: 'App',
    components: {
      HelloWorld
    }
  }
  </script>
  <!-- 样式 -->
  <style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
  </style>
  
  ```

  > 注意:
  >
  > - Vue组件的模板中必须有且只有一个根元素
  > - 如果需要使用less，可以在style标签内声明属性lang="less",开启less功能，并安装less和less-loader
  > - script标签中需要默认导出vue实例配置项供外部使用
  > - .vue文件又被称为单文件组件
  > - 默认style标签中的样式为全局样式，会作用于所有组件，如果想要声明局部样式，在style标签上加入scoped属性`<style scoped>`，该样式只会作用于当前组件。
  > - scoped的原理是为当前组件内的所有元素添加一个自定义属性:data-v-hash值，通过这个属性使用属性选择器区分不同组件的样式

- 组件中的data

  > 组件中的data配置项必须是一个函数，这可以保证每个组件实例，维护独立的一份数据对象

  ```
  data() {
  return {
  	count : 100,
  	age : 18,
  }
  }
  ```

  

- 根组件

  > Vue项目最顶层的组件，在它里面包含其他所需的组件，默认名为App.vue，这个组件会被自动注入index.html中

### 组件注册

> 组件需要先注册才能使用，分为局部注册与全局注册

- 局部注册

  > 在需要用到组件的组件的配置项中注册，只能在当前组件内使用

  ```javascript
  import 组件对象 from '.vue文件'
  export default {
  component: {
  	'组件名' : 组件对象
  //如果组件名与组件对象同名，不需要冒号声明
  	组件对象
  }
  }
  ```

- 全局注册

  > 在所有组件内都能使用，在main.js中进行注册，在所有组件内都能使用

  ```javascript
  import 组件对象 from '.vue文件' //导入组件
  Vue.conponent('组件名',组件对象) //注册组件
  ```

- 使用

> 当成普通的html标签使用，可以自闭合，也可也双标签

```
<组件名></组件名>
```

- 注意
  - 组件名要使用大驼峰命名或`-`分隔
  - 导入组件时，.vue文件可以不加后缀
  - 组件标签也支持使用vue指令

### 组件通信

> 就是组件与组件之间的数据传递。因为组件的数据是独立的，无法直接访问其他组件的数据，需要通过组件通信获得其他组件的数据

- 分类

  > 根据组件关系，组件通信可以分为父子关系与非父子关系

  - 父子关系

    > 借助props与$emit进行数据通信

    ①父组件通过props将数据传递给子组件

    ②子组件利用$emit通知父组件修改更新数据

    ![image (1)](D:\笔记\图片\image (1)-1745675845286.png)

    ```javascript
    //父组件传值
    
    //在父组件中通过v-bind绑定属性为子组件传值
    <div>
    		<MySon v-bind:title="title"></MySon>
    </div>
    
    //在子组件中通过props接收传递的值
    export default {
        	//props中的变量可以通过this直接访问
    		props: ['title'],
    	};
    
    //子组件修改值
    //父组件中监听子组件发出的事件，收到后调用处理函数
    <MySon v-bind:title="title" @changeTitle="changeTitle"></MySon>
    //在子组件中使用通知父组件事件触发
     methods: {
                changeTitle() {
                    //通知父组件事件触发，第一个参数为触发的事件，后续参数为父组件处理事件的方法所需参数
                    this.$emit('changeTitle','Hello,World')
                }
            }
    ```
  
  > props校验:对prop传递的值指定验证要求
    >
  > ①类型校验:props:{校验属性名:类型}
    >
  > ②非空校验③默认值④自定义校验
    >
    > props {
    >
    > 校验属性名:{
    >
    > ​	tpye : 类型，
    >
    > ​	required ：true，
    >
    > ​	default ： 默认值，
    >
    > ​	validator(value) {
    >
    > ​	return 是否通过校验	
    >
    > }
    >
    > }}
    >
    > vue遵循单向数据流，即父级prop的数据更新，会向下流动，影响子组件。这个数据流动是单项的，不能子组件更新数据，向上流动，所以不能更改prop，如果修改，要通知父组件修改
  
  - 非父子关系
  
    > 两种方案:①provide与inject②eventbus
  
  - 通用方案
  
    > 使用Vuex，适合复杂业务场景

# 插槽

> 通过插槽，可以让组件内部的一些结构根据使用场景自定义

- 使用

  1. 将组件内需要自定义的结构，改用`<slot></slot>`占位
  2. 使用组件时，在组件双标签中间传入自定义结构替换占位标签

  ```html
  子组件中:
  <div class="dialog-content">
  			<slot></slot>
  </div>
  父组件中:
  <MyDialog> <h1>您确定要退出本系统吗?</h1> </MyDialog>
  ```

  > 注意:
  >
  > - 可以向`<slot>`标签内填入内容，如果父组件不传入内容，则默认显示`<slot>`标签内的内容

## 具名插槽

> 上述为默认插槽语法，如果一个组件内有多处结构需要自定义，需要使用具名插槽。

- 使用

  1. 为多个`<slot>`标签添加name属性区分
  2. `template`配合`v-slot:名字`来分发对应标签

  ```
  <template>
  	<div class="dialog">
  		<div class="dialog-header">
  			<slot name="title"></slot>
  		</div>
  
  		<div class="dialog-content">
  			<slot name="content"></slot>
  		</div>
  		<div class="dialog-footer">
  			<slot name="footer"></slot>
  		</div>
  	</div>
  </template>
  
  
  
  <MyDialog>
  			<template v-slot:title>
  				<div>我是标题</div>
  			</template>
  			<template v-slot:content>
  				<div>我是内容</div>
  			</template>
  			<template #footer>
  				<button>点击</button>
  			</template>
  		</MyDialog>
  ```

  > 注意:
  >
  > - v-slot:名字可简写为`#插槽名`
  > - 如果不为插槽起名字，父组件传入的内容会渲染到所有插槽中

## 作用域插槽

> 作用域插槽是插槽的传参语法，不是插槽，即定义slot插槽的同时，可以给插槽绑定数据，供组件使用

- 使用

  1. 在`slot`标签上绑定属性的方式传值(可以动态绑定，也可以绑定普通属性)
  2. 所有添加的属性，都会被收集到有一个对象中
  3. 在template中，通过`#插槽名=对象名`接收数据，默认插槽名为default

  ```html
  在子组件slot标签中
  <slot msg="默认文本" :id="list.id">
      
  在父组件中:
      <MyTable :list="list">
      <template #default="obj">
          <p>
              {{obj.msg}}
          </p>
          <button @click="del(obj.id)">删除</button>
       </template>
      </MyTable>
  ```


# 路由

  

  > 路由就是一种映射关系，在Vue中，路由就是路径与组件的映射关系，通过路由，可以根据路径渲染对应的组件。在Vue中，通过VueRouter插件实现路由功能

  ## VueRouter

  > 由Vue官方提供，实现路由功能，当地址栏路径被修改时，切换匹配显示的组件。

-   使用

  > 以下操作除下载外，均在main.js中完成

  1. 下载对应Vue版本的路由插件，并将插件引入工程

     ```
     npm install vue-router@3.6.5
     
     import VueRouter from 'vue-router'
     ```

  2. 注册路由插件

     ```
     Vue.use(VueRouter)
     ```

  3. 创建路由对象

     ```
     const router = new VurRouter()
     ```

  4. 将路由对象注入到根组件Vue实例中，建立关联

     ```javascript
     new Vue({
       render: h => h(App),
       //对象名与属性名一致时，可以简写为对象名，不需写为冒号键值对形式
       router
     
     }).$mount('#app')
     ```

  5. 创建路由相关组件，并配置路由规则，即组件与路径的对应关系

     > 路由相关组件一般放到views目录下

     ```javascript
     const router = new VueRouter({
       routes : [
         {path : '/find' , component : Find},
         { path: '/friend', component: Friend },
         { path: '/My', component: My },
       ]
     })	
     ```

  6. 配置导航链接，配置路由出口(即路径匹配的组件显示的位置)

     > 路由是使用url切换显示的组件，而且路由路径前均有一个`#`标识

     ```html
     <div>
     		<div class="footer_wrap">
     			<a href="#/find">发现音乐</a>
     			<a href="#/my">我的音乐</a>
     			<a href="#/friend">朋友</a>
     		</div>
     		<div class="top">
     			<!-- 路由出口 → 匹配的组件所展示的位置 -->
     			<router-view></router-view>
     		</div>
     	</div>
     ```

-   路由封装

    > 为方便项目管理，一般将路由对象相关的配置从main.js中抽离出，新建一个router文件夹，下放index.js等路由相关文件，然后将路由对象导入到main.js中

    ```
    
    ```

-   router-link

    > 声明式导航,由vue-router提供的全局组件，用于取代a标签实现路由跳转，本质仍然是a标签
    >
    > 相比于a标签，这个组件默认提供高亮类名，可以直接用于设置高亮样式

    - 使用

      > 为router-link指定to属性来确定跳转目的地，相当于a标签的href属性,但路径不需要加属性,而且路由展示的router-link所对应的a标签上会被自动加上router-link-exact-active和router-link-active，当组件展示切换后，标签类名会自动移除与添加，可以直接通过这两个类来为router-link设置样式

      ```html
      	<router-link to="/find">发现音乐</router-link>
      	<router-link to="/my">我的音乐</router-link>
      	<router-link to="/friend">朋友</router-link>
      
      <style>
          .router-link-active {
              background-color : red;
          }
      </style>
      ```

      > 注意:
      >
      > - router-link-active类名是加在a标签上的，设置样式时注意选择器提权
      > - router-link-active用于模糊匹配路径，即既可以匹配/my,也可以匹配/my/a,/my/b，而router-link-exact-active用于精确匹配，仅可以匹配to属性对应的路径

### 路由对象配置项

> 在创建VueRouter对象时，可以传入一个配置对象，它的属性用于设置路由相关属性

- routes

  > 用于配置路径与组件的映射关系，是一个对象数组

  ```
  routes : [
  	{path : '/', redirect : '/My'},
      {path : '/find' , component : Find},
      { path: '/friend', component: Friend },
      { path: '/My', component: My },
    ]
  ```

- linkActiveClass与linkExactActiveClass

  > 配置`route-link`标签的高亮激活类名

  ```
  linkActiveClass : 'router-link-active',
  linkExactActiveClass : 'router-link-exact-active' 	
  ```

## 声明式导航跳转传参

  > 当进行路由跳转时，有时需要将一些参数传入到下一个页面， 	有两种方式传参: 	 	查询参数传参；动态路由传参

  ### 查询参数传参

  > 直接在声明式导航组件的to属性上增加查询参数

```js
//传参
<router-link to="/path?参数名=值"></router-link>

//接收参数
模板中:$route.query.参数名
js中: this.$route.query.参数名
```

### 动态路由传参

> 在配置路由路径时，将路由路径参数化

```js
//配置动态路由:路径中冒号开头的即为参数，可以动态指定，这种被称为动态路由，即可以匹配/search/xxx模式的所有路由路径
const router = new VueRouter({
  routes: [
    { path: '/home', component: Home },
      //这种配置方式表示必须传递参数；如果不传递，路由无法匹配；如果希望参数非必须，可以在参数名后加?即'/search/:words?'
    { path: '/search/:words', component: Search }
  ]
})
//配置导肮链接，直接在参数的位置填入需要传递的参数
<router-link to="/search/黑马程序员">黑马程序员</router-link>

//接收参数,配置动态路由时冒号后的变量名即为参数名
$route.params.参数名
```

- 动态路由传参与查询参数传参

  > 如果需要传递多个参数，使用查询参数传参，如果只传递单个参数，使用动态路由传参更加简洁

  ![image-20250501134427290](D:\笔记\图片\image-20250501134427290.png)

## 路由重定向

  > 当网页初次打开时，路由路径为空，无法匹配到组件，展示为空，可以通过路由重定向，将一些路径重定向到另一个路径

  ```
  {path : 路径，redirect : 重定向路径}
  ```

## 路由404

> 当路径匹配不到对应路由时，展示一个提示页面，这也是一个路由，需要配置在所有路由的最后

```
{path : '*',component : NotFind}
```

> 注意:
>
> - *表示匹配任意路径,这里的原理是路由会从上到下依次匹配，如果匹配成功则暂停并显示对应组件；如果前面均匹配失败，则会匹配到`*`

## 路由模式

> 路由默认模式为hash路由,这时候路径是带有`#`标识这是一个路由的，如果想要将路径配置为不带`#`的样式，需要使用history路由模式，两种模式的底层实现原理不同，history路由需要配置服务器提供支持

```
const router = new VueRouter({
routes,
mode : "history"
})
```

### 编程式导航

> 编程式导航即是通过js实现页面跳转，因为有时需要通过按钮实现路由跳转。有两种语法

#### path路径跳转

> 为按钮配置事件处理函数

```
简写:this.$router.push('路由路径')

完整写法:this.$router.push({
path : '路由路径'
})
```

#### name命名路由跳转

> 路由可以配置名字name属性，此时可以通过name直接跳转，这种方式适合path路径长的场景

```
{ name: 'search', path: '/search/:words?', component: Search }
 
 this.$router.push({
 name : '路由名'
 })
```

#### 编程式导航传参

> 编程式导航传参与声明式导航传参方式一致，支持查询参数传参与动态路由传参

```js
//查询参数传参
this.$router.push('/路径?参数名1=参数值&参数名2=参数值2')
//或
this.$router.push({
    path : '/路径',
    query : {
        参数名1 : '参数值1'，
        参数名2 : '参数值2'
    }
})

//动态路由传参
this.$router.push('/路径/参数值')
//或
this.$router.push({
    path : '/路径',
    params : {
        参数名 : '参数值'
    }
})
```

> 接收参数同声明式导航传参

## 嵌套路由

> 有时候路由组件中仍有其他路由，此时需要配置嵌套路由,需要配置路由的children属性，这个属性同普通路由一样，也是对象数组的格式，对象属性与普通路由一样

```js
routes : [{
path : '/',
component : Layout,
children : [
	{
	path : '/article',
	component : Article
	}
]
}]
```

> 嵌套路由需要在对应页面配置对应的不同路由出口

## 路由返回

> 通过$router.back()可以返回到当前路由前的一个路由

## 组件缓存

> 路由跳转后，原本的组件就被销毁了，之后通过back()方法回到之前的路由后，组件又被重新创建渲染，会导致之前组件中的状态数据全部清零 ，这时候 如果想要保存组件状态，可以使用keep-alive

### keep-alive

> Vue的内置组件，当它包裹动态组件时，会缓存不活动的组件实例，而不是销毁他们。
>
> 这是一个抽象组件:自身不会被渲染为DOM元素，也不会出现在父组件中

- 优点

  1. 在组建切换过程中，把切换出去的组件保存在内存中，防止重复渲染，减少加载时间及性能消耗，提高用户体验。

- 使用

  > 默认情况下，keep-alive会缓存包裹的所有组件，可以通过配置项(属性)配置需要缓存的组件

  ```
  include : 组件名数组，只有匹配的组件会被缓存
  exclude ： 组件名数组，匹配的组件不会被缓存
  max ： 最多可以缓存多少个组件实例
  ```

  > 注意:
  >
  > - 组件名是组件的name属性，如果没有配置，默认使用文件名
  > - 被缓存的组件会多两个生命周期钩子:actived(激活时)和deactived(失活时)

## 路由全局前置守卫

> 所有路由被匹配之前，都会先经过全局前置守卫，只有全局前置守卫放行，才会真正解析渲染组件

-  注册

  > 使用router.beforeEach()注册一个全局前置守卫，该方法的参数需要接收一个方法

  ```
  router.beforeEach((to,from,next) => {
  	//to，目的地路由信息对象
  	//from，出发地路由信息对象
  	//next，是一个函数，next()被调用，路由才会切换；next(路径)拦截到某个页面
  })
  ```

  ## 路由懒加载

## Vuex

> 是一个Vue的状态管理工具，状态就是数据，即vuex是一个管理vue多组件共享数据的插件

- 使用场景
  1. 某些数据需要在多个组件中使用
  2. 多个组件共同维护一份数据

- 优点

  1. 数据集中化管理
  2. 响应式变化
  3. 操作简洁(vuex提供了一些辅助函数，简化了操作)

  ![image-20250502140429186](D:\笔记\图片\image-20250502140429186.png)

## 仓库

> Vuex通过仓库管理数据,Vue中的所有组件都可以访问到仓库，直接通过this.$store可以访问到仓库，仓库是一个对象

- 创建仓库

  > 仓库相关的文件通常放在src下的store文件夹中，导出给Vue实例使用

  ```
  import Vue from 'vue'
  import Vuex from 'vuex'
  Vue.use(Vuex)
  const store = new Vuex.Store()
  export default store
  ```

- 挂载仓库

  ```
  简写:因为对象与属性名相同
  new Vue({
    render: h => h(App),
    store
  }).$mount('#app')
  
  完整写法:
  new Vue({
    render: h => h(App),
    store: store
  }).$mount('#app')
  ```

### state状态

> 所有共享的数据都会统一放到Store中的State属性中存储，可以在创建仓库时通过配置项添加数据

```
const store = new Vuex.Store({
  state: {
    count: 101
  }
})
//state更推荐的声明方法是像data一样，声明成一个函数，保证每份数据的独立性
const store = new Vuex.Store({
  state() {
  return {
  count : 100
  }
  }
})
```

- 获取数据

  - 通过store直接访问

    ```
    this.$store或者通过import直接导入仓库
    模板中 : $store.state.xxx
    组件逻辑中 : this.$store.state.xxx
    JS模块中 : store.state.xxx
    
    ```

  - 辅助函数访问

    > mapState辅助函数，可以直接将store中的数据映射为组件的计算属性

    1. 导入mapState

       ```
       import {mapState} from 'vuex'
       ```

    2. 数组方式引入state

       ```
       mapState(['count','title'])
       ```

    3. 展开运算符映射

       > 此时得到的是对象，里面封装了计算属性，需要使用展开运算符将其展开

       ```
       computed : {
       	...mapState(['count','title'])
       }
       ```

- 修改数据

  > vuex同样遵循单向数据流，组件中不能直接修改仓库数据，但是默认语法检测不开启，所以直接修改数据不会报错
  >
  > 修改store中数据的正确方法是使用mutations，将组件请求提交给仓库修改

  - 定义mutations对象，，对象中存放修改state的方法

    > mutations通过store的配置项定义，mutations是一个对象，mutations对象中方法的第一个形参均为state，可以通过它获得数据

    ```
    const store = new Vuex.Store({
      state: {
        count: 101
      },
      mutations: {
        addCount (state, n) {
          state.count += n
        }
      }
    })
    ```

  - 组件调用mutations

    > commit的第一个参数为调用的方法，后面的参数为这个被调用的方法的参数，被称为提交载荷,只能提交一个参数，如果需要传递多个参数，需要包装成一个对象传递

    ```
    this.$store.commit('addCount', 10)
    ```

  - mapMutations

    > 这是一个辅助函数，把位于mutations中的方法提取出来，映射到组件的methods中

    ```js
    methods: {
    	...mapMutations(['addCount'])
    }
    //等价于
    methods: {
    	addCount(n) {
    	this.$store.commit('addCount',n)
    	}
    }
    ```

  - 开启语法检查

    > 通过仓库的配置项修改,开启语法检查会消耗性能，所以vue默认关闭

    ```
    strict : true
    ```

- actions

  > mutations中的操作必须是同步的，这样便于监测数据变化，记录调试，vuex专门使用actions处理异步操作

  - 定义

    > 通过store的配置项actions配置异步操作函数,在actions中定义一个异步操作，在异步操作中调用mutations中的同步操作修改值

    ```
    actions : {
    	setAsyncCount(context, num) {
    	setTimeout(() => {
    	context.commit('changeCount',num)
    	})
    	}
    }
    ```

  - 使用

    ```
    $store.dispatch('seAsyncCount',200)
    ```

  - mapActions

    > 将actions中的方法提取出来，映射到组件methods中

    ```
    import {mapActions} from 'vuex'
    
    methods : {
    	...mapActions(['setAsyncCount'])
    }
    等价于
    methods : {
    setAsyncCount(n) {
    	this.$store.dispatch('setAsyncCount',n)
    }
    }
    ```

  - 调用

    ```
    this.$store.dispatch('setAsyncCount',200)
    ```

- getters

  > 类似计算属性，但是这个计算属性是依赖于state中的状态

  - 定义

    ```js
    getters: {
    //getters属性中的函数第一个参数是state，必须有返回值
    filter (state) {
    	return state.list.filter(item => item > 5)
    }
    }
    ```

  - 使用

    - 通过store直接访问

      ```js
      $store.getters.filter
      ```

    - 通过辅助函数mapGetters映射

      ```
      computed: {
      	...mapGetters(['filter'])
      }
      ```

## 模块(module)

> 由于vuex使用单一状态树，应用的所有状态都会集中到一个比较大的对象上。当应用变得越来越复杂后，store对象会变得很臃肿，难以维护。此时可以将store拆分为一个个模块，单独维护。然后将模块导出到store中使用

- 定义模块

  > 在store文件夹下创建modules文件夹，用来存放模块相关代码，每一个模块都是一个js文件

  ```
  const state = {}
  const mutations = {}
  const actions = {}
  const getters = {}
  export default {
    state,
    mutations,
    actions,
    getters
  }
  ```

- 挂载模块

  > 在store配置项中挂载

  ```
  import user from '@/store/modules/user.js'
  modules: {
  	user,
  }
  ```

- 访问模块

  > 即使分成模块，子模块的状态仍然会挂载到根级别的state中，属性名就是模块名

  - 直接通过模块名访问

    ```js
    $store.state.模块名.xxx
    ```

  - 通过mapState映射

    > 映射到组件的computed中，需要使用展开运算符展开

    ```
    默认根级别的映射: mapState(['xxx'])
    子模块的映射(需要开启命名空间): mapState('模块名'，['xxx'])
    开启命名空间:在子模块导出时，添加一个配置项:namespaced: true
    ```

  - 访问模块中的getters

    > 因为模块中的getters挂载时存在特殊字符，所以不能直接使用.访问

    ```
    直接通过模块名访问:$store.getters['模块名'/xxx]
    通过mapGetters映射:
    默认根级别映射:mapGetters(['xxx'])
    子模块的映射(需要开启命名空间):mapGetters('模块名'，['xxx'])
    ```

  - 访问模块中的mutations与actions

    > 默认模块中的mutations与actions会被挂载到全局，需要开启命名空间，才会挂载到子模块

    ```
    直接通过store调用: 
    $store.commit('模块名/xxx',额外参数)
    通过mapMutations映射:
    默认根级别的映射 mapMutations(['xxx'])
    子模块的映射(需要开启命名空间):mapMutations('模块名',['xxx'])
    
    直接通过store调用: 
    $store.dispatch('模块名/xxx',额外参数)
    通过mapActions映射:
    默认根级别的映射 mapActions(['xxx'])
    子模块的映射(需要开启命名空间):mapActions('模块名',['xxx'])
    ```

    

# 小知识

## 模板渲染

> 模板渲染（Template Rendering）是 Vue.js 使用模板（HTML 和 Vue 特定语法）来定义视图的过程。Vue 通过解析模板并将其与数据绑定，自动管理视图的更新。模板中的变量和指令会根据数据的变化动态渲染内容，确保页面与数据始终保持一致，这也叫做数据的响应式处理

## 模板

> 模板（Template）是一个声明式的 HTML 结构，包含了 Vue 特定的语法（如插值表达式、指令等）和HTML结构，用于描述视图的结构和布局。模板定义了页面内容的呈现方式，并且通过数据绑定机制与 Vue 实例中的数据（data）进行关联。当数据发生变化时，模板会自动更新，确保视图与数据保持一致。

## 卸载Vue实例

```
app.$destory()
```

## 通过v-model简化父子通信

> 在子组件中，props通过value接收值，触发input事件修改值

```html
//父组件中
<BaseSelect v-model="selectId"></BaseSelect>
非简化写法:<BaseSelect ：value="selectId" @input="selectId=$event"></BaseSelect>
//子组件中
<select :value="value" @change="handleChange">...</select>
props : {
value:String
},
methods: {
	handleChange(e) {
	this.$emit('input',e.target.value)
	}
}
```

> 原理是v-model的双向绑定是基于动态绑定属性与事件的双向绑定

```html
<input type="text" :value="text" @input="text=$event.target.value"/>
```

- .sync指令修饰符

> 通过`.sync`在使用v-model简化父子通信时，prop的属性名可以自定义，非固定为value。等价于:属性名和@update:属性名合写，其中`@update:属性名`为事件名 

```
	<BaseDialog :isVisible.sync="isShow"></BaseDialog>
	等价于
		<BaseDialog
			:isVisible="isShow"
			@update:isVisible="isShow = $event"
		></BaseDialog>
		
		//子组件中修改属性值:
		methods: {
			close() {
				this.$emit("update:isVisible",false);
			},
		},
```

## ref和$refs

> 利用ref或$refs可以获取dom元素或组件实例，获取范围为当前组件内

- 使用

```html
1.为需要被获取的标签添加ref属性，值自定义
<div ref="box"></div>
2.通过this.$refs.ref值获得元素(元素必须渲染完成后才能获取)
this.$refs.box
3.获得组件方法
为子组件添加ref属性
<BaseForm ref="form"></BaseForm>
在父组件获得组件实例进而调用方法
this.$refs.form
this.$refs.form.组件方法()
```

## $nextTick

> Vue中的dom元素为异步渲染更新，不会在方法中根据每一行代码实时更新，而是会等方法执行完成后，统一进行更新，有的代码需要在dom更新后，才可以执行(如获取元素的代码),此时可以使用$nextTick语法，函数体中的代码会在dom更新完成后执行

```
this.$nextTick(函数体)
```

## SPA

> Singele Page Application , 单页应用程序，即网站的所有功能均在一个HTML页面上实现

- 与多页面应用程序对比

  |          |       单页       |       多页       |
  | :------: | :--------------: | :--------------: |
  | 实现方式 |   一个html页面   |  多个个html页面  |
  | 页面性能 | 按需更新，性能高 | 整页更新，性能低 |
  | 开发效率 |        高        |       一般       |
  | 用户体验 |        好        |       一般       |
  | 学习成本 |        高        |       中等       |
  | 首屏加载 |        慢        |        快        |
  |   SEO    |        差        |        优        |


## Vue的233和344

> Vue2对应VueRouter和Vuex版本为3.x
>
> Vue3对应VueRouter和Vuex版本为4.x

## 组件分类

> 组件分为页面组件&复用组件，页面组件一般配合路由使用，放在src下的views目录下；复用组件封装用于复用，需要在页面组件中使用，一般放在src下的components目录下。分开存放易于维护。

## JavaScript Standard Style规范

> 为了统一代码风格，定制的一套对于代码格式约定规则。代码规范有很多种，JavaScript Standard Style是最常用的。

- 常见规则
  1. 字符串使用单引号
  2.  无分号 
  3. 关键字后加空格 
  4.  函数名后加空格 
  5.  坚持使用全等 === 摒弃 ==

> Eslint是一个用来检查代码规范的插件

## vant-ui

> 一个基于Vue2的移动端组件库

- 安装

  1. 通过npm安装

     ```
     npm i vant@latest-v2
     ```

  2. 通过CDN安装

     ```js
     <!-- 引入样式文件 -->
     <link rel="stylesheet" href="https://unpkg.com/vant@2.12/lib/index.css" />
     
     <!-- 引入 Vue 和 Vant 的 JS 文件 -->
     <script src="https://unpkg.com/vue@2.6/dist/vue.min.js"></script>
     <script src="https://unpkg.com/vant@2.12/lib/vant.min.js"></script>
     
     <script>
       // 在 #app 标签下渲染一个按钮组件
       new Vue({
         el: '#app',
         template: `<van-button>按钮</van-button>`,
       });
     
       // 调用函数组件，弹出一个 Toast
       vant.Toast('提示');
     
       // 通过 CDN 引入时不会自动注册 Lazyload 组件
       // 可以通过下面的方式手动注册
       Vue.use(vant.Lazyload);
     </script>
     ```

- 引入

  - 自动按需引入组件

    > 自动按需导入需要依赖babel-plugin-import插件，这是一款 babel 插件，它会在编译过程中将 import 的写法自动转换为按需引入的方式。

    1. 安装插件
  
    ```bash
    npm i babel-plugin-import -D
    ```
  
    2. 配置插件

       ```js
       // 在.babelrc 中添加配置
       // 注意：webpack 1 无需设置 libraryDirectory
       {
         "plugins": [
         ["import", {
             "libraryName": "vant",
             "libraryDirectory": "es",
             "style": true
           }]
         ]
       }
       
       // 对于使用 babel7 的用户，可以在 babel.config.js 中配置
       module.exports = {
         plugins: [
           ['import', {
             libraryName: 'vant',
             libraryDirectory: 'es',
             style: true
           }, 'vant']
         ]
       };
       ```
  
    3. 在main.js中按需导入组件后即可全局使用此组件了
  
       ```
       import Vue from 'vue'
       import {Button} from 'vant'
       
       Vue.use(Button)
       ```
  
       一般不会将组件导入写在main.js中，会将其抽离到utils目录下vant-ui.js，然后在main.js中导入
  
       ```
       import Vue from 'vue'
       import { Button, Calendar, Rate } from 'vant'
       Vue.use(Button)
       Vue.use(Calendar)
       Vue.use(Rate)
       //main.js中
       import '@/utils/vant-ui.js'
       ```
  
       
  
    > 注意:
    >
    > - 插件会自动将代码转化为手动按需引入组件中的按需引入形式
  
  - 手动按需引入组件
  
  - 导入所有组件
  
    ```
    import Vue from 'vue';
    import Vant from 'vant';
    import 'vant/lib/index.css';
    
    Vue.use(Vant);
    ```
  
  > 注意:
    >
    > - 配置按需引入后，将不允许直接导入所有组件。
    > - 全部导入后可以直接使用vant中的所有组件
  
  > 注意:
  >
  > - 全部导入直接将所有组件导入项目，可以全局直接使用，使用时更加方便；按需导入只将需要使用到的组件导入项目使用，性能更高
  
- ViewPort布局适配

  > Vant 默认使用 `px` 作为样式单位，如果需要使用 `viewport` 单位 (vw, vh, vmin, vmax)，推荐使用 postcss-px-to-viewport进行转换。postcss-px-to-viewport是一款 PostCSS 插件，用于将 px 单位转化为 vw/vh 单位。

  - 安装

    ```
    npm install postcss-px-to-viewport@1.1.1 -D
    ```

  - 配置

    ```javascript
    // postcss.config.js
    module.exports = {
      plugins: {
        'postcss-px-to-viewport': {
            //vw适配的标准屏的宽度
          viewportWidth: 375,
        },
      },
    };
    ```

    

## 常用CDN

- jsdeliver
- cdnjs
- unpkg

## vue ui

## 欢迎机制

```
import Login from '@/views/login/index.vue'
与
import Login from '@/views/login'
效果一样
```

## element-ui

> 一个基于vue2的pc端组件库

- 引入

  - 完整引入

    ```jsx
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

    