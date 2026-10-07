# Promise

`Promise` 在 ES6 规范中添加，是 `JS` 中进行异步编程的新解决方案。它是 JavaScript 中用于封装和管理异步操作结果的对象。

- 旧解决方案是单纯使用回调函数。

<h4>优点</h4>

- 支持链式调用，可以缓解回调地狱问题。回调地狱是指多个存在依赖关系的异步操作通过回调函数层层嵌套，导致代码层级过深、可读性和维护性变差。

●使用

Promise本质是一个构造函数，需要实例化这个对象并传入一个函数，函数中封装异步任务，函数具有两个形参：resolve，reject，这两个形参都是回调函数，当异步任务成功时，调用resolve，失败时调用reject
回调函数的具体逻辑通过then方法设置，then方法设置回调函数的同时调用异步任务
异步任务的结果值可以通过回调函数的形参进行传递



![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742476144789-6ba936e7-a68e-4753-8fe6-d95803cced77.png)



 Promise对象状态属性 

●PromiseState

表示Promise对象的状态，三种状态：pending（未决定） ， resolved/fulfilled(成功) ，rejected（失败）,且一个promise对象的状态只能改变一次
这个状态由resolve和reject函数改变

●PromiseResult

保存异步任务成功或失败的结果由resolve与reject函数改变

 Promise的执行过程 

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742480545306-7bbb624f-f4b2-41f7-a78d-79a4f5d78092.png)



 Promise构造函数![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742480867096-b9f2a6a4-89f1-40de-8b51-31a5505639a6.png) 

 then函数 

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742481021865-069ab6a7-7bf8-4f67-ab94-b09acb5090dc.png)



 catch函数![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742481033204-2618526c-d1e5-4fcd-9c99-065ab5fd555a.png) 

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742481122217-7fc4d204-f8bf-4e10-8267-9501cfdc5282.png)

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742481222677-35fd42bb-586c-4836-a1a0-308e13e4b145.png)

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742481262641-67fe8cf4-97b6-4941-88a3-56ebef789eca.png)

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742481355228-e5837558-b0dc-49ca-984e-9a4d6675f4fa.png)

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742481459753-59ae64eb-26c6-4d0a-875f-5fa039ed590f.png)

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742481763277-df7caa52-9c20-49d2-a8d1-e820102e5a80.png)





![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742481994482-f407f30c-e408-49cc-94f6-e62b2e48b960.png)



回调函数执行的时机是promise对象的状态改变时

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742482181121-7714bbf6-d40d-4d2f-befa-3bfabc35568d.png)

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742482688577-ba3ceb42-5d63-42a4-b84d-24b6fdd7d379.png)

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742538671959-c7a4fe3c-9a65-4233-ba7d-94ee43629fce.png)

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742538999927-c82172b3-ea25-4ace-b5b1-0e5b72719eb1.png)

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742539009712-18bc512e-227c-448d-8e32-aa5f4473e795.png)

![img](https://cdn.nlark.com/yuque/0/2025/png/40539843/1742539080483-88931d1d-afd1-4142-ab74-e21951331ca8.png)



 小知识 

使用运行窗口打开软件时，如果想要用管理员权限打开，需要按住ctrl + shift + enter

若有收获，就点个赞吧