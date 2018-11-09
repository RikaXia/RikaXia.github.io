---
layout:     post
title:      create-react-app+mobx入门初体验
subtitle:   create-react-app配置装饰器环境
date:       2018-11-07
author:     Rika
header-img: img/post-bg-keybord.jpg
catalog: true
tags:
    - mobx
---  
  
# 1.本文目标
阅读本文章您可能收获如下：

- 配置修饰器环境
- 理解响应式编程的概念
- 正确使用mobx关键api达到维护应用程序状态的目标

# 2.什么是修饰器？
Decorator是在 **声明阶段** 实现类与类成员注解的一种语法。  

说的直白点Decorator就是 **添加** 或者 **修改** 类的变量与方法。

## 2.1 一定需要修饰器吗？
在开始使用mobox，我们还需要纠结一个东西，就是配置环境启用ES7的修饰器语法，当然，如果你不需要修饰器，可以跳过这一部分。

如果是新手的话，建议配置，官方的MobX文档也说明了，不一定要使用修饰器，如果有人说你必须在MobX中使用decorator，那就不是真的，你也可以使用普通函数，如下：

```
import { decorate, observable } from "mobx";

class Todo {
    id = Math.random();
    title = "";
    finished = false;
}
decorate(Todo, {
    title: observable,
    finished: observable
})
```
## 2.2 使用和不使用修饰符的对比
**不使用**修饰符如下：

```
import React, {Componnet} from 'react';
import {observe} from 'mobx-react';

// 没有使用 修饰器
class Test extends Componnet {
    ...
}

export default observe(Test);
```
**使用**修饰符如下：

```
import React, {Componnet} from 'react';
import {observe} from 'mobx-react';

// 使用 修饰器
@observe class Test extends Componnet{
    ...
}

export default Test;
```
以上代码中，通过observer(App)定义类和@observer class App 装饰一个组件是一样的。  

那么多个修饰符组合到一个组件上是怎样的呢？。  

**不使用**修饰符如下：

```
import React, {Componnet} from 'react';
import {observe, inject} from 'mobx-react';
import {compose} from 'recompose';

// 没有使用 装饰器
class Test extends Componnet {
  render() {
    const {foo} = this.props;
  }
}

export default compose(
  observe,
  inject('foo')
)(Test)
```

**使用**修饰符如下：

```
import React, {Componnet} from 'react';
import {observe, inject} from 'mobx-react';

// 使用 修饰器
@inject('foo') @observe
class Test extends Componnet {
  render() {
    const {foo} = this.props;
  }
}

export default Test;
```
由上可以看出，如果没有修饰器的话，需要引入recompose,通过compose将多个修饰符组合到Test上，如果使用修饰符的话，则可以直接在class Test前进行修饰，如上面代码中@inject('foo') @observe，两者相比之下，可以看出通过修饰器来修饰的方式会更加简洁易懂些。更多详情，阅读[mobx中文文档](https://cn.mobx.js.org/best/decorators.html)

## 2.3 使用修饰符的优缺点

**使用修饰符的优点：**
- 样板文件最小化，声明式代码。
- 易于使用和阅读。大多数 MobX 用户都在使用。 

**使用修饰符的缺点：**
- ES.next 2阶段特性。
- 需要设置和编译，目前只有 Babel/Typescript 编译器支持。

# 3.create-react-app+mobx（装饰器）
create-react-app 目前还没有内置的装饰器支持，所以此小结要解决这个问题。

## 3.1.安装react-app-rewire相关
安装 [react-app-rewired ](https://github.com/timarney/react-app-rewired)
```
npm install react-app-rewired --save-dev
```
修改 package.json 里的启动配置

```
/* package.json */
"scripts": {
    "start": "node scripts/start.js",
    "build": "node scripts/build.js",
    "test": "node scripts/test.js"
  },
```
项目根目录创建一个 config-overrides.js 用于修改默认配置，文件位置如下：

```
+-- your-project
|   +-- config-overrides.js
|   +-- node_modules
|   +-- package.json
|   +-- public
|   +-- README.md
|   +-- src
```

## 3.2.安装eject
在创建工程项目后,由于没有传统的webpack.config文件首先安装eject生成自定义配置文件(注意,用eject生成webpack.config后该操作不能回滚,注意备份)

```
npm i eject
```

## 3.3.安装bable相关
[具体配置参考链接](https://babeljs.io/docs/en/babel-plugin-proposal-decorators)
```
npm install --save-dev @babel/core
npm install --save-dev @babel/plugin-proposal-class-properties
npm install --save-dev @babel/plugin-proposal-decorators
```

逐条安装完以上命令之后，package.json，如果在这里面设置的话，就不要重复新建
.babelrc文件了，否则会报【重复错误】
```
//package.json
"babel": {
    "plugins": [
      [
        "@babel/plugin-proposal-decorators",
        {
          "legacy": true
        }
      ],
      [
        "@babel/plugin-proposal-class-properties",
        {
          "loose": true
        }
      ]
    ],
    "presets": [
      "react-app"
    ]
  }
```
# 4.小试牛刀看下是否成功？
## 4.1.实现过程
安装上面的步骤，可以重新启动项目了

```
npm start
```
假设现在有一个父组件Father，一个子组件Child，在父组件中写被观察的数据、获取数据、设置数据、重置数据的方法，父组件代码如下：

```js
import React, {Component} from 'react';
// 引入 mobx
import {observable, computed, action} from "mobx";
// 引入子组件
import Child from "./Child.js";

class VM {
  @observable firstName = "";
  @observable lastName = "";

  @computed
  get fullName() {
    const {firstName, lastName} = this;
    if (!firstName && !lastName) {
      return "Please input your name!";
    } else {
      return firstName + " " + lastName;
    }
  }

  @action.bound
  setValue(key, event) {
    this[key] = event.target.value;
  }

  @action.bound
  doReset() {
    this.firstName = "";
    this.lastName = "";
  }
}

const vm = new VM();

export default class Father extends Component {

  render() {
    return (
     <Child vm={vm}/>
    )
  }
}
```

给子组件一个修饰符@observer
子组件代码如下

```js
import React, {Component} from 'react';
import {observer} from "mobx-react";


@observer
class Upload extends Component {
    render(){
        // 解构从父组件传来的数据
        const {vm} = this.props;
        return(
            <div>
              <h1>This is mobx-react!</h1>
              <p>
                First name:{" "}
                <textarea
                  type="text"
                  value={vm.firstName}
                  onChange={e => vm.setValue("firstName", e)}
                />
              </p>
              <p>
                Last name:{" "}
                <textarea
                  type="text"
                  value={vm.lastName}
                  onChange={e => vm.setValue("lastName", e)}
                />
              </p>
              <p>Full name: {vm.fullName}</p>
              <p>
                <button onClick={vm.doReset}>Reset</button>
              </p>
            </div>
        )        
    }
}
```
## 4.2.效果

![tab2.gif](https://upload-images.jianshu.io/upload_images/11189478-7eb178df84ead65e.gif?imageMogr2/auto-orient/strip)

# 5.mobx常用api
## 5.1.可观察的数据
### observable
observable：一种让数据的变化可以被观察的方法   

哪些数据可以被观察？  
JS基本数据类型、引用类型、普通对象、类实例、数组和映射

如下代码：

```js
const arr = observable(['a', 'b', 'c']);
const map = observable(new Map());
const obj = observable({
  a: 1,
  b: 1
});
```


### observable.box
observable.box：包装数值、布尔值、字符串

如下代码:

```js
let num = observable.box(20);
let str = observable.box('hello');
let bool = observable.box(true);

// 使用set()设置数值
num.set(50);

// get()获取原生数值
console.log(num.get(), str, bool);
// 50
// ObservableValue$$1 {name: "ObservableValue@5", isPendingUnobservation: false, isBeingObserved: false, observers: Set(0), diffValue: 0, …}
// ObservableValue$$1 {name: "ObservableValue@6", isPendingUnobservation: false, isBeingObserved: false, observers: Set(0), diffValue: 0, …}

```
### 使用修饰器
不使用修饰器是不是相对麻烦呢？，还要时刻记着变量的类型，而使用修饰器的话，内部会做一些变量判断转换，写法也更加的简洁。代码如下：
```
class Store {
  @observable array = [];
  @observable obj = {};
  @observable map = new Map();

  @observable string = 'hello';
  @observable number=20;
  @observable bool=false;
}

```
## 5.2.对可观察的数据做出反应
观察数据变化的方式：computed、autorun、when、Reaction  

**computed**：将多个可观察数据组合成一个可观察数据

**autorun**：重点了解一下，如果应用程序都是 **可观察数据** ，而应用程序渲染UI、写入缓存等动作都设置为autorun，我们是不是就可以安心写代码，只与数据状态打交道，从而实现数据统一管理的目标。  

**when**：提供了根据条件执行逻辑，是autorun的一种变种。

**reaction**：分离可观察数据声明，对autorun做出改进。

四个方法各有特点且互相补充

## 5.3.修改可观察数据（action）

之前有提到autorun,还有一个问题需要解决，那就是性能问题，如果数据众多，每一次小修改都会触发autorun，如下：

```js
import {observable, isArrayLike, computed, action, runInAction, autorun, when, reaction} from "mobx";

class Store {
  @observable array = [];
  @observable obj = {};
  @observable map = new Map();

  @observable string = 'hello';
  @observable number = 20;
  @observable bool = true;

  @computed get mixed() {
    return store.string + ':' + store.number;
  }
}

let store = new Store();

reaction(() => [store.string, store.number,store.bool], arr => console.log(arr.join('+')));

store.string = 'word';
store.number = 25;
store.bool = true;

// 打印结果
word+20+false
word+25+false
word+25+true
```
我们从上面的结果中得到，每次修改都会触发reaction，那么该怎么解决呢？    

可以使用action来解决这个问题。修改代码如下：

```js
import {observable, isArrayLike, computed, action, runInAction, autorun, when, reaction} from "mobx";

class Store {
  @observable array = [];
  @observable obj = {};
  @observable map = new Map();

  @observable string = 'hello';
  @observable number = 20;
  @observable bool = false;

  @action bar() {
    store.string = 'word';
    store.number = 333;
    store.bool = true;
  }

}

let store = new Store();

reaction(() => [store.string, store.number, store.bool], arr => console.log(arr.join('+')));

store.bar();

// 打印结果
word+333+true
```
使用action后发现，虽然修改了3个数据，但是只调用了一次reaction方法，做到了性能上的优化。所以当数据较多的时，建议使用action来更新数据。

# 6.mobx实现TodoList
功能如下：
- Todo条目的列表展示
- 增加Todo条目
- 修改完成状态
- 删除Todo条目

首先，创建一个TodoList文件夹，在目录下新建一个store.js文件，这个文件用来做数据的处理。

```js
// store.js
import {observable, computed, action} from "mobx";

class Todo {
  id = Math.random();
  @observable title = '';
  @observable finished = false;

  constructor(title) {
    this.title = title;
  }

  @action.bound toggle() {
    this.finished = !this.finished;
  }

}

class Store {
  @observable todos = [];

  @action.bound createTodo(title) {
    this.todos.unshift(new Todo(title))
  }

  @action.bound removeTode(todo) {
    // remove不是原生的方法，是mobx提供的
    this.todos.remove(todo);
  }

  @computed get left() {
    return this.todos.filter(item => !item.finished).length;
  }
}

var store = new Store();


export default store;
```
新建一个TodoList.js文件

```
import React, {Component} from 'react';
import PropTypes from 'prop-types';
import {observer, PropTypes as ObservablePropTypes} from 'mobx-react';
import TodoItem from "./TodoItem";


@observer
class TodoList extends Component {
  // 属性类型要在全局这里定义
  static propTypes = {
    store: PropTypes.shape({
      createTodo: PropTypes.func,
      todos: ObservablePropTypes.observableArrayOf(ObservablePropTypes.observableObject).isRequired
    }).isRequired
  };

  state = {
    inputTile: ""
  };

  handleSubmit = (e) => {
    // 表单提交，阻止整个页面被提交
    e.preventDefault();
    let {store} = this.props;
    let {inputTile} = this.state;

    store.createTodo(inputTile);

    // 创建完新的条目之后，要清空输入框
    this.setState({
      inputTile: ""
    })
  };

  handleChange = (e) => {
    var inputTile = e.target.value;
    this.setState({
      inputTile
    })
  };

  render() {
    let {inputTile} = this.state;
    let {store} = this.props;
    return <div className="todoList">
      <header>
        <form onSubmit={this.handleSubmit}>
          <input type="text"
                 onChange={this.handleChange}
                 value={inputTile}
                 placeholder="你想要到哪里去？"
                 className="input"
          />
        </form>
      </header>
      <ul>
        {
          store.todos.map((item) => {
            return (
              <li key={item.id}>
                <TodoItem todo={item}/>
                <span onClick={()=>{store.removeTode(item)}}>删除</span>
              </li>
            )
          })
        }
      </ul>
      <footer>
        {store.left} 项 未完成
      </footer>
    </div>
  }
}

export default TodoList
```
新建一个TodoItem.js文件

```js
import React, {Component} from 'react';
import {observer} from 'mobx-react';
import PropTypes from 'prop-types';

@observer
class TodoItem extends Component {
  // 从父组件传来的属性类型要在全局这里定义，做一些限制
  static propTypes = {
    todo: PropTypes.shape({
      id: PropTypes.number.isRequired,
      title: PropTypes.string.isRequired,
      finished: PropTypes.bool.isRequired
    }).isRequired
  };

  handleClick = (e) => {
    let {todo} = this.props;
    todo.toggle();
  }

  render() {
    // 这里的Item是一个对象
    let {todo} = this.props;
    return (
      <div>
        <input
          type="checkbox"
          checked={todo.finished}
          onChange={this.handleClick}
        />
        <span className="title">{todo.title}</span>
      </div>
    )
  }
}

export default TodoItem;
```
在你组件中引入TodoList组件、以及store.js

```js

import React, {Component} from 'react';

import TodoList from "../../component/TodoList/TodoList";
import store from "../../store/store";

class PictureResources extends Component {
  render() {
    return (
      <TodoList store={store}/>
    )
  }
}

export default PictureResources
```
