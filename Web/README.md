# Web

## Topic

### Cookie & session & token

Cookie也是实现Session的一种方式。
session 在服务器端，cookie 在客户端（浏览器）

1. session 在服务器端，cookie 在客户端（浏览器）
2. session 默认被存在在服务器的一个文件里（不是内存）
3. session 的运行依赖 session id，而 session id 是存在 cookie 中的，也就是说，如果浏览器禁用了 cookie ，同时 session 也会失效（但是可以通过其它方式实现，比如在 url 中传递 session_id）
4. session 可以放在 文件、数据库、或内存中都可以。
5. 用户验证这种场合一般会用 session 因此，维持一个会话的核心就是客户端的唯一标识，即 session id

name(名称） | value 
----------- | --- 
path  |表示 cookie 影响到的路径，匹配该路径才发送这个 cookie
expires 和 maxAge | 告诉浏览器这个 cookie 什么时候过期，expires 是 UTC 格式时间，maxAge 是 cookie 多久后过期的相对时间。当不设置这两个选项时，会产生 session cookie，session cookie 是 transient 的，当用户关闭浏览器时，就被清除。一般用来保存 session 的 session_id。
secure |当 secure 值为 true 时，cookie 在 HTTP 中是无效，在 HTTPS 中才有效
httpOnly |浏览器不允许脚本操作 document.cookie 去更改 cookie。一般情况下都应该设置这个为 true，这样可以避免被 xss 攻击拿到 cookie。

session 的数据存储在服务器端， token的数据存储在客户端
session id == token id
> [参考文档](http://www.cnblogs.com/songhan/archive/2012/07/23/2604299.html)


## Vue

### Topic

#### 异步组件加载

首先，可以将异步组件定义为返回一个 Promise 的工厂函数 
(该函数返回的 Promise 应该 resolve 组件本身)：
`const Foo = () => Promise.resolve({/* 组件定义对象 */})`

第二，在 Webpack 2 中，我们可以使用动态 import语法来定义代码分块点 (split point)：
`import('./Foo.vue') // 返回 Promise`

结合这两者，这就是如何定义一个能够被 Webpack 自动代码分割的异步组件。
`const Foo = () => import('./Foo.vue')`


### 自定义指令（v-directive） 

Mixin  

### Vue-router

#### 基本用法

```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
```

##### router.beforeEach

每次进入路由前都会执行的函数

```js
router.beforeEach((to, from, next) => {
    next() // 不会再次调用 beforeEach 函数
    next({path:'/home'}) // 会重新调用一次 beforeEach 函数
    // 确保要调用 next 方法，否则钩子就不会被 resolved（释放）。
})
```


### Vuex

#### 基本用法

State, Getter, Mutation, Action, Module

```js
import Cookies from 'js-cookie'
const app = {
  namespaced: true,   // 是否启用命名空间模块
                      // 当模块被注册后，它的所有 getter、action 及 mutation 
                      // 都会自动根据模块注册的路径调整命名
  state: {
    sidebar: {
      opened: !+Cookies.get('sidebarStatus')
    },
  },
  getters: {
    sidebar: state => {
      return state.sidebar
    },
  },
  mutations: {
    TOGGLE_SIDEBAR: state => {
      if (state.sidebar.opened) {
        Cookies.set('sidebarStatus', 1)
      } else {
        Cookies.set('sidebarStatus', 0)
      }
      state.sidebar.opened = !state.sidebar.opened
    },
  },
  actions: {
    toggleSideBar({ commit }) {
      commit('TOGGLE_SIDEBAR')
    },
  }
}
    
export default app
```

组建内调用  

```js
import {mapGetters, mapActions} from 'vuex'

export default {
    data(){
        return {}
    },
    computed: {
        ...mapGetters({
            'getter' : 'modules/getter',
        }),
        otherData(){},
    },
    method: {
        ...mapActions({
            'action' : 'modules/action',
        }),
        otherMethod(){},
    },
    
}
```



-------


## React

### lifecycle

![](media/15113179729510/15294772568103.jpg)



example:

```js
class Clock extends React.Component {
    constructor(props) {
        super(props)
        this.state = { date: new Date() };
    }
}
// Mount
// The componentDidMount() hook runs after the component output has been rendered to the DOM
componentDidMount() {
    this.timerID = setInterval(
        () => this.tick(),
        1000
    )
}

// 
// Unmount
componentWillUnmount() {
    clearInterval(this.timerID);
}

tick() {
    this.setState({
        date: new Date()
    })
}

render() {
    return (
        <h2>It is { this.state.date.toLocalTimeString() }.</h2>
    );
}


ReactDOM.render(
    <Clock />,
    document.getElementById('root')
);
```

### concept

* The Data Flows Down
* you cannot return `false` to prevent default behavior in React. You must call `preventDefault` explicitly



### 规范

* React events are named using camelCase









-------


## JavaScript(ES6)

### 常用指令

#### let 

1. 不存在变量提升
2. 不允许重复声明
3. 块级作用域

#### const

1. 声明常量， 一旦申明，其值不能改变  
2. 跨模块常量
   eg: `export const A = 1;`

3. 全局对象的属性  
    
    ```js
    var a = 1;
    window.a  // 1
    let b = 1;
    window.b  // undefined
    ```

#### yield 

```js
function* countAppleSales () {
  var saleList = [3, 7, 5];
  for (var i = 0; i < saleList.length; i++) {
    yield saleList[i];
  }
}

var appleStore = countAppleSales(); // Generator { }
console.log(appleStore.next()); // { value: 3, done: false }
console.log(appleStore.next()); // { value: 7, done: false }
console.log(appleStore.next()); // { value: 5, done: false }
console.log(appleStore.next()); // { value: undefined, done: true }
```

### 常用方法

#### Math

```js
// 向上取整
Math.ceil(2.0) // 2
Math.ceil(2.1) // 3

// 向下取整
Math.floor(1.23) // 1

// 四舍五入
Math.round(1.4) // 1
Math.round(1.5) // 2 
```

### 异步编程

#### Promise 的用法  

```js
let new_Promise = new Promise((resolve, reject) => {
    if(true){
        resolve('data')  // 执行成功，返回成功结果
    }else{
        reject('error')  // 执行失败，返回失败信息
    }
})
```
#### async/await 的用法


```js
// 定义异步操作函数
do_async = (interval) => {
    return new Promise((resolve) => {
        setTimeout(() => resolve('ok'), interval)
    })
}

// 使用 async & await
async function test(interval){
    let t = await do_async(interval)
    console.log('i am' ${t})
}

test(1000)

// output:
// test(1000)
// Promise {[[PromiseStatus]]: "pending", [[PromiseValue]]: undefined}
// i am ok
```

### 复制变量值
- 基本类型值：原值改变，副本不变
- 引用类型值（object，list）：指向同一个地址空间

### 新建引用类型值(object, array)

#### 创建 Object 实例

```js
// method 1
var person1 = new Object();
person.name = 'zyl';
// method 2
var person2 = {name: 'zyl'}
// method 3 深度拷贝一个对象
let person3 = Object.assign({}, person2)
let person4 = JSON.parse(JSON.stringify(person3))
```    

#### 创建 Array 实例

```js
// method 1
var array = new Array()

// method 2
var array = []
```

#### 深度copy一个引用类型值

```js
// 只能copy一层，
new_data = Object.assign({}, data)

// 一个全新的对象
let new_data = JSON.parse(JSON.stringify(data))
```

#### 数组

##### 数组的基本操作

###### 遍历一个数组

```js
data = ['one', 'two', "three", "four"]
// Use for-loop
for (let index in data) {
    console.log(index + '=>' + data[index])
}
/*  output
0=>one
1=>two
2=>three
3=>four
*/

// Use Array property `forEach`

data.forEach((value, index) => {
    console.log(index + '=>' + value)
})
/*  output
0=>one
1=>two
2=>three
3=>four
*/
```

###### 过滤数组返回匹配的新数组

```js
data = ['one', 'two', "three", "four"]
new_data = data.filter((value, index) => {
    console.log(index + '=>' + value)
    return value == "two"
})
console.log(new_data)
/* output
0=>one
1=>two
2=>three
3=>four
["two"]
*/
```

###### 删除数组中的某个元素

```js
// 删除指定索引的元素
data = ['one', 'two', "three", "four"]
data.splice(1, 1) // output: ["one", "three", "four"]

```

###### 对象转换为数组


```js

```

#### 对象

##### 对象的基本操作

###### 遍历一个对象

```js
// Use for-loop
obj = {"name": "test", "age": 24, "sex": "man"}
for (let key in obj) {
    console.log(key + '=>' +obj[key])
}
/* output Note: **Must have a single binding**
name=>test
age=>24
sex=>man
*/
```

###### hasOwnProperty 

```js
obj = {"name": "test", "age": 24, "sex": "man"}
obj.hasOwnProperty("name")  // true
obj.hasOwnProperty("name2") // false
```


### 变量的解构赋值

> *尽量避免使用圆括号，会产生解构歧义*

#### 数组的解构赋值

```js
// 模式匹配
let [a, b, c] = [1, 2, 3]
a // 1
b // 2 
c // 3

let [a, , b] = [1, 2, 3]
a // 1
b // 3

// 默认值
let [a = 1] = [];
a // 1
```

#### 对象的解构赋值


```js
let { foo, bar } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"
```

### 字符串内置的一些常用方法

#### include & startsWith & endsWith

```js
let a = 'hello world !'
a.startsWith('h') // true
a.endsWith('!')   // true
a.include('o')    // true
```

#### repeat

返回一个新字符串，表示将原字符串重复n次
`'x'.repeat(3) // 'xxx'`



### 定时器
g

----------

## TypeScript

### 优点

1. TypeScript 增加了代码的可读性和可维护性
类型系统实际上是最好的文档，大部分的函数看看类型的定义就可以知道如何使用了
可以在编译阶段就发现大部分错误，这总比在运行时候出错好
增强了编辑器和 IDE 的功能，包括代码补全、接口提示、跳转到定义、重构等

2. TypeScript 非常包容
TypeScript 是 JavaScript 的超集，.js 文件可以直接重命名为 .ts 即可
即使不显式的定义类型，也能够自动做出类型推论
可以定义从简单到复杂的一切类型
即使 TypeScript 编译报错，也可以生成 JavaScript 文件
兼容第三方库，即使第三方库不是用 TypeScript 写的，也可以编写单独的类型文件供 TypeScript 读取

3. TypeScript 拥有活跃的社区
大部分第三方库都有提供给 TypeScript 的类型定义文件
Google 开发的 Angular2 就是使用 TypeScript 编写的
ES6 的一部分特性是借鉴的 TypeScript 的（这条需要来源）
TypeScript 拥抱了 ES6 规范，也支持部分 ES7 草案的规范

### 缺点

1. 有一定的学习成本，需要理解接口（Interfaces）、泛型（Generics）、类（Classes）、枚举类型（Enums）等前端工程师可能不是很熟悉的东西。而且它的中文资料也不多  
2. 短期可能会增加一些开发成本，毕竟要多写一些类型的定义，不过对于一个需要长期维护的项目，TypeScript 能够减少其维护成本（这条需要来源）
3. 集成到构建流程需要一些工作量
4. 可能和一些库结合的不是很完美（这条需要举例）

### 基本用法

> [参考学习文档](https://ts.xcatliu.com/introduction/what-is-typescript.html)  

Installing TypeScript
`npm install -g typescript`

Edit `greeter.ts`  

```js
function greeter(person) {
    return "Hello, " + person;
}

let user = "Jane User";

document.body.innerHTML = greeter(user);
```

Compiling your code
`tsc greeter.ts`

## jQuery

## Webpack

### 使用vue-cli Webpack 

    





