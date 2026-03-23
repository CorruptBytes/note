# 概述

## 相比vue2的优势

- 使用组合式API，具有更好的TypeScript支持，更易于维护
- 重写diff算法，模板编译优化，更高效的组件初始化，具有更快的速度
- 具有良好的TreeShaking，支持按需引入，打包后具有更小的体积
- 采用Es7中的Proxy做底层代理，具有更优的数据响应式

## .vue文件与vue2区别

> 调整了`<script>`与`<template>`的位置

```vue
//加上setup表示使用组合式API
<script setup>
</script>

<template>
//模板中允许有多个根元素
  <header>
  </header>
  <main>
  </main>
</template>

<style scoped>
</style>

```

# 组合式API

> Vue3增加了使用组合式API定义组件逻辑，实现组件功能的选择。这是相比于vue2的最大区别

- 选项式API(Options API)

  > Vue 2 中使用选项式 API（Options API）来定义组件，它按功能类型组织逻辑，将组件的状态、方法、计算属性等分散在多个配置选项中。这些选项相互配合，共同实现组件的完整功能。

- 组合式API(composition API)

  > 组合式API按功能逻辑组织逻辑，通过在setup() 函数中统一处理组件逻辑，将相关的状态、方法、计算属性等聚合在一起定义。

- 优势
  1. 减少了代码量
  2. 分散式维护转为集中式维护，更容易封装复用

> 注意:
>
> - vue3仍然支持选项式API

## setup配置项

> `setup()` 是一个在组件初始化之前执行的函数，用于统一编写响应式数据、计算属性、方法、生命周期钩子等逻辑，最终通过 `return` 将它们暴露给模板使用。它是组合式API的入口

```jsx
<script>
export default {
	setup() {
	const message = 'this is Vue3'
	const logMessage = () => {
	console.log(message)
    }
	return {
	message,
	loMessage
	}
	},
	beforeCreate () {
	
	}
}
</script>
```

> 注意:
>
> - setup执行时机早于beforeCreate钩子 ,所以在setup中无法通过this关键字找到当前组件实例(undefined)
> - 在setup中定义的数据与方法必须return才可以在模板中使用

- setup语法糖

  > 通过在script标签中添加setup选项简化了setup配置项的写法

  ```jsx
  <script setup>
  	const message = 'this is Vue3'
  	const logMessage = () => {
  	console.log(message)
  	}
  </script>
  ```

  > 注意:
  >
  > - setup选项的原理是声明这个选项后，vue底层会将相关数据与方法声明在setup()选项中，并在这个函数中return这些数据与方法
  > - setup是将script标签中的所有内容全部变为setup配置项的内容,所以正常情况下无法在加了setup语法糖的标签下配置其他配置项

## reactive 与 ref

> 默认在script中声明的数据并不是响应式数据，需要使用reactive函数或ref函数声明响应式数据

- reactive

  > 接受对象类型数据的参数传入并返回一个响应式对象,从传入对象的属性会直接挂载到这个响应式对象上

  ```
  <script setup>
  import { reactive } from "vue";
  
  const state = reactive({count: 100})
  const add = () => {
    state.count++
  }
  </script>
  ```

- ref

  > 接收简单类型或者对象类型的数据并返回一个响应式对象，传入的数据被挂载到这个响应式对象的value属性上

  ```
  <script setup>
  import { reactive,ref } from "vue";
  
  const count = ref(0)
  const addRef = () => {
    count.value++
  }
  </script>
  <template>
  <div>{{ count }}</div>
  <button @click="addRef">点击ref</button>
  </template>
  ```

  > 注意:
  >
  > - ref本质上是将传入的数据转换为复杂类型后，再借助reactive实现响应式
  > - 在模板中，可以直接通过变量名访问数据，不需要通过value属性

## computed

> 在组合式API中，通过computed函数声明一个计算属性，computed的参数为一个函数，这个函数的必须有返回值，这个返回值就是计算属性

```js
<script setup>

import {computed} from 'vue'
//简写，计算属性不包括set方法
const computedState = computed(() => {
return 基于响应式数据的计算属性
})
//完整写法
const computedState = computed({
    get: () => {return 基于响应式数据的计算属性},
    //val接收传来的值
    set: (val) => {
        set逻辑
    }
})
</script>
```

> 注意：
>
> - 计算属性需要用一个变量接收
> - 计算属性中不应该有"副作用"，比如异步请求或dom操作
> - 避免直接修改计算属性的值

## watch

> 组合式API通过watch函数获得一个监听器，watch函数的第一个参数为被监视的数据，第二个为数据变化后的回调函数，第三个参数为配置项

- 监听单个数据

  ```js
  <script setup>
  import {ref, watch} from 'vue'
  const count = ref(0)
  //第一个参数是被监听的响应式数据，第二书参数是数据修改时的回调函数
  watch(count,(newValue,oldValue) => {
      console.log(newValue, oldValue)
  })
  </script>
  ```

- 监听多个数据

  > 将多个数据用数组封装起来作为watch函数的第一个参数

  ```js
  <script setup>
  import {ref, watch} from 'vue'
  const count = ref(0)
  //第一个参数是被监听的响应式数据，第二书参数是数据修改时的回调函数
  watch([count, name] ,([newCount, newName], [oldCount,oldName]) => {
  })
  </script>
  ```

  > 注意:
  >
  > - watch函数接收的数据是响应式对象，不是对象中的数据
  > - watch()函数调用后不需要使用变量接收

- 配置项

  > watch函数的第三个参数，为一个对象，用于配置immediate立即执行和deep深度监视

  ```
  watch(count,()=> {},{immediate: true,deep: true})
  ```

- 监视对象的某个属性

  > 此时watch的第一个参数不再是对象形式，而是一个函数，这个函数的返回值为对象被监视的属性

  ```
  const info = ref({
  name: 'cp',
  age: 18
  })
  watch(() => info.value.age,() => {
  
  })
  ```

## 生命周期钩子

> 在vue3中，仍然可以通过组合式api的配置项配置声明周期钩子，也可以使用组合式api的函数配置钩子，

![image-20250506172046201](D:\笔记\图片\image-20250506172046201.png)

> 注意:
>
> - beforeCreated/created相关的钩子全部配置在setup中
> - vue2中销毁的钩子函数为beforedestroy与destroyed，在vue3中变成了beforeUnmount与ummounted
> - 可以配置多个生命周期钩子，他们不会冲突，而是会按照顺序依次执行

## 父子通信

> 组合式API下的父子通信与选项式API思想一致，仍然是父组件给子组件绑定属性，子组件通过props选项接收；只是组合式API中使用了setup语法糖后，无法再声明配置项，此时需要借助"编译器宏"函数接收传递的数据

- 传递非响应式数据

```js
//父组件
<template>
<MySon message="你好"></MySon>
</template>

/子组件
<script setup>
    //props是一个对象，封装了传递过来的数据，对象属性名即为父组件绑定的属性名
const props =  defineProps({
    message: String
})
console.log(props.message)
</script> 

<template>
    //在模板中，可以直接通过属性名使用父组件传递的数据
<div class="son">我是子组件 {{message}}</div>
</template>
```

- 传递响应式数据

  ```
  //父组件
  <template>
  <MySon :message="响应式对象"></MySon>
  </template>
  
  /子组件
  <script setup>
      //props是一个对象，封装了传递过来的数据，对象属性名即为父组件绑定的属性名
  const props =  defineProps({
      message: 响应式对象封装数据的类型
  })
  console.log(props.message)
  </script> 
  
  <template>
      //在模板中，可以直接通过属性名使用父组件传递的数据
  <div class="son">我是子组件 {{message}}</div>
  </template>
  ```

## 子传父

> 父组件不变，仍然是为子组件绑定事件并提供事件处理函数
>
> 子组件通过emit方法触发事件，但是在setup中无法通过this获得当前vue实例，需要借助defineEmits编译器宏函数生成emit方法

```js
//父组件
<template>
<MySon :count="count" @addCount="count = $event"></MySon>
</template>

//子组件
//defineEmits的参数为一个数组，数组为被触发的事件名，返回值为一个函数，通过这个函数触发事件
const emit = defineEmits(['addCount'])
const add = () => {
    //调用函数通知父组件，第一个参数为事件名，第二个参数为事件传递的值
    emit('addCount', props.count+1)
}
```

## 模板引用

> 通过ref标识获取真实的dom对象或者组件实例对象

- 使用

  1. 调用ref函数获得ref对象

     ```
     import {ref} from 'vue'
     const h1Ref = ref(null)
     
     ```

  2. 通过ref标识绑定ref对象

     ```
     <template>
     <h1 ref="h1Ref" > </h1>
     </template>
     ```

  3. 通过ref对象操作dom或组件

     > 页面渲染完成后可以通过ref对象的value属性拿到dom元素或组件实例，之后可以通过它调用组件中的属性和方法

     - defineExpose()

       > 使用setup语法糖的组件内部的属性与方法是不开放给父组件访问的，此时即使拿到组件实例仍然无法使用，可以通过defineExpose编译宏指定哪些属性和方法允许访问

       ```
       defineExpose({
       需要暴露的属性和方法
       })
       ```

## provide与inject

> 实现顶层组件向任意的底层组件传递数据和方法，实现跨层组件通信

## 传递普通数据

1. 顶层组件通过provide函数提供数据

   ```
   provide('key',顶层组件中的数据)
   ```

2. 底层组件通过inject函数获取数据

   ```
   inject('key')
   ```

### 传递响应式数据

1. 顶层组件通过provide函数提供响应式对象

   ```
   provide('key',响应式对象)
   ```

2. 底层组件通过inject函数获取数据

   > 子组件接收到的数据仍然是一个响应式对象，此时如果想要修改它，顶层组件可以顺带传递一个修改这个数据的函数

   ```
   inject('key')
   ```

# Vue3新特性

![image-20250506181953556](D:\笔记\图片\image-20250506181953556.png)

## defineOptions

> 在vue3.3引入，用来定义Options API中的其他配置项。可以用来定义任意的选项，除了props，emits，expose，slots(这些有专门的宏函数)

```
<script setup>
defineOptions({
name:'',
inheritAttrd:false,
//...更多自定义属性
})
</script>
```

## defineModel

> vue3中修改了v-model的底层原理

![image-20250506182708818](D:\笔记\图片\image-20250506182708818.png)

> defineModel简化了这个语法，可以通过defineModel()函数直接接收传递的值，然后可以直接在子组件中修改这个值

![image-20250506183041643](D:\笔记\图片\image-20250506183041643.png)

> 这是一个实验性的更新，需要在vite.config.js中配置才能使用

```
import { fileURLToPath, URL } from 'node:url'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueDevTools from 'vite-plugin-vue-devtools'

// https://vite.dev/config/
export default defineConfig({
  plugins: [
    vue({
    //配置开启defineModel
      script: {
        defineModel: true
      }
    }),
    vueDevTools(),
  ],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    },
  },
})

```

> 现在已经不需要配置了，可以直接使用

# Pinia

> Vue最新的状态管理工具，已经替换了Vuex

- 优势

  1. 更加简单的API(去掉了mutation)
  2. 提供符合组合式风格的API(和Vue3新语法统一)
  3. 去掉modules的概念，每一个store都是一个独立的模块
  4. 配合TypeScript更加友好，提供可靠的类型推断

- 组成

  > 现在完整的仓库由state，actions，getters三个模块组成
  
- 安装

  > 可以通过包管理器安装

  ```
  npm install pinia
  ```

- 挂载pinia

  ```js
  import { createApp } from 'vue'
  import { createPinia } from 'pinia'
  import App from './App.vue'
  
  const pinia = createPinia()
  const app =  createApp(App).mount('#app')
  app.use(pinia)
  
  //相当于
  createApp(App).mount('#app').use(pinia)
  ```

- 定义store

  > pinia中，通过defineStore()函数创建一个store，第一个参数为这个store的名字，是唯一且必须的，第二个参数用来配置store，可以接收两类值:Setup 函数或 Option 对象

  ```js
  import { defineStore } from 'pinia'
  
  // 你可以任意命名 `defineStore()` 的返回值，但最好使用 store 的名字，同时以 `use` 开头且以 `Store` 结尾,因为返回值是一个函数
  export const useAlertsStore = defineStore('alerts', {
   	//其他配置
  })
  ```

  > defineStore()的返回值是一个函数，通过这个函数可以获得仓库实例，因此需要将这个函数export暴露出去

  - Option Store

    > 使用选项式API的形式配置store

    ```
    export const useCounterStore = defineStore('counter', {
      state: () => ({ count: 0, name: 'Eduardo' }),
      getters: {
        doubleCount: (state) => state.count * 2,
      },
      actions: {
        increment() {
          this.count++
        },
      },
    })
    ```

  - Setup Store

    > 以组合式API的形式配置store，第二个参数传入一个函数，在函数中，通过ref定义state，computed定义getters，function定义actions

    ```js
    export const useCounterStore = defineStore('counter', () => {
      const count = ref(0)
      const doubleCount = computed(() => count.value * 2)
      function increment(num) {
        count.value+=num
      }
    
      return { count, doubleCount, increment }
    })
    ```

    > 注意:
    >
    > - 所有定义的配置都需要通过return对象的形式暴露出去

- 使用store

  > 通过defineStore()返回值的函数在组件中获得仓库实例，所有state，getters，actions都会直接挂载在仓库实例上，通过实例.名调用即可

  ```js
  <script setup>
  import { useCounterStore } from '@/store/counter'
  const counterStore = useCounterStore()
  </script>
  
  <template>
    <div>我是Son1组件-{{counterStore.count}} 
        <button @click="counterStore.increment(1)">+</button></div>
  </template>
  ```

  > **Pinia 中的 store 是通过 `reactive` 包装的响应式对象**，因此无论访问 `state` 中的属性还是 `getters`，都可以**直接使用**，**无需在后面添加 `.value`**

## store解构数据

> 由于配置项定义的状态是直接挂载到store上的，所以可以对其进行结构获得所需状态；但是默认情况下，结构后的数据会丢失响应式特性；如果想要避免，需要使用storeToRefs(仓库实例)后进行结构，此时数据仍具有响应式,因为这个函数将为每一个响应式属性创建引用;方法可以直接结构

```
const {count,msg} = storeToRefs(counterStore)
```

## Pinia持久化插件

- 安装

  ```
  npm install pinia-plugin-persistedstate
  ```

- 将插件添加到pinia实例上

  ```JS
  import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'
  const pinia = createPinia()
  pinia.use(piniaPluginPersistedstate)
  ```

- 在store中声明开启插件

  - 组合式API

    > 在第三个参数的位置用以对象属性的形式声明

    ```
    import { defineStore } from 'pinia'
    import { ref } from 'vue'
    
    export const useStore = defineStore(
      'main',
      () => {
        const someState = ref('hello pinia')
        return { someState }
      },
      {
        persist: true,
      },
    )
    ```

  - 选项式API

    > 在配置项中添加persist选项

    ```js
    import {defineStore} from 'pinia'
    export const useStore = defineStore('main',{
    	state () {
    		return {
                
            }
    	},
        persist: true,
    })
    ```

> 注意：
>
> - 开启插件后，整个state将持久化到存储中，默认使用localStorage 作为存储，store.$id 作为存储的默认 key。JSON.stringify/destr 作为序列化器/反序列化器。

- 配置

  > 可以向第三个参数的persist属性传入一个对象来配置持久化

  ```js
  import { defineStore } from 'pinia'
  import { ref } from 'vue'
  
  export const useStore = defineStore('main', () => {
    const someState = ref('hello pinia')
    return { someState }
  }, {
    persist: {
      //配置持久化时的键名
      key: '', 
      //配置持久化的方式
      sotrage: sessionStorage, 
      //配置需要持久化的数据，默认全部持久化；下面为对象属性配置方式与普通数据的持久化方式
      paths:['save.me','saveMeToo'] 
    }
  })
  ```




# VueRouter4

> 这是Vue3所使用的路由插件

- 创建路由器

  > createRouter()函数创建一个路由器

  ```js
  import { createRouter, createWebHistory } from 'vue-router'
  
  const router = createRouter({
    //配置路由模式,这里import.meta.env是一个对象，对象上的属性是vite中的环境变量，可以在配置文件中配置
    history: createWebHistory(import.meta.env.BASE_URL),
    routes: [
      {
  
      },
    ],
  })
  ```

  > 注意:
  >
  > - createWebHistory表示路由模式为历史模式，createWebHashHistory表示路由为哈希模式，这两个函数的参数是基础路径，默认是`/`
  > - 

- 在组件中获取并使用路由

  ```
  //获得路由对象
  const router = useRouter()
  rouer.push('路由路径')
  //获得当前路由参数
  const route = useRoute()
  ```

  > 注意:
  >
  > - 在模板中，仍然可以通过$router获得路由对象
  > - 路由对象也可以通过导入获得，但是在setup中更推荐使用use函数

# 小知识

## create-vue

> create-vue是Vue官方新的脚手架工具，将底层切换到了vite(新一代构建工具)

- 创建并启动vue应用

  ```bash
  //这一指令会安装并执行create-vue
  npm init vue@latest
  //安装依赖
  npm install
  //启动应用
  npm run dev
  ```

> 注意:
>
> - create-vue只支持16.0或以上版本的Node.js

## 组件使用

> vue3中仍然支持局部注册组件和全局注册组件，全局注册组件同vue2，局部注册只需要导入，导入后可直接在模板中使用

## pnpm

> 包管理工具，相比npm与yarn，pnpm速度更快，而且更加节省磁盘空间

- 安装

  ```
  npm install -g pnpm
  ```

- 创建vue项目

  ```
  pnpm create vue
  ```

- 运行vue项目

  ```
  pnpm dev
  ```

![image-20250507164209759](D:\笔记\图片\image-20250507164209759.png)

## Element Plus

> 基于Vue3的PC端组件库

- 安装

  ```
  pnpm add element-plus
  ```

- 引入

  - 完整引入

    ```
    import { createApp } from 'vue'
    import ElementPlus from 'element-plus'
    import 'element-plus/dist/index.css'
    import App from './App.vue'
    const app = createApp(App)
    app.use(ElementPlus)
    app.mount('#app')
    ```

  - 自动按需引入

    1. 下载插件

       > 需要借助`unplugin-vue-components` 和 `unplugin-auto-import`这两款插件

       ```
       npm install -D unplugin-vue-components unplugin-auto-import
       ```

    2. 添加配置

       > 需要在Vite或Webpack的配置文件中添加配置

       ```
       //Vite
       import { defineConfig } from 'vite'
       import AutoImport from 'unplugin-auto-import/vite'
       import Components from 'unplugin-vue-components/vite'
       import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'
       
       export default defineConfig({
         // ...
         plugins: [
           // ...
           AutoImport({
             resolvers: [ElementPlusResolver()],
           }),
           Components({
             resolvers: [ElementPlusResolver()],
           }),
         ],
       })
       //Webpack
       const AutoImport = require('unplugin-auto-import/webpack')
       const Components = require('unplugin-vue-components/webpack')
       const { ElementPlusResolver } = require('unplugin-vue-components/resolvers')
       
       module.exports = {
         // ...
         plugins: [
           AutoImport({
             resolvers: [ElementPlusResolver()],
           }),
           Components({
             resolvers: [ElementPlusResolver()],
           }),
         ],
       }
       ```

    3. 直接使用组件

       > 直接在需要的位置写上element-plus定义的组件标签就可以使用，此时不但可以直接使用element-plus中的组件，自己定义的组件也可以直接使用，无需导入，默认组件名为驼峰命名转横线

## eslint整合prettier

- 一个貌似可用的eslint.config.js文件

  ```js
  import { defineConfig } from 'eslint/config'
  import pluginVue from 'eslint-plugin-vue'
  import vuePrettier from '@vue/eslint-config-prettier'
  import globals from 'globals'
  
  export default defineConfig([
    {
      files: ['**/*.{js,vue}'],
      plugins: { vue: pluginVue },
      languageOptions: {
        globals: { ...globals.browser, ...globals.node },
        parserOptions: {
          ecmaVersion: 'latest',
          sourceType: 'module'
        }
      },
      rules: {
        'prettier/prettier': [
          'warn',
          {
            singleQuote: true, // 单引号
            semi: false, // 无分号
            printWidth: 80, // 每行宽度至多80字符
            trailingComma: 'none', // 不加对象|数组最后逗号
            endOfLine: 'auto' // 换行符号不限制（win mac 不一致）
          }
        ],
        'vue/multi-word-component-names': [
          'warn',
          {
            ignores: ['index'] // vue组件名称多单词组成（忽略index.vue）
          }
        ],
        'vue/no-setup-props-destructure': ['off'], // 关闭 props 解构的校验
        // 💡 添加未定义变量错误提示，create-vue@3.6.3 关闭，这里加上是为了支持下一个章节演示。
        'no-undef': 'error'
      }
    },
    ...pluginVue.configs['flat/essential'],
    vuePrettier, // ✅ 直接展开配置（不再用 .configs[0]），这个插件貌似是配置生效的关键
    { ignores: ['**/dist/**'] }
  ])
  
  ```

  

## magicui

## aceternity UI

## Framer Motion

![image-20250516112324456](D:\笔记\图片\image-20250516112324456.png)