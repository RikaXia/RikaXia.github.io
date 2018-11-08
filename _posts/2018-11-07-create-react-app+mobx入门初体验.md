---
layout:     post
title:      create-react-app+mobx入门初体验
subtitle:   create-react-app配置装饰器环境
date:       2018-11-07
author:     Rika
header-img: img/post-bg-keybord.jpg
catalog: true
tags:
    - React
---  
  
# 前言
在开始使用mobox，我们还需要纠结一个东西，就是配置环境启用ES7的修饰器语法，当然，如果你不需要修饰器，可以跳过这一部分。如果是新手的话，建议配置。

# 一定需要修饰器吗？
官方的MobX文档也说明了，不一定要使用修饰器，如果有人说你必须在MobX中使用decorator，那就不是真的，你也可以使用普通函数，如下：

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
## 使用和不使用修饰符的对比
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

## 使用修饰符的优缺点

**使用修饰符的优点：**
- 样板文件最小化，声明式代码。
- 易于使用和阅读。大多数 MobX 用户都在使用。 

**使用修饰符的缺点：**
- ES.next 2阶段特性。
- 需要设置和编译，目前只有 Babel/Typescript 编译器支持。

# create-react-app+mobx（装饰器）
create-react-app 目前还没有内置的装饰器支持，所以此小结要解决这个问题。

## 安装react-app-rewire相关
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

## 安装eject
在创建工程项目后,由于没有传统的webpack.config文件首先安装eject生成自定义配置文件(注意,用eject生成webpack.config后该操作不能回滚,注意备份)

```
npm i eject
```

## 安装bable相关
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
# 小试牛刀看下是否成功？
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
     <Upload vm={vm}/>
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
## 效果

![tab2.gif](https://upload-images.jianshu.io/upload_images/11189478-7eb178df84ead65e.gif?imageMogr2/auto-orient/strip)
