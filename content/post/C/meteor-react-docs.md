---
title: Meteor React 文档
date: 2021-10-15
draft: true
categories: ["技术文档"]
tags: ["Meteor", "React", "翻译"]
comment: false
---

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
<p style="text-align:center;font-size:1.3em">文档目录结构</p>

- [1 创建app](#1-创建app)
    - [1.1 安装 Meteor](#11-安装-meteor)
    - [1.2 创建 Meteor 项目](#12-创建-meteor-项目)
    - [1.3 创建任务组件](#13-创建任务组件)
    - [1.4 创建任务示例](#14-创建任务示例)
    - [1.5 渲染任务示例](#15-渲染任务示例)
    - [1.6 移动端外观](#16-移动端外观)
    - [1.7 更换热模块](#17-更换热模块)
- [2 集合](#2-集合)
    - [2.1 创建任务集合](#21-创建任务集合)
    - [2.2 初始化任务集](#22-初始化任务集)
    - [2.3 渲染任务集合](#23-渲染任务集合)
- [3 表单和事件](#3-表单和事件)
    - [3.1 创建任务表单](#31-创建任务表单)
    - [3.2 更新App组件](#32-更新app组件)
    - [3.3 更新样式表](#33-更新样式表)
    - [3.4 添加提交处理程序](#34-添加提交处理程序)
    - [3.5 首先显示最新的任务](#35-首先显示最新的任务)
- [4 更新和删除](#4-更新和删除)
    - [4.1 添加复选框](#41-添加复选框)
    - [4.2 切换复选框](#42-切换复选框)
    - [4.3 删除任务](#43-删除任务)
- [5 样式](#5-样式)
    - [5.1 CSS](#51-css)
    - [5.2 应用样式](#52-应用样式)
- [6 筛选任务](#6-筛选任务)
    - [6.1 useState](#61-usestate)
    - [6.2 按钮样式](#62-按钮样式)
    - [6.3 过滤任务](#63-过滤任务)
    - [6.4 Meteor开发工具插件](#64-meteor开发工具插件)
    - [6.5 待办任务](#65-待办任务)
- [7 添加用户账户](#7-添加用户账户)
    - [7.1 密码认证](#71-密码认证)
    - [7.2 创建用户账户](#72-创建用户账户)
    - [7.3 登录表单](#73-登录表单)
    - [7.4 要求认证](#74-要求认证)
    - [7.5 登录表单样式](#75-登录表单样式)
    - [7.6 服务器启动](#76-服务器启动)
    - [7.7 任务所有者](#77-任务所有者)
    - [7.8 退出登录](#78-退出登录)
- [8 方法](#8-方法)
    - [8.1 禁用快速原型技术](#81-禁用快速原型技术)
    - [8.2 添加任务方法](#82-添加任务方法)
    - [8.3 实现方法调用](#83-实现方法调用)
    - [8.4 api和db文件夹](#84-api和db文件夹)
- [9 发布](#9-发布)
    - [9.1 自动发布](#91-自动发布)
    - [9.2 任务发布](#92-任务发布)
    - [9.3 任务订阅](#93-任务订阅)
    - [9.4 加载状态](#94-加载状态)
    - [9.5 检查用户权限](#95-检查用户权限)
- [10 在移动端运行](#10-在移动端运行)
    - [10.1 iOS 模拟器](#101-ios-模拟器)
    - [10.2 安卓模拟器](#102-安卓模拟器)
    - [10.3 安卓设备](#103-安卓设备)
    - [10.4 iPhone or iPad](#104-iphone-or-ipad)
- [11 测试](#11-测试)
    - [11.1 安装依赖](#111-安装依赖)
    - [11.2 脚手架测试](#112-脚手架测试)
    - [11.3 数据准备](#113-数据准备)
    - [11.4 测试任务删除](#114-测试任务删除)
    - [11.5 更多的测试](#115-更多的测试)
- [12 部署](#12-部署)
    - [12.1 创建账号](#121-创建账号)
    - [12.2 部署应用](#122-部署应用)
    - [12.3 访问并享受吧](#123-访问并享受吧)
- [13 后续](#13-后续)

<!-- markdown-toc end -->

> 本文档翻译自 [Meteor React Tutorial](https://react-tutorial.meteor.com)

在本教程中，我们将使用 React 和 Meteor 平台建立一个简单的待办事项应用。Meteor 与其他一些框架如 Blaze、Angular 和 Vue 都可以开箱即用。

我们推荐你在这里查看我们所有的教程。现在，让我们开始用 Meteor 构建你的 React To-Do应用程序吧!

# 1 创建app

## 1.1 安装 Meteor

首先我们需要安装 Meteor。按照接下来文档中的步骤安装最新的 Meteor 官方发布版本。

## 1.2 创建 Meteor 项目

用 React 设置 Meteor 的最简单的方法是使用 `meteor create` 命令，并加上选项 `--react` 和你的项目名称（你也可以省略 `--react` 选项，因为它是默认的）。

    meteor create simple-todos-react

Meteor 将为你创建所有必要的文件。

位于客户端目录下的文件正在设置你的客户端（web），你可以看到例如 client/main.jsx，Meteor 正在将你的应用程序的主要组件渲染成 HTML。

另外，检查服务器目录，Meteor 正在设置服务器端（Node.js），你可以看到 server/main.js 正在用一些数据初始化你的 MongoDB 数据库。你不需要安装 MongoDB，因为 Meteor 提供了它的嵌入式版本供你使用。

现在你可以用以下方式运行你的 Meteor 应用程序。

    meteor run

别担心，从现在开始，Meteor会让你的应用程序与你的所有变化保持同步。

你的React代码将位于 import/ui 目录下，而 App.jsx 文件是你的 React To-do应用的根组件。

快速浏览一下 Meteor 创建的所有文件，你现在不需要理解它们，但知道它们在哪里是很好的。

## 1.3 创建任务组件

现在你将进行第一次修改。在你的 ui 文件夹中创建一个名为 Task.jsx 的新文件。

这个文件将导出一个名为 Task 的 React组件，它将代表你的待办事项列表中的一个任务。

**imports/ui/Task.jsx**

    import React from 'react';
    
    export const Task = ({ task }) => {
      return <li>{task.text}</li>
    };

由于这个组件将在一个列表中，你将返回一个li元素。

## 1.4 创建任务示例

由于你还没有连接到你的服务器和数据库，让我们定义一些样本数据，这些数据很快就会被用来呈现一个任务列表。它将是一个数组，你可以称它为tasks。

**imports/ui/App.jsx**

    import React from 'react';
    
    const tasks = [
      {_id: 1, text: 'First Task'},
      {_id: 2, text: 'Second Task'},
      {_id: 3, text: 'Third Task'},
    ];
    
    export const App = () => ...

你可以将任何内容作为每个任务上的text属性。要有创造性!

## 1.5 渲染任务示例

现在我们可以用React实现一些简单的渲染逻辑。我们现在可以使用我们之前的任务组件来渲染我们的列表项。

在React中，你可以使用{ }来在它们之间编写Javascript代码。

请看下面，你将使用数组对象的.map函数来迭代你的样本任务。

**imports/ui/App.jsx**

    import React from 'react';
    import { Task } from './Task';
    
    const tasks = ..;
    
    export const App = () => (
      <div>
        <h1>Welcome to Meteor!</h1>
    
        <ul>
          { tasks.map(task => <Task key={ task._id } task={ task }/>) }
        </ul>
      </div>
    );

记得给你的任务添加 `key` 属性，否则React会发出警告，因为它将看到许多相同类型的组件作为兄弟节点。如果没有键，React将很难在必要时重新渲染其中一个。

> 你可以在[这里](https://reactjs.org/docs/lists-and-keys.html#keys)阅读更多关于React和键的信息。

从你的App组件中删除 Hello 和 Info，记得也要删除文件顶部的导入。同时删除 Hello.jsx 和 Info.jsx 文件。

## 1.6 移动端外观

让我们来看看你的应用程序在移动端上的表现如何。你可以通过在浏览器中右击你的应用程序来模拟移动环境（我们假设你使用的是谷歌浏览器，因为它是当今最流行的浏览器），然后检查，这将在你的浏览器中打开一个名为 "开发工具 "的新窗口。在开发工具中，你有一个小图标，显示一个移动设备和一个平板电脑：

<center><img src="/image/step01-dev-tools-mobile-toggle.png" width="100%" /></center>

点击它，然后在顶部栏中选择你想模拟的手机。

> 你也可以在你的手机上检查你的应用程序。要做到这一点，在你的手机浏览器的导航浏览器中使用你的本地IP连接到你的应用程序。
> 在Unix系统中，这个命令至少可以为你打印出你的本地IP: `ifconfig | grep "inet " | grep -Fv 127.0.0.1 | awk '{print $2}'`

你会看到类似这样的内容。

<center><img src="/image/step01-mobile-without-meta-tags.png" width="20%" /></center>

正如你所看到的，一切都很小，因为我们没有为移动设备调整视图端口。你可以通过在你的 client/main.html 文件中的 head标签内、title标签后添加下面这几行来解决这个问题和其他类似问题。

**client/main.html**

    <meta charset="utf-8"/>
    <meta http-equiv="x-ua-compatible" content="ie=edge"/>
    <meta
      name="viewport"
      content="width=device-width, height=device-height, viewport-fit=cover, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no"
    />
    <meta name="mobile-web-app-capable" content="yes"/>
    <meta name="apple-mobile-web-app-capable" content="yes"/>

现在你的应用程序应该看起来像这样:

<center><img src="/image/step01-mobile-with-meta-tags.png" width="20%" /></center>

## 1.7 更换热模块

Meteor默认在使用React时已经为你添加了一个叫做 hot-module-replacement的包。这个包会更新正在运行的应用程序中在重建过程中被修改的javascript模块。缩短了开发时的反馈周期，所以你可以更快地查看和测试变化（它甚至会在构建完成之前更新应用）。你也不会丢失状态，你的应用代码会被更新，你的状态也会是一样的。

你还应该在这时添加 dev-error-overlay包，这样你就可以在你的网络浏览器中看到错误。

    meteor add dev-error-overlay

你可以试着犯一些错误，然后你就会在浏览器中看到错误，而不仅仅是在控制台中。

> 回顾：你可以在[这里](https://github.com/meteor/react-tutorial/tree/master/src/simple-todos/step01)检查你的代码在当前步骤结束时应该是怎样的。

下一步，我们将与 MongoDB数据库合作，以存储我们的任务。

# 2 集合

Meteor已经为你设置了 MongoDB。为了使用数据库，我们需要创建一个集合，这就是存储文件的地方，在我们的例子中就是 tasks。

> 你可以在[这里](http://guide.meteor.com/collections.html)阅读更多关于集合的信息。

在这一步中，我们将实现所有必要的代码，使用React钩子(hooks)为我们的任务建立和运行一个基本集合。

## 2.1 创建任务集合

我们可以通过在 import/api/TasksCollection.js 创建一个新文件来存储我们的任务，该文件实例化了一个新的 Mongo集合并将其导出。

**imports/api/TasksCollection.js**

    import { Mongo } from 'meteor/mongo';
    export const TasksCollection = new Mongo.Collection('tasks');

注意，我们把文件存放在 imports/api目录下，这是一个存放API相关代码的地方，比如出版物和方法。你可以随心所欲地命名这个文件夹，这只是一种选择而已。

你可以删除这个文件夹中的 link.js文件，因为我们不打算使用这个集合。

> 你可以在[这里](http://guide.meteor.com/structure.html)阅读更多关于应用程序结构和 imports/exports 的信息。

## 2.2 初始化任务集

为了让我们的集合发挥作用，你需要在服务器中导入它，以便它设置一些管道。

你可以使用 `import "/imports/api/TasksCollection"` 或者 `import { TasksCollection } from "/imports/api/TasksCollection"` 如果你要在同一个文件上使用，但要确保它被导入。

现在很容易检查我们的集合中是否有数据，否则我们也可以轻松地插入一些样本数据。

你不需要保留 server/main.js的旧内容。

**server/main.js**

    import { Meteor } from 'meteor/meteor';
    import { TasksCollection } from '/imports/api/TasksCollection';
    
    const insertTask = taskText => TasksCollection.insert({ text: taskText });
    
    Meteor.startup(() => {
      if (TasksCollection.find().count() === 0) {
        [
          'First Task',
          'Second Task',
          'Third Task',
          'Fourth Task',
          'Fifth Task',
          'Sixth Task',
          'Seventh Task'
        ].forEach(insertTask)
      }
    });

因此，你正在导入 TasksCollection，并在其上添加一些任务：迭代一个字符串数组，对每个字符串调用一个函数，在我们的任务文件中插入这个字符串作为我们的文本字段。

## 2.3 渲染任务集合

现在有趣的部分来了，你将使用一个React函数组件和一个叫做 useTracker的Hook从一个叫做 react-meteor-data的包中渲染任务。

> Meteor与 Meteor包和 NPM包一起工作，通常 Meteor包是使用 Meteor的内部程序或其他 Meteor包。

这个包已经包含在React的骨架(meteor create yourproject)，所以你不需要添加它，但你可以随时运行 `meteor add package-name` 添加 Meteor包。

    meteor add react-meteor-data

现在你已经准备好从这个包中导入代码了，当从 Meteor包中导入代码时，与NPM模块唯一不同的是，你需要在导入的 from部分前加上 meteor/。

react-meteor-data 导出的 useTracker函数是一个 React Hook，允许你的React组件作出反应。每当数据发生变化时，你的组件将重新渲染。很酷，对吗？

> 关于React Hooks的更多信息，请阅读[这里](https://reactjs.org/docs/hooks-faq.html)。

**imports/ui/App.jsx**

    import React from 'react';
    import { useTracker } from 'meteor/react-meteor-data';
    import { TasksCollection } from '/imports/api/TasksCollection';
    import { Task } from './Task';
    
    export const App = () => {
      const tasks = useTracker(() => TasksCollection.find({}).fetch());
    
      return (
        <div>
          <h1>Welcome to Meteor!</h1>
    
          <ul>
            { tasks.map(task => <Task key={ task._id } task={ task }/>) }
          </ul>
        </div>
      );
    };

看看你的应用程序现在应该是什么样子:

<center><img src="/image/step02-tasks-list.png" width="20%" /></center>

你可以在服务器上改变你的 MongoDB的数据，你的应用程序将为你做出反应并重新渲染。

你可以从你的应用程序文件夹或使用 Mongo UI客户端，如 NoSQLBooster，在终端通过运行 `meteor mongo` 连接到的MongoDB。你的嵌入式 MongoDB在 3001端口运行。

请看如何连接：

<center><img src="/image/step02-connect-mongo.png" width="100%" /></center>

看看数据库：

<center><img src="/image/step02-see-your-db.png" width="100%" /></center>

你可以双击你的集合，以查看存储在其中的文件：

<center><img src="/image/step02-see-your-collection.png" width="100%" /></center>

但是，等等，我的任务是如何从服务器传到客户端的？我们将在后面关于出版物和订阅的步骤中解释这个问题。你现在需要知道的是，你正在把数据库中的所有数据发布到客户端。这一点以后会被删除，因为我们不想一直发布所有的数据。

> 回顾：你可以在[这里](https://github.com/meteor/react-tutorial/tree/master/src/simple-todos/step02)检查你的代码在这一步骤结束时应该是怎样的

在下一步，我们将使用一个表单来创建任务。

# 3 表单和事件
所有的应用程序都需要允许用户与存储的数据进行某些类型的交互。在我们的案例中，第一种类型的交互是插入新的任务。没有它，我们的 To-Do应用就不会有什么帮助。

用户在网站上插入或编辑数据的主要方式之一是通过表单。在大多数情况下，使用 `<form>` 标签是个好主意，因为它为里面的元素赋予了语义。

## 3.1 创建任务表单
首先，我们需要创建一个简单的表单组件来封装我们的逻辑。如你所见，我们设置了 `useState` React Hook。

请注意数组的解构`[text, setText]`，其中`text`是我们要使用的存储值，在这里是一个字符串；而`setText`是一个用来更新该值的函数。

在你的`ui`文件夹中创建一个新文件 `TaskForm.jsx`。

**imports/ui/TaskForm.jsx**

    import React, { useState } from 'react';
     
    export const TaskForm = () => {
      const [text, setText] = useState("");
     
      return (
        <form className="task-form">
          <input
            type="text"
            placeholder="Type to add new tasks"
          />
     
          <button type="submit">Add Task</button>
        </form>
      );
    };

## 3.2 更新App组件
然后，我们可以简单地将其添加到你的任务列表上方的App组件中。

**imports/ui/App.jsx**

    import { useTracker } from 'meteor/react-meteor-data';
    import { Task } from './Task';
    import { TasksCollection } from '/imports/api/TasksCollection';
    import { TaskForm } from './TaskForm';
     
    export const App = () => {
      const tasks = useTracker(() => TasksCollection.find({}).fetch());

      return (
        <div>
          <h1>Welcome to Meteor!</h1>

          <TaskForm/>

          <ul>
            { tasks.map(task => <Task key={ task._id } task={ task }/>) }
          </ul>
        </div>
      );
    };

## 3.3 更新样式表
你也可以按照你的意愿来设计它的样式。现在，我们只需要在顶部留出一些空白，这样表格就不会显得不伦不类。添加CSS类`.task-form`，这需要与表单组件中的 `className`属性的名称相同。

**client/main.css**

    .task-form {
      margin-top: 1rem;
    }

## 3.4 添加提交处理程序
现在你可以使用`onSubmit`事件给你的表单附加一个提交处理程序；同时将你的React钩子插入输入元素中的`onChange`事件。

正如你所看到的，你正在使用`useState` React钩子来存储你的`<input>`元素的值。注意，你还需要将你的`value`属性设置为`text`常量，这将使`input`元素与我们的钩子保持同步。

> 在更复杂的应用中，如果在潜在的频繁事件（如onChange）之间有许多计算发生，你可能想实现一些去噪或节流逻辑。有一些库可以帮助你做到这一点，比如说 `Lodash`。

**imports/ui/TaskForm.jsx**

    import React, { useState } from 'react';
    import { TasksCollection } from '/imports/api/TasksCollection';
     
    export const TaskForm = () => {
      const [text, setText] = useState("");
      
      const handleSubmit = e => {
        e.preventDefault();

        if (!text) return;

        TasksCollection.insert({
          text: text.trim(),
          createdAt: new Date()
        });

        setText("");
      };
     
      return (
        <form className="task-form" onSubmit={handleSubmit}>
          <input
            type="text"
            placeholder="Type to add new tasks"
            value={text}
            onChange={(e) => setText(e.target.value)}
          />
     
          <button type="submit">Add Task</button>
        </form>
      );
    };

同时在你的`task`文件中插入日期`createdAt`，这样你就能知道每个任务是什么时候创建的。

## 3.5 首先显示最新的任务
现在你只需要做一个能让用户满意的改变：我们需要先显示最新的任务。我们可以通过对我们的Mongo查询进行排序来快速完成。

**imports/ui/App.jsx**

    ..
     
    export const App = () => {
      const tasks = useTracker(() => TasksCollection.find({}, { sort: { createdAt: -1 } }).fetch());
    ..

你的app应该看起来像这样：

<center><img src="/image/step03-form-new-task.png" width="20%" style="display: inline;" />
<img src="/image/step03-new-task-on-list.png" width="20%" style="display: inline;" /></center>

> 回顾：你可以在[这里](https://github.com/meteor/react-tutorial/tree/master/src/simple-todos/step03)检查你的代码在这一步骤结束时应该是怎样的。

在下一步，我们将更新你的任务状态，并为用户提供一个删除任务的方法。

# 4 更新和删除
到目前为止，你只是在我们的集合中插入了文件。让我们来看看如何通过与用户界面的交互来更新和删除它们。

## 4.1 添加复选框
首先，你需要给你的任务组件添加一个复选框元素。

> 请确保添加了 `readOnly` 属性，因为我们不使用 `onChange` 来更新状态。
>
> 我们还必须将我们的 `checked` 属性强制设为布尔值，因为React理解 `undefined` 的值是不存在的，因此会导致组件从不受控的状态切换到受控状态。
>
> 我们还邀请你做更多的实践，看看这个app是如何表现的，以便学习。

你还需要接收一个回调，一个当复选框被点击时将被调用的函数。

**imports/ui/Task.jsx**

    import React from 'react';

    export const Task = ({ task, onCheckboxClick }) => {
      return (
        <li>
          <input
            type="checkbox"
            checked={!!task.isChecked}
            onClick={() => onCheckboxClick(task)}
            readOnly
          />
          <span>{task.text}</span>
        </li>
      );
    };

## 4.2 切换复选框
现在你可以更新你的任务文件，切换其isChecked字段。

创建一个函数来改变你的文档，并将其传递给你的任务组件。

**imports/ui/App.jsx**

    const toggleChecked = ({ _id, isChecked }) => {
      TasksCollection.update(_id, {
        $set: {
          isChecked: !isChecked
        }
      })
    };

    export const App = () => {
      ..
      <ul>
        { tasks.map(task => <Task key={ task._id } task={ task } onCheckboxClick={toggleChecked} />) }
      </ul>
      ..

你的app应该看起来像这样:

<center><img src="/image/step04-checkbox.png" width="20%" /></center>

## 4.3 删除任务
你只需要几行代码就可以删除任务。

首先在你的任务组件中的文本后添加一个按钮，并接收一个回调函数。

**imports/ui/Task.jsx**

import React from 'react';

    export const Task = ({ task, onCheckboxClick, onDeleteClick }) => {
      return (
    ..
          <span>{task.text}</span>
          <button onClick={ () => onDeleteClick(task) }>&times;</button>
    ..

现在在app中添加删除逻辑，你需要有一个删除任务的函数，并在任务组件的回调属性中提供这个函数。

**imports/ui/App.jsx**

    const deleteTask = ({ _id }) => TasksCollection.remove(_id);

    export const App = () => {
      ..
      <ul>
        { tasks.map(task => <Task
          key={ task._id }
          task={ task }
          onCheckboxClick={toggleChecked}
          onDeleteClick={deleteTask}
        />) }
      </ul>
      ..
    }

你的app应该看起来像这样:

<center><img src="/image/step04-delete-button.png" width="20%" /></center>

> 回顾：你可以在[这里](https://github.com/meteor/react-tutorial/tree/master/src/simple-todos/step04)检查你的代码在这一步骤结束时应该是怎样的

在下一步，我们将使用 CSS 和 Flexbox 来改善你的app的外观。

# 5 样式

## 5.1 CSS
到目前为止，我们的用户界面看起来相当难看。让我们添加一些基本的样式，这将作为一个更专业的app的基础。

用下面的文件替换 `client/main.css`文件的内容，我们的想法是在顶部有一个应用栏，和一个可滚动的内容，包括:

- 添加新任务的表单
- 任务列表

**client/main.css**

    body {
      font-family: sans-serif;
      background-color: #315481;
      background-image: linear-gradient(to bottom, #315481, #918e82 100%);
      background-attachment: fixed;

      position: absolute;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;

      padding: 0;
      margin: 0;

      font-size: 14px;
    }

    button {
      font-weight: bold;
      font-size: 1em;
      border: none;
      color: white;
      box-shadow: 0 3px 3px rgba(34, 25, 25, 0.4);
      padding: 5px;
      cursor: pointer;
    }

    button:focus {
      outline: 0;
    }

    .app {
      display: flex;
      flex-direction: column;
      height: 100vh;
    }

    .app-header {
      flex-grow: 1;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }

    .main {
      display: flex;
      flex-direction: column;
      flex-grow: 1;
      overflow: auto;
      background: white;
    }

    .main::-webkit-scrollbar {
      width: 0;
      height: 0;
      background: inherit;
    }

    header {
      background: #d2edf4;
      background-image: linear-gradient(to bottom, #d0edf5, #e1e5f0 100%);
      padding: 20px 15px 15px 15px;
      position: relative;
      box-shadow: 0 3px 3px rgba(34, 25, 25, 0.4);
    }

    .app-bar {
      display: flex;
      justify-content: space-between;
    }

    .app-bar h1 {
      font-size: 1.5em;
      margin: 0;
      display: inline-block;
      margin-right: 1em;
    }

    .task-form {
      display: flex;
      margin: 16px;
    }

    .task-form > input {
      flex-grow: 1;
      box-sizing: border-box;
      padding: 10px 6px;
      background: transparent;
      border: 1px solid #aaa;
      width: 100%;
      font-size: 1em;
      margin-right: 16px;
    }

    .task-form > input:focus {
      outline: 0;
    }

    .task-form > button {
      min-width: 100px;
      height: 95%;
      background-color: #315481;
    }

    .tasks {
      list-style-type: none;
      padding-inline-start: 0;
      padding-left: 16px;
      padding-right: 16px;
      margin-block-start: 0;
      margin-block-end: 0;
    }

    .tasks > li {
      display: flex;
      padding: 16px;
      border-bottom: #eee solid 1px;
    }

    .tasks > li > span {
      flex-grow: 1;
    }

    .tasks > li > button {
      justify-self: flex-end;
      background-color: #ff3046;
    }

> 如果你想了解更多关于这个样式表的信息，请查看这篇关于[Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)的文章，以及 [Wes Bos](https://twitter.com/wesbos) 关于它的免费[视频教程](https://flexbox.io/)。
> 
> Flexbox是一个很好的工具，可以在你的用户界面中分配和对齐元素。

## 5.2 应用样式
现在你需要在你的组件周围添加一些元素。你要在 `App`中的主`div`上添加一个 `className`，还要在 `h1`周围添加一个带有几个 `div`的 `header`元素，并在表单和列表周围添加一个主`div`。看看下面应该是怎样添加的，注意类的名称，它们需要与CSS文件中的相同:

**imports/ui/App.jsx**

    ..
      return (
        <div className="app">
          <header>
            <div className="app-bar">
              <div className="app-header">
                <h1>Welcome to Meteor!</h1>
              </div>
            </div>
          </header>

          <div className="main">
            <TaskForm />

            <ul className="tasks">
              {tasks.map(task => (
                <Task
                  key={task._id}
                  task={task}
                  onCheckboxClick={toggleChecked}
                  onDeleteClick={deleteTask}
                />
              ))}
            </ul>
          </div>
        </div>
      );

> 在React中，我们使用 `className` 而不是 `class`，因为 React使用 Javascript 来定义用户界面，而 `class`是 Javascript中的一个保留词。

另外，为你的app选择一个更好的标题，Meteor很了不起，但你不希望在你的应用程序顶部栏中一直看到 `Welcome to Meteor!`。

你可以选择类似下面的内容:

**imports/ui/App.jsx**

    ..
      <h1>📝️ To Do List</h1>
    ..

你的app应该看起来像这样:

<center><img src="/image/step05-styles.png" width="20%" /></center>

> 回顾：你可以在[这里](https://github.com/meteor/react-tutorial/tree/master/src/simple-todos/step05)检查你的代码在这一步骤结束时应该是怎样的

在下一步，我们将使这个任务列表更具交互性，例如，提供一种过滤任务的方法。

# 6 筛选任务
在这一步，你将按状态过滤你的任务，并显示待办任务的数量。

## 6.1 useState
首先，你要添加一个按钮来显示或隐藏列表中的已完成任务。

来自 React 的 `useState`函数是保持这个按钮状态的最好方法。它返回一个有两个元素的数组，其中第一个元素是状态的值，第二个是一个setter函数，是你要更新状态的方式。你可以使用数组析构来获得这两项的返回，并且已经为它们声明了一个变量。

请记住，用于常量的名字不属于React API，你可以随心所欲地命名它们。

同时在任务表单下面添加一个按钮，它将根据当前状态显示不同的文本。

**imports/ui/App.jsx**

    import React, { useState } from 'react';
    ..
    export const App = () => {
      const [hideCompleted, setHideCompleted] = useState(false);
     
      ..
        <div className="main">
          <TaskForm />
           <div className="filter">
             <button onClick={() => setHideCompleted(!hideCompleted)}>
               {hideCompleted ? 'Show All' : 'Hide Completed'}
             </button>
           </div>
      ..

你可以在[这里](https://reactjs.org/docs/hooks-state.html)阅读更多关于 `useState` hook 的信息。

我们建议你把钩子总是添加在组件的顶部，这样会更容易避免一些问题，比如总是以相同的顺序运行它们。

## 6.2 按钮样式
你应该给按钮添加一些样式，这样它就不会显得灰暗和没有良好的对比度。你可以使用下面的样式作为参考：

**client/main.css**

    .filter {
      display: flex;
      justify-content: center;
    }

    .filter > button {
      background-color: #62807e;
    }

## 6.3 过滤任务
现在，如果用户只想看到待处理的任务，你可以在 Mini Mongo 查询中给你的选择器添加一个过滤器，你需要得到所有不是 isChecked=true 的任务。

**imports/ui/App.jsx**

    ..
      const hideCompletedFilter = { isChecked: { $ne: true } };

      const tasks = useTracker(() =>
        TasksCollection.find(hideCompleted ? hideCompletedFilter : {}, {
          sort: { createdAt: -1 },
        }).fetch()
      );
    ..

## 6.4 Meteor开发工具插件
你可以安装一个扩展来可视化你在 Mini Mongo 中的数据。

Meteor DevTools Evolved 将帮助你调试你的app，因为你可以看到 Mini Mongo 上有哪些数据。

<center><img src="/image/step06-extension.png" width="100%" /></center>

你还可以看到 Meteor 从服务器上发送和接收的所有信息，这对你了解 Meteor 的工作方式很有帮助。

<center><img src="/image/step06-ddp-messages.png" width="100%" /></center>

使用[此链接](https://chrome.google.com/webstore/detail/meteor-devtools-evolved/ibniinmoafhgbifjojidlagmggecmpgf)在你的谷歌浏览器中安装它。

## 6.5 待办任务
更新App组件，以便在应用栏中显示待办任务的数量。

当没有待办任务时，你应该避免在你的应用栏中添加零。

**imports/ui/App.jsx**

    ..
      const pendingTasksCount = useTracker(() =>
        TasksCollection.find(hideCompletedFilter).count()
      );

      const pendingTasksTitle = `${
        pendingTasksCount ? ` (${pendingTasksCount})` : ''
      }`;
    ..

        <h1>
          📝️ To Do List
          {pendingTasksTitle}
        </h1>
    ..

你可以在同一个 useTracker 中进行这两种查找，然后返回一个具有这两种属性的对象，但是为了有一个更容易理解的代码，我们在这里创建了两个不同的跟踪器。

你的app应该看起来像这样：

<center><img src="/image/step06-all.png" width="20%" style="display: inline;" />
<img src="/image/step06-filtered.png" width="20%" style="display: inline;" /></center>

> 回顾：你可以在[这里](https://github.com/meteor/react-tutorial/tree/master/src/simple-todos/step06)检查你的代码在这一步骤结束时应该是怎样的

在下一步，我们将在你的app中包含用户访问。

# 7 添加用户账户

## 7.1 密码认证
Meteor 已经配备了一个基本的认证和账户管理系统，所以你只需要添加 accounts-password 来启用用户名和密码认证。

    meteor add accounts-password

> 支持的认证方法还有很多。你可以在[这里](https://docs.meteor.com/api/accounts.html)阅读更多关于账户系统的信息。

我们还建议你安装 `bcrypt` node模块，否则你会看到一个警告，说你正在使用它的纯 JavaScript 实现。

    meteor npm install --save bcrypt

> 你应该总是使用 `meteor npm`，而不是只使用 `npm`，所以你总是使用 Meteor 绑定的 `npm` 版本，这有助于你避免由于不同版本的npm安装不同的模块而产生的问题。

## 7.2 创建用户账户
现在你可以为我们的app创建一个默认用户，使用 `meteorite` 作为用户名，如果我们没有在数据库中找到它，就在服务器启动时创建一个新用户。

**server/main.js**

    import { Meteor } from 'meteor/meteor';
    import { Accounts } from 'meteor/accounts-base';
    import { TasksCollection } from '/imports/api/TasksCollection';

    ..

    const SEED_USERNAME = 'meteorite';
    const SEED_PASSWORD = 'password';

    Meteor.startup(() => {
      if (!Accounts.findUserByUsername(SEED_USERNAME)) {
        Accounts.createUser({
          username: SEED_USERNAME,
          password: SEED_PASSWORD,
        });
      }
      ..
    });

在你的app用户界面中，你应该还没有看到任何变化。

## 7.3 登录表单
你需要为用户提供一种方法来输入密钥并进行认证，为此我们需要一个表单。

我们可以使用 `useState` 钩子实现它。创建一个名为 `LoginForm.jsx` 的新文件并在其中添加一个表单。你应该使用 `Meteor.loginWithPassword(username, password);` 来用所提供的输入来验证你的用户。

**imports/ui/LoginForm.jsx**

    import { Meteor } from 'meteor/meteor';
    import React, { useState } from 'react';

    export const LoginForm = () => {
      const [username, setUsername] = useState('');
      const [password, setPassword] = useState('');

      const submit = e => {
        e.preventDefault();

        Meteor.loginWithPassword(username, password);
      };

      return (
        <form onSubmit={submit} className="login-form">
          <label htmlFor="username">Username</label>

          <input
            type="text"
            placeholder="Username"
            name="username"
            required
            onChange={e => setUsername(e.target.value)}
          />

          <label htmlFor="password">Password</label>

          <input
            type="password"
            placeholder="Password"
            name="password"
            required
            onChange={e => setPassword(e.target.value)}
          />

          <button type="submit">Log In</button>
        </form>
      );
    };

好了，现在你有了一个表单，让我们来使用它。

## 7.4 要求认证
我们的app应该只允许认证的用户访问其任务管理功能。

当我们没有认证的用户时，我们可以实现返回 `LoginForm` 组件，否则我们返回表单、过滤器和列表组件。

你应该首先将3个组件（表单、过滤器和列表）包裹在一个 `<Fragment>` 中，Fragment 是 React 中的一个特殊组件，你可以用它将组件组合在一起而不影响你的最终DOM，这意味着不影响你的UI，因为它不会在HTML中引入其它元素。

> 在[这里](https://reactjs.org/docs/fragments.html)阅读更多关于 Fragments 的信息

所以你可以从 `Meteor.user()` 中获得你的认证用户或null，你应该把它包在 `useTracker` 钩子里，这样才是反应式的。然后你可以根据用户是否在会话中，返回带有任务和其他内容的 `Fragment` 或 `LoginForm`。

**imports/ui/App.jsx**

    import { Meteor } from 'meteor/meteor';
    import React, { useState, Fragment } from 'react';
    import { useTracker } from 'meteor/react-meteor-data';
    import { TasksCollection } from '/imports/api/TasksCollection';
    import { Task } from './Task';
    import { TaskForm } from './TaskForm';
    import { LoginForm } from './LoginForm';

    ..
    export const App = () => {
      const user = useTracker(() => Meteor.user());
      
      ..
      return (
          ..
          <div className="main">
            {user ? (
              <Fragment>
                <TaskForm />

                <div className="filter">
                  <button onClick={() => setHideCompleted(!hideCompleted)}>
                    {hideCompleted ? 'Show All' : 'Hide Completed'}
                  </button>
                </div>

                <ul className="tasks">
                  {tasks.map(task => (
                    <Task
                      key={task._id}
                      task={task}
                      onCheckboxClick={toggleChecked}
                      onDeleteClick={deleteTask}
                    />
                  ))}
                </ul>
              </Fragment>
            ) : (
              <LoginForm />
            )}
          </div>
    ..

## 7.5 登录表单样式
好了，现在让我们来设计登录表单。

把你的标签和输入都包在 `DIV` 里，这样会更容易用 CSS 来控制它。

**client/main.css**

    .login-form {
      display: flex;
      flex-direction: column;
      height: 100%;

      justify-content: center;
      align-items: center;
    }

    .login-form > div {
      margin: 8px;
    }

    .login-form > div > label {
      font-weight: bold;
    }

    .login-form > div  > input {
      flex-grow: 1;
      box-sizing: border-box;
      padding: 10px 6px;
      background: transparent;
      border: 1px solid #aaa;
      width: 100%;
      font-size: 1em;
      margin-right: 16px;
      margin-top: 4px;
    }

    .login-form > div > input:focus {
      outline: 0;
    }

    .login-form > div > button {
      background-color: #62807e;
    }

现在你的登录表单应该是集中的、漂亮的了。

## 7.6 服务器启动
从现在起，每个任务都应该有一个拥有者。所以去你的数据库，就像你之前学的那样，从那里删除所有的任务: `db.tasks.remove({})。`

修改你的 `server/main.js`，用你的 `meteorite` 用户作为所有者添加种子任务。

确保在这次修改后重启服务器，这样 `Meteor.startup` 部分将再次运行。这可能会以任何方式自动发生，因为要在服务器端代码中进行修改。

**server/main.js**

    import { Meteor } from 'meteor/meteor';
    import { Accounts } from 'meteor/accounts-base';
    import { TasksCollection } from '/imports/api/TasksCollection';

    const insertTask = (taskText, user) =>
      TasksCollection.insert({
        text: taskText,
        userId: user._id,
        createdAt: new Date(),
      });

    const SEED_USERNAME = 'meteorite';
    const SEED_PASSWORD = 'password';

    Meteor.startup(() => {
      if (!Accounts.findUserByUsername(SEED_USERNAME)) {
        Accounts.createUser({
          username: SEED_USERNAME,
          password: SEED_PASSWORD,
        });
      }

      const user = Accounts.findUserByUsername(SEED_USERNAME);

      if (TasksCollection.find().count() === 0) {
        [
          'First Task',
          'Second Task',
          'Third Task',
          'Fourth Task',
          'Fifth Task',
          'Sixth Task',
          'Seventh Task',
        ].forEach(taskText => insertTask(taskText, user));
      }
    });

可以看到，我们正在使用一个新的字段，叫做 `userId`，值是用户的 `_id` 字段，同时也设置了 `credateAt` 字段。

## 7.7 任务所有者
现在你可以在用户界面中通过认证的用户过滤任务。当从 Mini Mongo 获取任务时，使用用户的 `_id` 将字段 `userId` 添加到你的 Mongo 选择器。

**imports/ui/App.jsx**

    ..
        const hideCompletedFilter = { isChecked: { $ne: true } };
      
        const userFilter = user ? { userId: user._id } : {};
      
        const pendingOnlyFilter = { ...hideCompletedFilter, ...userFilter };
      
        const tasks = useTracker(() => {
          if (!user) {
            return [];
          }
      
          return TasksCollection.find(
            hideCompleted ? pendingOnlyFilter : userFilter,
            {
              sort: { createdAt: -1 },
            }
          ).fetch();
        });
      
        const pendingTasksCount = useTracker(() => {
          if (!user) {
            return 0;
          }
      
          return TasksCollection.find(pendingOnlyFilter).count();
        });
    ..

        <TaskForm user={user} />
    ..

同时更新插入的调用，在 `TaskForm` 中包括 `userId` 字段。你应该把用户从 `App` 组件中传递到 `TaskForm` 中。

**imports/ui/TaskForm.jsx**

    ..
    export const TaskForm = ({ user }) => {
      const [text, setText] = useState('');

      const handleSubmit = e => {
        e.preventDefault();

        if (!text) return;

        TasksCollection.insert({
          text: text.trim(),
          createdAt: new Date(),
          userId: user._id
        });

        setText('');
      };
    ..

## 7.8 退出登录
我们还可以通过在应用栏下面显示所有者的用户名来更好地组织我们的任务。你可以在我们的 `Fragment` 开始标签之后加入一个新的 `div`。

在这里你可以添加一个 `onClick` 处理程序来注销用户。这是非常简单的，只要调用 `Meteor.logout()` 就可以了。

**imports/ui/App.jsx**

    ..
      const logout = () => Meteor.logout();

      return (
    ..
        <Fragment>
          <div className="user" onClick={logout}>
            {user.username} 🚪
          </div>
    ..

记得也要给你的用户名加上样式。

**client/main.css**

    .user {
      display: flex;

      align-self: flex-end;

      margin: 8px 16px 0;
      font-weight: bold;
    }

唷! 在这一步，你已经做了很多事情：认证了用户，在任务中设置了用户，并为用户提供了一种注销方式。

你的app应该看起来像这样:

<center><img src="/image/step07-login.png" width="20%" style="display: inline;" />
<img src="/image/step07-logout.png" width="20%" style="display: inline;" /></center>

> 回顾：你可以在[这里](https://github.com/meteor/react-tutorial/tree/master/src/simple-todos/step07)检查你的代码在这一步骤结束时应该是怎样的。

在下一步，我们将开始使用方法，只在检查一些条件后改变数据。

# 8 方法
在这一步之前，app的任何用户都可以编辑数据库的任何部分，直接在客户端进行更改。这对于快速的原型设计来说可能不错，但真正的应用需要控制对其数据的访问。

在 Meteor 中，安全地在服务器中进行修改的最简单办法是声明方法，而不是在客户端直接调用 `insert`、`update` 或 `remove`。

通过方法，你可以验证用户是否经过认证，是否被授权执行某些操作，然后相应地改变数据库。

Meteor 方法是使用函数 `Meteor.call` 与你的服务器通信的一种方式，你需要提供你的方法的名称和参数。

> 你可以在[这里](https://guide.meteor.com/methods.html)阅读更多关于方法的信息。

## 8.1 禁用快速原型技术
每个新创建的 Meteor 项目都默认安装了不安全的包。

这个包允许我们从客户端编辑数据库，就像我们上面说的，这对快速制作原型很有用。

我们需要删除它，因为顾名思义它是不安全的。

    meteor remove insecure

现在你的app的改变不再起作用了，因为你已经撤销了所有客户端数据库的权限。例如，尝试插入一个新的任务，你会看到 `insert failed: Access denied` 出现在你的浏览器控制台中。

## 8.2 添加任务方法
现在你需要定义方法。

你需要为我们想在客户端执行的每个数据库操作定义一个方法。

方法应该被定义在客户端和服务器上执行的代码中，以支持优化的用户界面。

**优化的用户界面**

当我们使用 `Meteor.call` 在客户端调用一个方法时，有两件事是并行发生的。

1. 客户端向服务器发送请求，以便在安全环境下运行该方法。
2. 该方法的模拟直接在客户端运行，试图预测调用的结果。

这意味着，在服务器返回结果之前，一个新创建的任务实际出现在了屏幕上。

如果结果与服务器的结果一致，一切都保持原样，否则用户界面会被修改以反映服务器的实际状态。

> Meteor 为你做了所有这些工作，你不需要担心这些，但了解正在发生的事情很重要。你可以在[这里](https://blog.meteor.com/optimistic-ui-with-meteor-67b5a78c3fcf)阅读更多关于 Optimistic UI 的内容。

现在你应该在你的 `imports/api` 文件夹中添加一个名为 `tasksMethods` 的新文件。在这个文件中，对于你在客户端进行的每个操作，接下来我们将从客户端调用这些方法，而不是直接使用 Mini Mongo 操作。

在 Methods 里面，你有一些特殊的属性准备在这个对象上使用，例如，认证用户的 `userId`。

**imports/api/tasksMethods.js**

    import { check } from 'meteor/check';
    import { TasksCollection } from './TasksCollection';
     
    Meteor.methods({
      'tasks.insert'(text) {
        check(text, String);
     
        if (!this.userId) {
          throw new Meteor.Error('Not authorized.');
        }
     
        TasksCollection.insert({
          text,
          createdAt: new Date,
          userId: this.userId,
        })
      },
     
      'tasks.remove'(taskId) {
        check(taskId, String);
     
        if (!this.userId) {
          throw new Meteor.Error('Not authorized.');
        }
     
        TasksCollection.remove(taskId);
      },
     
      'tasks.setIsChecked'(taskId, isChecked) {
        check(taskId, String);
        check(isChecked, Boolean);
     
        if (!this.userId) {
          throw new Meteor.Error('Not authorized.');
        }
     
        TasksCollection.update(taskId, {
          $set: {
            isChecked
          }
        });
      }
    });

正如你在代码中所看到的，我们也在使用检查包来确保我们收到了预期的输入类型，这对于确保你确切地知道你在数据库中插入或更新什么是很重要的。

最后一部分是确保你的服务器正在注册这些方法，你可以在 `server/main.js` 中导入这个文件，以强制评估。

**server/main.js**

    import { Meteor } from 'meteor/meteor';
    import { Accounts } from 'meteor/accounts-base';
    import { TasksCollection } from '/imports/db/TasksCollection';
    import '/imports/api/tasksMethods';

看，你不需要从导入中得到任何对象，你只需要要求你的服务器导入文件，然后 `Meteor.methods` 将被评估，并在服务器启动时注册你的方法。

## 8.3 实现方法调用
由于你已经定义了方法，你需要将操作集合的代码替换成它们。

在 `TaskForm` 文件中，你应该调用 `Meteor.call('tasks.insert', text);` 而不是 `TasksCollection.insert`。记住，也要修正你导入的方法。

**imports/ui/TaskForm.jsx**

    import { Meteor } from 'meteor/meteor';
    import React, { useState } from 'react';

    export const TaskForm = () => {
      const [text, setText] = useState('');

      const handleSubmit = e => {
        e.preventDefault();

        if (!text) return;

        Meteor.call('tasks.insert', text);

        setText('');
      };
      ..
    };

可以看到，你的 `TaskForm` 组件不需要再接收用户，因为我们在服务器中得到了 `userId`。

在 `App` 文件中，你应该调用 `Meteor.call('tasks.setIsChecked', _id, !isChecked);` 而不是 `TasksCollection.update`，以及 `Meteor.call('tasks.remove', _id)` 而不是 `TasksCollection.remove`。

记住也要从你的 `<TaskForm />` 中删除用户属性。

**imports/ui/App.jsx**

    import { Meteor } from 'meteor/meteor';
    import React, { useState, Fragment } from 'react';
    import { useTracker } from 'meteor/react-meteor-data';
    import { TasksCollection } from '/imports/db/TasksCollection';
    import { Task } from './Task';
    import { TaskForm } from './TaskForm';
    import { LoginForm } from './LoginForm';

    const toggleChecked = ({ _id, isChecked }) =>
      Meteor.call('tasks.setIsChecked', _id, !isChecked);

    const deleteTask = ({ _id }) => Meteor.call('tasks.remove', _id);
    ..

                <TaskForm />
    ..

现在你的输入和按钮将重新开始工作。你获得了什么？

1. 当我们向数据库插入任务时，我们可以安全地验证用户是经过认证的；`createdAt` 字段是正确的；`userId` 是合法的。
2. 如果我们愿意，我们可以在以后的方法中添加额外的验证逻辑。
3. 我们的客户端代码与数据库逻辑更加隔离。没有在事件处理程序中发生大量的处理逻辑，而是在任何地方都有可调用的方法。

## 8.4 api和db文件夹
我们想在这里花点时间思考一下，集合文件所在的文件夹是api，但API在你的项目中意味着服务器和客户端之间的通信层，但集合不再执行这个角色了。所以你应该把你的 `TasksCollection` 文件移到一个叫`db`的新文件夹里。

这个改变不是必须的，但建议保持我们的命名与实际情况一致。

记住要修复你的导入，你有4个导入到 `TasksCollection` 的文件，在以下文件中:

1. `imports/api/tasksMethods.js`
2. `imports/ui/TaskForm.jsx`
3. `imports/ui/App.jsx`
4. `server/main.js`

它们应该从 `import { TasksCollection } from '/imports/api/TasksCollection';` 改为 `import { TasksCollection } from '/imports/db/TasksCollection';`。

你的app看起来应该和以前一样，因为在这一步中，我们没有改变任何用户可见的内容。你可以使用 `Meteor DevTools` 来查看送往服务器的信息和返回的结果，这些信息可以在`DDP`标签页中找到。

> DDP 是 Meteor 通信层背后的协议，你可以在[这里](https://github.com/meteor/meteor/blob/devel/packages/ddp/DDP.md)了解更多关于它的信息。

我们建议你改变对错误类型的检查调用，以产生一些错误，这样你就可以理解在这些情况下会发生什么。

> 回顾：你可以在[这里](https://github.com/meteor/react-tutorial/tree/master/src/simple-todos/step08)检查你的代码在这一步骤结束时应该是怎样的。

在下一步中，我们将开始使用 `Publication`，只发布每个案例中需要的数据。

# 9 发布
现在我们已经把所有的app的敏感代码移到了方法中，我们需要了解 Meteor 的另一半涉及安全内容。到目前为止，我们一直假设整个数据库存在于客户端，这意味着如果我们调用 `Tasks.find()`，我们将得到集合中的每个任务。如果我们app的用户想要存储隐私敏感的数据，这就不好了。我们需要一种方法来控制 Meteor 将哪些数据发送到客户端的数据库。

## 9.1 自动发布
就像上一步中的 `insecure` 一样，所有新的 Meteor 应用程序开始时都会自动发布包，它会自动将所有数据库内容同步到客户端。所以你应该删除它:

    meteor remove autopublish

当应用程序刷新时，任务列表将是空的。如果没有自动发布包，我们将不得不明确指定服务器向客户端发送的内容。Meteor 中实现这个的函数是 `Meteor.publish` 和 `Meteor.subscribe`。

- `Meteor.publish`：允许将数据从服务器发布到客户端。
- `Meteor.subscribe`：允许客户端的代码向客户端索取数据。

## 9.2 任务发布
你需要首先在你的服务器上添加一个 `publication`，这个 `publication` 应该发布来自认证用户的所有任务。在方法中，你也可以在发布功能中使用 `this.userId` 来获得认证用户的身份。

在 `api` 文件夹中创建一个名为 `tasksPublications.js` 的新文件。

**imports/api/tasksPublications.js**

    import { Meteor } from 'meteor/meteor';
    import { TasksCollection } from '/imports/db/TasksCollection';

    Meteor.publish('tasks', function publishTasks() {
      return TasksCollection.find({ userId: this.userId });
    });

由于你是在这个函数里面使用的，所以不应该使用箭头函数（=>），因为箭头函数不提供上下文。你需要以传统的方式，通过函数关键字使用这个函数。

最后一部分是确保你的服务器注册这个 `publication`。你可以这样做，在 `server/main.js` 中导入这个文件，以强制评估。

**server/main.js**

    import { Meteor } from 'meteor/meteor';
    import { Accounts } from 'meteor/accounts-base';
    import { TasksCollection } from '/imports/db/TasksCollection';
    import '/imports/api/tasksMethods';
    import '/imports/api/tasksPublications';

## 9.3 任务订阅
然后我们可以在客户端订阅该发布。

由于我们想要接收来自该发布的变化，我们将在一个 `useTracker` 钩子中订阅它。

这也是我们重构代码的好时机，使用一个 `useTracker` 从 `TasksCollection` 获取数据。

**imports/ui/App.jsx**

    ..
      const { tasks, pendingTasksCount, isLoading } = useTracker(() => {
        const noDataAvailable = { tasks: [], pendingTasksCount: 0 };
        if (!Meteor.user()) {
          return noDataAvailable;
        }
        const handler = Meteor.subscribe('tasks');

        if (!handler.ready()) {
          return { ...noDataAvailable, isLoading: true };
        }

        const tasks = TasksCollection.find(
          hideCompleted ? pendingOnlyFilter : userFilter,
          {
            sort: { createdAt: -1 },
          }
        ).fetch();
        const pendingTasksCount = TasksCollection.find(pendingOnlyFilter).count();

        return { tasks, pendingTasksCount };
      });
    ..

## 9.4 加载状态
你也应该为你的app添加一个加载状态。也就是说，当订阅数据还没有准备好的时候，你应该向你的用户告知这一点。要发现订阅是否准备好了，你应该得到订阅调用的返回值，它是一个包含订阅状态的对象，包括将返回一个布尔值的ready函数。

**imports/ui/App.jsx**

    ..
      <div className="filter">
        <button onClick={() => setHideCompleted(!hideCompleted)}>
            {hideCompleted ? 'Show All' : 'Hide Completed'}
        </button>
      </div>
      
      {isLoading && <div className="loading">loading...</div>}
      
      <ul className="tasks">
    ..

让我们也调整一下这种加载的样式。

    .loading {
      display: flex;
      flex-direction: column;
      height: 100%;

      justify-content: center;
      align-items: center;

      font-weight: bold;
    }

一旦你这样做了，所有的任务将重新出现。

在服务器上调用 `Meteor.publish` 会注册一个名为 `tasks` 的发布。当 `Meteor.subscribe` 在客户端被调用时，客户端就会订阅该发布的所有数据。在这个例子中，它是数据库中的所有任务，用于认证用户。

## 9.5 检查用户权限
只有任务的所有者才能改变某些内容。你应该改变你的方法，以检查被验证的用户是否是创建任务的同一个用户。

**imports/api/tasksMethods.js**

    ..
      'tasks.remove'(taskId) {
        check(taskId, String);

        if (!this.userId) {
          throw new Meteor.Error('Not authorized.');
        }

        const task = TasksCollection.findOne({ _id: taskId, userId: this.userId });

        if (!task) {
          throw new Meteor.Error('Access denied.');
        }

        TasksCollection.remove(taskId);
      },

      'tasks.setIsChecked'(taskId, isChecked) {
        check(taskId, String);
        check(isChecked, Boolean);

        if (!this.userId) {
          throw new Meteor.Error('Not authorized.');
        }

        const task = TasksCollection.findOne({ _id: taskId, userId: this.userId });

        if (!task) {
          throw new Meteor.Error('Access denied.');
        }

        TasksCollection.update(taskId, {
          $set: {
            isChecked,
          },
        });
      },
    ..

为什么不在客户端返回其他用户的任务很重要呢？

这很重要，因为任何人都可以使用浏览器控制台调用 Meteor 方法。你可以使用你的 DevTools 控制台标签来测试，然后输入并点击回车：`Meteor.call('tasks.remove', 'xtPTsNECC3KPuMnDu');` 。如果你从 remove 方法中移除验证，并从数据库中传递一个有效的任务 `_id`，你将能够移除它。

> 回顾：你可以在[这里](https://github.com/meteor/react-tutorial/tree/master/src/simple-todos/step09)检查你的代码在这一步骤结束时应该是怎样的。

在下一步，我们将在移动环境中以本地应用程序的形式运行该应用程序。

# 10 在移动端运行
目前，Windows 上的 Meteor 不支持在移动端构建。如果你在 Windows 上使用 Meteor，你应该跳过这个步骤。

到目前为止，我们只在网页浏览器中构建我们的app并进行测试，但 Meteor 的设计是为了在不同的平台上工作 -- 你的简单的todo列表网站只需几个命令就可以变成一个 iOS 或 Android app。

Meteor 使建立移动应用所需的所有工具变得简单，但下载所有程序可能需要一段时间 -- 对于Android来说，下载量约为300MB；对于iOS来说，你需要安装Xcode，这大约是2GB。如果你不想等待下载这些工具，请随时跳到下一步。

> 重要提示：手机的设置和设定是非常动态的。如果你发现下面的任何步骤没有按照规定工作，或者任何链接的文件不是最新的，请打开一个issue，我们将更新它。如果你知道应该如何改变，你也可以开PRs。

## 10.1 iOS 模拟器
如果你有一台Mac，你可以在iOS模拟器内运行你的应用程序。

按照这个指南来安装iOS的所有开发先决条件。

当你完成后，输入：

    meteor add-platform ios
    meteor run ios

你会看到iOS模拟器弹出，里面运行着你的应用程序。

<center><img src="/image/step10-ios-simulator.png" width="20%" /></center>

## 10.2 安卓模拟器
按照这个指南来安装安卓的所有开发先决条件。

当你完成所有安装后，输入:

    meteor add-platform android

在你同意许可条款后，输入:

    meteor run android

在一些初始化之后，你会看到一个安卓模拟器弹出，在一个本地安卓包装内运行你的应用程序。该模拟器可能有点慢，所以如果你想看看使用你的app的真实情况，你应该在实际设备上运行它。

<center><img src="/image/step10-android-emulator.png" width="20%" /></center>

## 10.3 安卓设备
首先，完成上述所有步骤，在你的系统上设置安卓工具。然后，确保你在手机上启用了USB调试功能，并且用USB线将其插入电脑。另外，在设备上运行之前，你必须退出安卓模拟器。

然后，运行以下命令：

    meteor run android-device

该app将被构建并安装在你的设备上

## 10.4 iPhone or iPad
> 这需要一个苹果开发者账号。

如果你有一个苹果开发者账号，你也可以在iOS设备上运行你的app。运行以下命令：

    meteor run ios-device

这将为你的iOS应用程序打开Xcode的项目。你可以使用Xcode在任何设备或Xcode支持的模拟器上启动该应用程序。

> 回顾：你可以在[这里](https://github.com/meteor/react-tutorial/tree/master/src/simple-todos/step10)检查你的代码在这一步骤结束时应该是怎样的。

在下一步，我们将添加自动测试。

# 11 测试
现在我们已经为app创建了一些功能，让我们添加一个测试，以确保我们不会倒退和以我们期望的方式工作。

我们将写一个测试，执行一个方法，并验证它是否正常工作。

## 11.1 安装依赖
我们将为 Mocha JavaScript 测试框架添加一个测试驱动，以及一个测试断言库。

    meteor add meteortesting:mocha
    meteor npm install --save-dev chai

现在我们可以在"测试模式"下运行我们的应用程序，方法是运行 `meteor test` 并指定一个测试驱动包（你需要停止常规应用程序的运行，或者用 `-port XYZ` 指定一个备用端口）。

    TEST_WATCH=1 meteor test --driver-package meteortesting:mocha

它应该输出类似这样的内容:

    simple-todos-react
      ✓ package.json has correct name
      ✓ server is not client

    2 passing (10ms)

这两个测试是从哪里来的？每一个新的 Meteor 应用程序都包括一个 `test/main.js` 模块，其中包含了几个使用描述、断言风格的测试例子，这些风格在 Mocha 等测试框架中很流行。

> Meteor Mocha 的集成是由社区维护的，你可以在[这里](https://github.com/meteortesting/meteor-mocha)阅读更多信息。

当你用这些选项运行时，你也可以在浏览器中的应用程序URL中看到测试的结果:

<center><img src="/image/step11-test-report.png" width="100%" /></center>

## 11.2 脚手架测试
然而，如果你希望将你的测试分成多个模块，你也可以这样做：添加一个新的测试模块，名为 `imports/api/tasksMethods.test.js`。

**imports/api/tasksMethods.tests.js**

    import { Meteor } from 'meteor/meteor';

    if (Meteor.isServer) {
      describe('Tasks', () => {
        describe('methods', () => {
          it('can delete owned task', () => {});
        });
      });
    }

并在 `test/main.js` 中导入它，比如导入 `'/imports/api/tasksMethods.test.js';` 并删除该文件中的其他所有内容，因为我们不需要这些测试。

**tests/main.js**

    import '/imports/api/tasksMethods.tests.js';

## 11.3 数据准备
在任何测试中，你需要确保数据库在开始之前处于我们期望的状态。你可以使用 Mocha 的 `beforeEach` 结构来轻松做到这一点。

**imports/api/tasksMethods.tests.js**

    import { Meteor } from 'meteor/meteor';
    import { Random } from 'meteor/random';
    import { TasksCollection } from '/imports/db/TasksCollection';

    if (Meteor.isServer) {
      describe('Tasks', () => {
        describe('methods', () => {
          const userId = Random.id();
          let taskId;

          beforeEach(() => {
            TasksCollection.remove({});
            taskId = TasksCollection.insert({
              text: 'Test Task',
              createdAt: new Date(),
              userId,
            });
          });
        });
      });
    }

在这里，你正在创建一个单一的任务，它与一个随机的 userId 相关联，这个 userId 在每次测试运行中都会不同。

## 11.4 测试任务删除
现在你可以编写测试，以该用户的身份调用 `tasks.remove` 方法，并验证任务是否被删除，因为你要测试一个方法并模拟认证的用户。你可以安装这个实用程序包，使你的生活更轻松：

    meteor add quave:testing

**imports/api/tasks.tests.js**

    import { Meteor } from 'meteor/meteor';
    import { Random } from 'meteor/random';
    import { mockMethodCall } from 'meteor/quave:testing';
    import { assert } from 'chai';
    import { TasksCollection } from '/imports/db/TasksCollection';
    import '/imports/api/tasksMethods';

    if (Meteor.isServer) {
      describe('Tasks', () => {
        describe('methods', () => {
          const userId = Random.id();
          let taskId;

          beforeEach(() => {
            TasksCollection.remove({});
            taskId = TasksCollection.insert({
              text: 'Test Task',
              createdAt: new Date(),
              userId,
            });
          });

          it('can delete owned task', () => {
            mockMethodCall('tasks.remove', taskId, { context: { userId } });

            assert.equal(TasksCollection.find().count(), 0);
          });
        });
      });
    }

记得从 `chai` 导入 `assert` (import { assert } from 'chai';)

## 11.5 更多的测试
你可以添加你想要的测试。下面的代码提供了一些其他的测试，可以帮助你更多的了解测试什么及如何测试：

**imports/api/tasks.tests.js**

    import { Meteor } from 'meteor/meteor';
    import { Random } from 'meteor/random';
    import { mockMethodCall } from 'meteor/quave:testing';
    import { assert } from 'chai';
    import { TasksCollection } from '/imports/db/TasksCollection';
    import '/imports/api/tasksMethods';

    if (Meteor.isServer) {
      describe('Tasks', () => {
        describe('methods', () => {
          const userId = Random.id();
          let taskId;

          beforeEach(() => {
            TasksCollection.remove({});
            taskId = TasksCollection.insert({
              text: 'Test Task',
              createdAt: new Date(),
              userId,
            });
          });

          it('can delete owned task', () => {
            mockMethodCall('tasks.remove', taskId, { context: { userId } });

            assert.equal(TasksCollection.find().count(), 0);
          });

          it(`can't delete task without an user authenticated`, () => {
            const fn = () => mockMethodCall('tasks.remove', taskId);
            assert.throw(fn, /Not authorized/);
            assert.equal(TasksCollection.find().count(), 1);
          });

          it(`can't delete task from another owner`, () => {
            const fn = () =>
              mockMethodCall('tasks.remove', taskId, {
                context: { userId: 'somebody-else-id' },
              });
            assert.throw(fn, /Access denied/);
            assert.equal(TasksCollection.find().count(), 1);
          });

          it('can change the status of a task', () => {
            const originalTask = TasksCollection.findOne(taskId);
            mockMethodCall('tasks.setIsChecked', taskId, !originalTask.isChecked, {
              context: { userId },
            });

            const updatedTask = TasksCollection.findOne(taskId);
            assert.notEqual(updatedTask.isChecked, originalTask.isChecked);
          });

          it('can insert new tasks', () => {
            const text = 'New Task';
            mockMethodCall('tasks.insert', text, {
              context: { userId },
            });

            const tasks = TasksCollection.find({}).fetch();
            assert.equal(tasks.length, 2);
            assert.isTrue(tasks.some(task => task.text === text));
          });
        });
      });
    }

如果你再次运行测试命令或之前让它在观察模式下运行，你应该看到以下输出:

    Tasks
      methods
        ✓ can delete owned task
        ✓ can't delete task without an user authenticated
        ✓ can't delete task from another owner
        ✓ can change the status of a task
        ✓ can insert new tasks

    5 passing (70ms)

为了方便输入测试命令，你可能想在package.json文件的scripts部分添加一个速记。

事实上，新的Meteor应用程序带有一些预先配置好的npm脚本，欢迎你使用或修改它们。

标准的meteor npm test命令运行以下命令:

    meteor test --once --driver-package meteortesting:mocha

这个命令适合在持续集成（CI）环境中运行，如 Travis CI 或 CircleCI，因为它只运行你的服务器端测试。如果所有的测试都通过了，则以0退出。

如果你想在开发应用程序时运行测试（并在开发服务器重新启动时重新运行），考虑使用 `meteor npm run test-app`，这相当于:

    TEST_WATCH=1 meteor test --full-app --driver-package meteortesting:mocha

这和前面的命令几乎一样，只是它也会正常加载你的应用程序代码（由于`--full-app`），允许你在浏览器中与你的应用程序交互，同时运行客户端和服务器测试。

你可以用 Meteor 测试做更多的事情! 你可以在 Meteor 指南关于测试的文章中阅读更多内容。

> 回顾：你可以在[这里](https://github.com/meteor/react-tutorial/tree/master/src/simple-todos/step11)检查你的代码在这一步骤结束时应该是怎样的。

在下一步，我们将把你的应用程序部署到 Galaxy，这是 Meteor 应用程序的最佳托管，由 Meteor 背后的同一个团队开发。

# 12 部署
现在你的应用已经测试完毕，准备发布，以便任何人都可以使用它。

运行你的 Meteor 应用程序的最好地方是 Galaxy。Galaxy 提供免费的部署。很酷，对吗？

如果你在这一步有任何问题，应该给 Galaxy 支持部门发邮件，他们会帮助你。把你的信息发到 `support@meteor.com`，尽量详细解释问题是什么，你将会尽快得到帮助。同时确保在主题中包括 `React Tutorial`，这样对方就知道你是从哪里来的了。

## 12.1 创建账号

你有一个 Meteor Cloud 账号吗？没有吗？好吧，让我们来解决这个问题。

进入[cloud.meteor.com](https://cloud.meteor.com/?isSignUp=true)，你将会看到一个像这样的表单：

<center><img src="/image/step12-sign-up.png" width="100%" /></center>

在GitHub上注册，然后从那里开始。它只要求你提供用户名和密码，你将需要这些来部署你的应用程序。

完成后，你的账户就创建好了。你可以用这个账户访问 [atmospherejs.com](https://atmospherejs.com/)、论坛和更多的内容，包括 Galaxy 的免费部署。

## 12.2 部署应用
现在你已经准备好部署了，确保在部署前运行 `meteor npm install`，以确保所有的依赖项都已安装。

你还需要选择一个子域名来发布你的应用程序。我们将使用主域名 `meteorapp.com`，这是免费的，包含在任何 Galaxy 计划中。

在这个例子中，我们将使用 `react-tutorial.meteorapp.com`，但请确保你选择了不同的，否则你将收到一条错误信息，说它已经被使用了。

> 你可以在[这里](https://cloud-guide.meteor.com/custom-domains.html)学习如何在 Galaxy 上使用自定义域名。自定义域名从 Essentials 计划开始可用。

运行部署命令：

    meteor deploy react-tutorial.meteorapp.com --free --mongo

确保你用一个想作为子域名的自定义名称来替换 `react-tutorial`。

你将会看到一个类似这样的日志:

    meteor deploy react-tutorial.meteorapp.com --free --mongo
    Talking to Galaxy servers at https://us-east-1.galaxy-deploy.meteor.com
    Preparing to build your app...                
    Preparing to upload your app... 
    Uploaded app bundle for new app at react-tutorial.meteorapp.com.
    Galaxy is building the app into a native image.
    Waiting for deployment updates from Galaxy... 
    Building app image...                         
    Deploying app...                              
    You have successfully deployed the first version of your app.

    *** Your MongoDB shared instance database URI will be here as well ***

    For details, visit https://galaxy.meteor.com/app/react-tutorial.meteorapp.com

这个过程通常需要5分钟左右，但这取决于你的网速，因为它要把应用程序包发送到 Galaxy 服务器。

> Galaxy 构建了一个新的 Docker 镜像，其中包含了你的应用包，然后用它来部署容器，阅读[更多](https://cloud-guide.meteor.com/container-environment.html)。

你可以在 Galaxy 上查看日志，包括 Galaxy 正在构建的 Docker 镜像和部署的部分。

## 12.3 访问并享受吧
现在你应该能够访问你的 Galaxy 仪表板，网址是 https://galaxy.meteor.com/app/react-tutorial.meteorapp.com （用你的子域替换 `react-tutorial`）。

当然，也能够在你选择的域中访问和使用你的应用程序，在我们的例子中是 `react-tutorial.meteorapp.com`。祝贺你!

> 我们部署了在美国运行的 Galaxy 系统（us-east-1），我们也有在世界其他地区运行的 Galaxy 系统，请查看[这里](https://cloud-guide.meteor.com/deploy-region.html)的列表。

这很了不起，你的 Meteor 应用程序运行在 Galaxy 上，准备好被世界上的任何人使用吧！

> 回顾：你可以在[这里](https://github.com/meteor/react-tutorial/tree/master/src/simple-todos/step12)检查你的代码在这一步骤结束时应该是怎样的。

在下一步，我们将为你提供一些继续开发你的应用程序的想法，以及接下来要看到的更多内容。

# 13 后续
祝贺你新建立了 Meteor 应用程序，它正在 Galaxy 上运行! 哇！

你的应用程序目前支持为认证的用户添加私人任务。

对新功能的想法:

- 将已完成的任务样式化，使其状态更加明显
- 在登录表单中也添加创建新用户的选项
- 与他人分享任务

在 Galaxy 上要做的事情:

- 在 Galaxy 上检查你的日志，看[这里](https://www.youtube.com/watch?v=WPYyHeWM21Q)或读[这里](https://cloud-guide.meteor.com/logs.html)
- 设置你的免费SSL证书，这样你就可以使用`https`了，阅读[这里](https://cloud-guide.meteor.com/encryption.html)
- 设置你的通知，阅读[这里](https://cloud-guide.meteor.com/notifications.html)
- 根据需求自动扩展你的应用程序，观看[这里](https://www.youtube.com/watch?v=rwLoviLzG6s)或阅读[这里](https://cloud-guide.meteor.com/triggers.html)
- 在APM上进行专业操作并观察你的指标，阅读[这里](https://cloud-guide.meteor.com/apm-getting-started.html)
- 查看所有的 Galaxy 指南，了解[更多](https://cloud-guide.meteor.com/)

这里有一些关于你下一步可以去哪里的选择：

- 阅读[Meteor指南](https://guide.meteor.com/)，了解最佳实践和有用的社区包
- 查阅[完整的文档](https://docs.meteor.com/)
