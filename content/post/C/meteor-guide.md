---
title: Meteor 指南
date: 2021-10-17
draft: true
categories: ["技术文档"]
tags: ["Meteor", "翻译"]
comment: false
---

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
<p style="text-align:center;font-size:1.3em">文档目录结构</p>

- [1 介绍](#1-介绍)
    - [1.1 什么是Meteor?](#11-什么是meteor)
    - [1.2 快速入门](#12-快速入门)
    - [1.3 Meteor资源](#13-meteor资源)
    - [1.4 什么是Meteor指南?](#14-什么是meteor指南)
    - [1.5 指南开发](#15-指南开发)
- [2 代码风格](#2-代码风格)
    - [2.1 风格一致的好处](#21-风格一致的好处)
    - [2.2 JavaScript风格指南](#22-javascript风格指南)
    - [2.3 用ESLint检查你的代码](#23-用eslint检查你的代码)
    - [2.4 Meteor编码风格](#24-meteor编码风格)
- [3 应用程序结构](#3-应用程序结构)
    - [3.1 通用JavaScript](#31-通用javascript)
    - [3.2 文件结构](#32-文件结构)
    - [3.3 默认的文件加载顺序](#33-默认的文件加载顺序)
    - [3.4 拆分成多个应用程序](#34-拆分成多个应用程序)
- [4 迁移到 Meteor 2.4](#4-迁移到-meteor-24)
    - [4.1 从2.3以前的版本迁移?](#41-从23以前的版本迁移)

<!-- markdown-toc end -->


# 1 介绍
这是使用 Meteor 的指南，它是一个开发现代网络和移动应用的全栈式 JavaScript 平台。

## 1.1 什么是Meteor?
Meteor 是一个用于开发现代网络和移动应用的全栈 JavaScript平台。Meteor 包括一套用于构建连接客户端反应式应用程序的关键技术、一个构建工具、以及来自 Node.js 和一般 JavaScript社区的一套精心策划的软件包。

- Meteor 允许你用一种语言，即 JavaScript，在所有环境中进行开发：应用服务器、网络浏览器和移动设备。
- Meteor 使用线上数据，这意味着服务器发送数据，而不是HTML，然后由客户端进行渲染。
- Meteor 拥抱生态系统，以一种谨慎和深思熟虑的方式将极其活跃的JavaScript社区的最佳部分带给你。
- Meteor 提供了全堆栈的反应性，允许你的用户界面以最小的开发工作量无缝地反映世界的真实状态。

## 1.2 快速入门
按照我们[文档中的步骤](https://docs.meteor.com/install.html)安装最新的官方 Meteor版本。

一旦你安装了 Meteor，打开一个新的终端窗口并创建一个项目:

    meteor create myapp

在本地运行它:

    cd myapp
    meteor npm install
    meteor
    # Meteor server running on: http://localhost:3000/

> Meteor 捆绑了 npm，所以你可以输入 `meteor npm` 而不用担心自己安装。如果你喜欢，你也可以使用全局安装的 npm 来管理你的软件包。

## 1.3 Meteor资源
1. 从 [教程页面](https://www.meteor.com/developers/tutorials) 开始着手使用 Meteor。
2. [Meteor Examples](https://github.com/meteor/examples) 里有一系列使用 Meteor 的例子。你也可以将你的例子提交上去。
3. 一旦你熟悉了基础知识，[Meteor指南](http://guide.meteor.com/) 涵盖了关于如何在更大规模的应用中使用 Meteor 的中间材料。
4. 访问 [Meteor论坛](https://forums.meteor.com/) 来宣布项目，获得帮助，谈论社区，或者讨论对核心代码的修改。
5. [Meteor Slack社区](https://join.slack.com/t/meteor-community/shared_invite/enQtODA0NTU2Nzk5MTA3LWY5NGMxMWRjZDgzYWMyMTEyYTQ3MTcwZmU2YjM5MTY3MjJkZjQ0NWRjOGZlYmIxZjFlYTA5Mjg4OTk3ODRiOTc) 是询问（和回答！）技术问题的最佳场所，同时也可以认识 Meteor 开发者。
6. [Atmosphere](https://atmospherejs.com/) 是专门为 Meteor 设计的社区包的代码仓库。

## 1.4 什么是Meteor指南?
这是一组文章，概述了使用 Meteor 平台开发应用程序的最佳实践的意见。我们的目标是涵盖所有现代 Web和移动应用开发中常见的模式，所以这里记录的许多概念不一定是针对 Meteor 的，可以应用于任何以现代交互式用户界面为重点的应用程序。

Meteor指南中的任何内容都不是建立 Meteor应用的必要条件 -- 你当然可以用与指南中的原则和模式相矛盾的方式来使用这个平台。然而，该指南是一个记录最佳实践和社区惯例的尝试，所以我们希望 Meteor社区的大多数人能够从采用这里记录的实践中受益。

Meteor平台的API可以在 [docs网站](https://docs.meteor.com/) 上找到，你也可以在 [atmosphere](https://atmospherejs.com/) 上浏览社区包。

**1.目标受众**

本指南针对的是对 JavaScript、Meteor平台和一般的Web开发有一定了解的中级开发者。如果你刚开始接触 Meteor，我们建议从教程开始。

**2.应用实例**

如果你想看一些例子，我们有一个专门的资源库，里面有几个由社区提供的例子，展示了许多概念，在用 Meteor 实现你的应用程序时可以使用。要了解更多，你可以看[这里](https://github.com/meteor/examples)。

## 1.5 指南开发
**1.贡献**

持续的 Meteor指南开发是在 [GitHub](https://github.com/meteor/guide) 上公开进行的。我们鼓励 pull requests 和 issues 来讨论任何可以对内容进行修改的问题。我们希望保持我们的过程是公开和诚实的，这将使我们清楚地知道我们计划在指南中包括什么，以及在未来的 Meteor 版本中会有什么变化。

**2.指南目标**

指南中做出的决定和实践必然是有取舍的。某些最佳实践将被强调，而其他有效的方法则被忽略。我们的目标是在重大决策上达成与社区的共识，但在开发你的应用程序时，总会有其他方法来解决问题。我们相信，重要的是，要知道解决一个问题的"标准"方法是什么，然后再进行其他选择。如果另一种方法被证明是优越的，那么它就应该进入本指南的未来版本。

指南的一个重要功能是塑造 Meteor 平台的未来发展。通过记录最佳实践，该指南指出了平台中可以更好、更容易或更高性能的领域，因此该指南将被用来关注许多未来的平台选择。

同样地，指南中强调的平台中的差距往往可以由社区包来填补；我们希望，如果你看到了一个可以通过编写包来改善 Meteor 工作流程的机会，你应该抓住它。如果你不确定如何最好地设计或架构你的包，请在论坛上展开讨论。

# 2 代码风格
推荐的代码风格指南。

读完这篇文章，你会知道：

1. 为什么拥有一致的代码风格是个好主意。
2. 我们为 JavaScript 代码推荐哪种风格指南。
3. 如何设置 ESLint 来自动检查代码风格。
4. 对 Meteor 特定模式的风格建议，如方法、发布等。

## 2.1 风格一致的好处
多年来，开发者们花了无数的时间来争论单引号和双引号、在哪里放括号、要打多少空格、以及其他各种表面上的代码风格问题。这些问题充其量只是与代码质量有切身关系，但却很容易产生意见，因为它们很直观。

虽然对字符串使用单引号还是双引号对你的代码库并不一定重要，但一旦做出决定，并在整个组织中保持一致，会有很大的好处。这些好处也适用于整个 Meteor 和 JavaScript 开发社区。

**1.易于阅读的代码**

就像你不会一个字一个字地读英语句子一样，你也不会一个符号一个符号地读代码。大多数情况下，你只是看某个表达式的形状，或者它在编辑器中的突出显示方式，然后假设它是做什么的。如果每段代码的风格都是一致的，那就可以确保看起来相同的代码位实际上就是相同的 -- 没有任何隐藏的标点符号或你意想不到的麻烦，因此你可以专注于理解逻辑而不是符号。这方面的一个例子是缩进 -- 虽然在 JavaScript 中，缩进是没有意义的，但是让你所有的代码都有一致的缩进是很有帮助的，因为这样你就不需要详细地阅读所有的括号来了解它想表达什么。

    // This code is misleading because it looks like both statements
    // are inside the conditional.
    if (condition)
      firstStatement();
      secondStatement();

    // Much clearer!
    if (condition) {
      firstStatement();
    }

    secondStatement();

**2.自动检查错误**

拥有一致的风格意味着更容易采用标准工具进行错误检查。例如，如果你采用了一个惯例，即你必须始终使用 `let` 或 `const` 而不是 `var`，那么你现在就可以使用一个工具来确保你的所有变量都以你所期望的方式进行范围界定。这意味着你可以避免变量产生异常表现的bug。另外，通过强制要求所有的变量在使用前都要声明，你可以在运行任何代码前就发现打字错误。

**3.更深层次的理解**

要想一下子学会一门编程语言的所有知识是很难的。例如，刚接触到 JavaScript 的程序员经常会在 var 关键字和函数范围上纠结。使用社区推荐的具有自动提示功能的编码风格可以主动警告你这些陷阱。这意味着你可以直接进入编码，而不必提前了解 JavaScript 的所有边缘情况。

当你写更多的代码并遇到推荐的风格规则时，你可以把它作为一个机会来学习更多关于你的编程语言和不同的人喜欢如何使用它。

## 2.2 JavaScript风格指南
基于多方面的原因，我们坚信 JavaScript 是 Meteor 中构建 Web 应用的最佳语言。JavaScript 在不断地改进，而围绕 ES2015 的标准确实将 JavaScript社区聚集在一起。下面是我们关于今天如何在你的应用程序中使用 ES2015 JavaScript 的建议。

<center><img src="/image/ben-es2015-demo.gif" /></center>

> 从 JavaScript 重构到 ES2015 的一个例子。

**1.使用 `ecmascript` 包**

ECMAScript 是每个浏览器的 JavaScript 实现所依据的语言标准，已经转为每年发布一次标准。最新的完整标准是 ES2015，它包括了一些期待已久的、对 JavaScript 语言非常重要的改进。Meteor 的 `ecmascript` 包使用流行的 [Babel编译器](https://babeljs.io/) 将这个标准编译成所有浏览器都能理解的普通 JavaScript。它完全向后兼容"常规"的 JavaScript，所以如果你不想使用任何新的功能，就不必使用。我们花了很多精力使先进的浏览器功能（如源码地图）与这个包很好得配合，这样你就可以用你喜欢的开发工具调试你的代码，而不必看到任何编译后的输出。

`ecmascript` 包默认包含在所有新的应用程序和软件包中，并自动编译所有带有 `.js` 扩展名的文件。请看 [ecmascript 包所支持的所有ES2015功能的列表](https://docs.meteor.com/packages/ecmascript.html#Supported-ES2015-Features)。

为了获得完整的体验，你还应该使用 `es5-shim` 包，它默认包含在所有新的应用程序中。这意味着你可以依赖 `Array#forEach` 等运行时特性，而不必担心哪些浏览器支持它们。

本指南和未来的 Meteor 教程中的所有代码样本将使用所有新的 ES2015 特性。你也可以在 Meteor 博客上阅读更多关于 ES2015 以及如何开始使用它的信息:

- [开始使用 ES2015 和 Meteor](http://info.meteor.com/blog/es2015-get-started)
- [为 ES2015 设置 Sublime Text](http://info.meteor.com/blog/set-up-sublime-text-for-meteor-es6-es2015-and-jsx-syntax-and-linting)
- [ES2015 的开销如何？](http://info.meteor.com/blog/how-much-does-es2015-cost)

**2.遵循JavaScript风格指南**

我们建议选择并坚持使用一个 JavaScript 风格指南，并通过工具强制执行。我们推荐的一个流行的选择是 [Airbnb风格指南](https://github.com/airbnb/javascript) 与 ES6扩展（以及可选的 React 扩展）。

## 2.3 用ESLint检查你的代码
"代码提示"是自动检查你的代码是否有常见错误或风格问题的过程。例如，ESLint可以确定你是否在一个变量名称中出现了错别字，或者你的代码的某些部分由于`if`条件写得不好而无法执行。

我们推荐使用 [Airbnb eslint 配置](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb)，它可以验证 Airbnb 风格指南。

下面，你可以找到在许多不同的开发阶段设置自动刷新的指示。一般来说，你希望尽可能频繁地运行 linter，因为它是一种自动识别错别字和小错误的方法。

**1.安装和运行ESLint**

为了在你的应用程序中设置 ESLint，你可以安装以下 npm 包:

    meteor npm install --save-dev babel-eslint eslint-config-airbnb eslint-plugin-import eslint-plugin-meteor eslint-plugin-react eslint-plugin-jsx-a11y eslint-import-resolver-meteor eslint @meteorjs/eslint-config-meteor

> Meteor 捆绑了 npm，所以你可以输入 `meteor npm` 而不用担心自己安装它。如果你喜欢，你也可以使用全局安装的 npm 命令。

你也可以在 `package.json` 中添加 `eslintConfig` 部分，以指定你想使用的 Airbnb 的配置，并启用 [ESLint-plugin-Meteor](https://github.com/dferber90/eslint-plugin-meteor)。你也可以设置任何你想改变的额外规则，以及添加一个 `lint npm` 命令:

    {
      ...
      "scripts": {
        "lint": "eslint .",
        "pretest": "npm run lint --silent"
      },
      "eslintConfig": {
        "extends": "@meteorjs/eslint-config-meteor"
      }
    }

要运行 linter，你现在可以输入:

    meteor npm run lint

更多细节，请阅读 ESLint 网站上的 [入门指南](http://eslint.org/docs/user-guide/getting-started)。

**2.与你的编辑器集成**

Linting 是发现你的代码中潜在错误的最快方法。运行 linter 通常比运行你的应用程序或单元测试要快，所以一直运行它是个好主意。在你的编辑器中设置 linting，一开始似乎很烦人，因为当你保存格式不好的代码时，它会经常抱怨。但随着时间的推移，你会对首先写出格式良好的代码形成肌肉记忆。这里有一些在不同编辑器中设置 ESLint 的方法：

*1) Sublime Text :* 略

*2) Atom :* 略

*3) WebStorm :* 略

*4) Visual Studio Code :*

在 VSCode 中使用 ESLint 需要安装第三方的 ESLint 扩展。为了安装该扩展，请遵循以下步骤：

- 启动 VSCode，通过输入 `Ctrl+P`打开 `快速打开` 菜单。
- 在命令窗口中粘贴 `ext install vscode-eslint`，然后按回车键。
- 重新启动 VSCode。

## 2.4 Meteor编码风格
上面的部分谈到了一般的 JavaScript 代码 - 你可以在任何 JavaScript 应用程序中应用它，而不仅仅是 Meteor 应用程序。然而，有一些是Meteor特有的风格问题，特别是如何命名和组织你的应用程序的所有不同组件。

**1.集合**

集合应该被命名为复数名词，用 [PascalCase](https://en.wikipedia.org/wiki/PascalCase)。数据库中的集合名称（集合构造函数的第一个参数）应该与 JavaScript 符号的名称相同。

    // Defining a collection
    Lists = new Mongo.Collection('lists');

数据库中的字段应该像你的JavaScript变量名一样使用camelCased。

    // Inserting a document with camelCased field names
    Widgets.insert({
      myFieldName: 'Hello, world!',
      otherFieldName: 'Goodbye.'
    });

**2.方法和发布**
方法和发布的名称应该用camelCase，并与它们所在的模块命名间隔：

    // in imports/api/todos/methods.js
    updateText = new ValidatedMethod({
      name: 'todos.updateText',
      // ...
    });

注意，这个代码样本使用了 Methods 文章中推荐的 [ValidatedMethod 包](https://guide.meteor.com/methods.html#validated-method)。如果你没有使用那个包，你可以使用这个名字作为传递给 `Meteor.methods` 的属性。

下面是这个命名规则应用于一个 publication 时的样子:

    // Naming a publication
    Meteor.publish('lists.public', function listsPublic() {
      // ...
    });

**3.文件、导出和包**

你应该使用ES2015的导入和导出功能来管理你的代码。这将让你更好地了解你的代码的不同部分之间的依赖关系，并且能够帮助你导航到依赖关系的源代码。

你的应用程序中的每个文件应该代表一个逻辑模块。避免拥有导出各种不相关的函数和符号的万能工具模块。通常，这可能意味着每个文件有一个类、UI组件或集合是好的，但也有些例外，例如：你有在该文件之外不会使用的UI组件和小的子组件。

当一个文件代表一个单一的类或UI组件时，该文件应该与它所定义的内容命名相同，并使用相同的首字母。因此，如果你有一个文件导出了类：

    export default class ClickCounter { ... }

这个类应该被定义在一个叫做 `ClickCounter.js` 的文件中。当你导入它时，它看起来会像这样:

    import ClickCounter from './ClickCounter.js';

注意，导入使用相对路径，并在文件名的末尾包括文件扩展名。

对于 `Atmosphere` 包来说，由于旧的 `pre-1.3 api.export` 语法允许每个包有一个以上的 export，你往往会看到符号使用非默认的 export。例如:

    // You'll need to destructure here, as Meteor could export more symbols
    import { Meteor } from 'meteor/meteor';

    // This will not work
    import Meteor from 'meteor/meteor';

**4.模板和组件**

由于 Spacebars 模板总是全局性的，不能作为模块导入和导出，而且需要在整个应用程序中拥有完全独特的名称，我们建议用命名空间的完整路径来命名您的 Blaze 模板，并用下划线隔开。在这种情况下，下划线是一个很好的选择，因为这样你就可以在 JavaScript 中把模板的名字输入为一个符号。

    <template name="Lists_show">
      ...
    </template>

如果这个模板是一个加载服务器数据和访问路由器的"智能"组件，请在名称后加上 `_page`:

    <template name="Lists_show_page">
      ...
    </template>

通常，当你处理模板或UI组件时，你会有几个紧密配合的文件需要管理。它们可能是两个或多个HTML、CSS和JavaScript文件。在这种情况下，我们建议将这些文件放在同一目录下，并使用相同的名称:

    # The Lists_show template from the Todos example app has 3 files:
    show.html
    show.js
    show.less

整个目录或路径应表明这些模板与 Lists 模块有关，所以没有必要在文件名中重现这一信息。请阅读[下面](https://guide.meteor.com/structure.html#javascript-structure)关于目录结构的更多信息。

如果你在React中编写你的UI，你不需要使用下划线分割的名字，因为你可以使用 JavaScript 模块系统导入和导出你的组件。

# 3 应用程序结构
如何用 ES2015 模块构造你的 Meteor 应用，将代码运送到客户端和服务器，并将你的代码分成多个应用。

读完这篇文章，你会知道:

1. 在文件结构方面，Meteor应用程序与其他类型的应用程序相比是怎样的。
2. 如何组织你的应用程序，包括小型和大型的应用程序。
3. 如何以一致和可维护的方式格式化你的代码和命名你的应用程序的各个部分。

## 3.1 通用JavaScript
Meteor 是一个用于构建 JavaScript 应用程序的全栈框架。这意味着 Meteor 应用程序与大多数应用程序不同，因为它包括在客户端、Web浏览器 或 Cordova 移动应用程序内运行的代码，还有在服务器、[Node.js](http://nodejs.org/) 容器，以及两种环境中同时运行的通用代码。[Meteor构建工具](http://nodejs.org/) 允许你指定哪些 JavaScript 代码（包括任何支持的UI模板、CSS规则和静态文件）使用 ES2015 导入和导出功能 及 Meteor构建系统 [默认文件的加载顺序](https://guide.meteor.com/structure.html#load-order) 规则的组合，来在每个环境中运行。

**1.ES2015模块**

从1.3版本开始，Meteor 已经完全支持 [ES2015模块](https://developer.mozilla.org/en/docs/web/javascript/reference/statements/import) 了。ES2015 模块标准是对 [CommonJS](http://requirejs.org/docs/commonjs.html) 和 [AMD](https://github.com/amdjs/amdjs-api) 的替代，它们是通用的 JavaScript 模块格式和加载系统。

在ES2015中，你可以使用 `export` 关键字使变量在文件之外可用。要在其他地方使用这些变量，你必须通过源文件的路径来导入它们。输出一些变量的文件被称为"模块"，因为它们代表了一个可重用的代码单元。明确导入你使用的模块和包有助于你以模块化的方式编写代码，避免引入全局符号和"远距离操作"。

由于这是在 Meteor 1.3 中引入的新功能，你会发现网上有很多代码使用旧的、更集中的围绕包和应用程序声明全局符号的例子。这个旧系统仍然有效，所以要选择加入新的模块系统，代码必须放在你的应用程序的 `imports/` 目录内。我们希望未来的 Meteor 版本将默认开启所有代码的模块，因为这与更广泛的 JavaScript 社区的开发者编写代码的方式更一致。

你可以在模块包 [README](https://docs.meteor.com/#/full/modules) 中详细了解模块系统。这个包作为 ecmascript [元包](https://docs.meteor.com/#/full/ecmascript) 的一部分，自动包含在了每个新的 Meteor 应用程序中，所以大多数应用程序不需要做任何事情就可以立即开始使用模块。

**2.使用 `import` 和 `export` 的介绍**

Meteor不仅允许你在应用程序中导入 JavaScript，还可以导入 CSS 和 HTML 来控制加载顺序:

    import '../../api/lists/methods.js';  // import from relative path
    import '/imports/startup/client';     // import module with index.js from absolute path
    import './loading.html';              // import Blaze compiled HTML from relative path
    import '/imports/ui/style.css';       // import CSS from absolute path

> 关于导入样式的更多方法，请看 [Build System](https://guide.meteor.com/build-tool.html#css-importing) 一文。

Meteor 也支持标准的 ES2015 模块导出语法:

    export const listRenderHold = LaunchScreen.hold();  // named export
    export { Todos };                                   // named export
    export default Lists;                               // default export
    export default new Collection('lists');             // default export

**3.从包中导入**

在 Meteor 中，使用 `import` 语法在客户端或服务器上加载 npm 包，并像使用其他模块一样访问包的导出符号，也是简单明了的。你也可以从 Meteor Atmosphere 包中导入，但导入路径必须以 `meteor/` 为前缀，以避免与 npm 包命名空间冲突。例如，要从 npm 导入 `moment`，从 Atmosphere 导入 `HTTP`。

    import moment from 'moment';          // default import from npm
    import { HTTP } from 'meteor/http';   // named import from Atmosphere

关于使用导入包的更多细节，请看 Meteor 指南中的 [Using Packages](https://guide.meteor.com/using-packages.html)。

**4.使用 `require`**

在 Meteor 中，`import` 语句可以编译成 CommonJS 的 `require` 语法。然而，作为一种惯例，我们鼓励你使用 `import`。

也就是说，在某些情况下，你可能需要直接调用 `require`。一个值得注意的例子是：当需要从一个公共文件中获得客户端或服务器上的代码时。由于导入必须在顶层范围内，你不能把它们放在 `if` 语句中，所以你需要写这样的代码：

    if (Meteor.isClient) {
      require('./client-only-file.js');
    }

请注意，对 `require()` 的动态调用（被 require 的名称在运行时可能会改变）不能被正确分析，并可能导致客户端捆绑的损坏。

如果你需要从具有默认导出的 ES2015 模块中执行 `require`，你可以用 `require("package").default` 来获取导出。

**5.使用 CoffeeScript**

详见文档：[Modules » Syntax » CoffeeScript](https://docs.meteor.com/packages/modules.html#CoffeeScript)

    // lists.coffee
    export Lists = new Collection 'lists'

    import { Lists } from './lists.coffee'

## 3.2 文件结构
为了充分使用模块系统，并确保我们的代码只在我们要求的时候运行，我们建议所有的应用程序代码都应该放在 `import/` 目录内。这意味着 Meteor 构建系统只会在该文件被另一个文件使用导入的方式引用时才会捆绑和包含该文件（也称为 "懒执行或加载"）。

Meteor 将使用默认的文件加载顺序规则加载应用程序中任何 `import/` 目录之外的所有文件（也称为 "紧急执行或加载"）。建议你创建两个需要紧急加载的文件：`client/main.js` 和 `server/main.js`，以便为客户端和服务器定义明确的入口点。Meteor确保任何名为 `server/` 的目录中的文件都只能在服务器上使用，同理任何名为 `client/` 的目录。这也排除了试图从任何名为 `client/` 的目录中导入在服务器上使用的文件，即使它被嵌套在 `import/` 目录中，反之亦然，从 `server/` 中导入客户端文件。

这些 `main.js` 文件本身不会做任何事情，但它们应该在应用程序加载时导入一些启动模块，这些模块将分别在客户端和服务器上立即运行。这些模块应该为应用程序中使用的包做任何必要的配置，并导入应用程序的其他代码。

**1.目录布局示例**

首先，让我们看看我们的 [Todos示例应用程序](https://github.com/meteor/todos)，它是一个很好的例子，在构建你的应用程序时可以借鉴。下面是其目录结构的概述。你可以使用 `meteor create appName --full` 命令生成一个具有这种结构的新应用。

    imports/
      startup/
        client/
          index.js                 # import client startup through a single index entry point
          routes.js                # set up all routes in the app
          useraccounts-configuration.js # configure login templates
        server/
          fixtures.js              # fill the DB with example data on startup
          index.js                 # import server startup through a single index entry point

      api/
        lists/                     # a unit of domain logic
          server/
            publications.js        # all list-related publications
            publications.tests.js  # tests for the list publications
          lists.js                 # definition of the Lists collection
          lists.tests.js           # tests for the behavior of that collection
          methods.js               # methods related to lists
          methods.tests.js         # tests for those methods

      ui/
        components/                # all reusable components in the application
                                   # can be split by domain if there are many
        layouts/                   # wrapper components for behaviour and visuals
        pages/                     # entry points for rendering used by the router

    client/
      main.js                      # client entry point, imports all client code

    server/
      main.js                      # server entry point, imports all server code

**2.结构化的导入**

现在我们已经把所有文件放在了 `import/` 目录下，让我们考虑一下如何用模块来组织我们的代码。把所有在应用程序启动时运行的代码放在 `import/startup` 目录下是有意义的。另一个好主意是将数据和业务逻辑与 UI 渲染代码分开。我们建议使用名为`imports/api` 和 `imports/ui` 的目录来进行这种逻辑区分。

在 `imports/api` 目录下，根据代码所提供的API的域将代码分在几个目录是很明智的 - 通常这与你在应用中定义的集合相对应。例如，在Todos示例应用程序中，我们有 `imports/api/lists` 和 `imports/api/todos` 域。在每个目录中，我们定义了用于操作相关域数据的集合、发布和方法。

> 注意：在一个更大的应用程序中，考虑到 todos 本身是列表的一部分，把这两个域归入一个更大的"列表"模块可能是有意义的。Todo 的例子足够小，我们需要把这些分开，只是为了证明模块化。

在 `imports/ui` 目录下，通常有必要根据它们所定义的UI端代码的类型将文件分组，即顶层页面、包装布局或可重用组件。

对于上面定义的每个模块，将各种辅助文件与基本的JavaScript文件放在一起是有意义的。例如，一个 Blaze UI 组件应该将其模板 HTML、JavaScript逻辑 和 CSS规则 放在同一个目录下。一个有业务逻辑的 JavaScript 模块应该与该模块的单元测试放在一起。

**3.启动文件**

代码中有一部分不是业务逻辑或用户界面的单元，而是一些设置或配置代码，它们需要在应用程序启动时在其上下文中运行。在 Todos示例应用程序中，`imports/startup/client/useraccounts-configuration.js` 文件配置了 `useraccounts` 登录模板（关于 `useraccounts` 的更多信息，请参考 [Accounts](https://guide.meteor.com/accounts.html) 文章）。`imports/startup/client/routes.js` 文件配置了所有的路由，然后导入客户端所需的所有其他代码。

    import { FlowRouter } from 'meteor/ostrio:flow-router-extra';
    import { BlazeLayout } from 'meteor/kadira:blaze-layout';
    import { AccountsTemplates } from 'meteor/useraccounts:core';

    // Import to load these templates
    import '../../ui/layouts/app-body.js';
    import '../../ui/pages/root-redirector.js';
    import '../../ui/pages/lists-show-page.js';
    import '../../ui/pages/app-not-found.js';

    // Import to override accounts templates
    import '../../ui/accounts/accounts-templates.js';

    // Below here are the route definitions

然后我们在 `imports/startup/client/index.js` 中导入这两个文件:

    import './useraccounts-configuration.js';
    import './routes.js';

这使我们可以很容易从主要的急加载的客户端入口 `client/main.js` 文件中导入所有客户端启动代码。只需要一个导入模块：

    import '/imports/startup/client';

我们在服务器上使用了同样的技术，在 `import/startup/server/index.js` 中导入所有启动代码:

    // This defines a starting set of data to be loaded if the app is loaded with an empty db.
    import '../imports/startup/server/fixtures.js';

    // This file configures the Accounts package to define the UI of the reset password email.
    import '../imports/startup/server/reset-password-email.js';

    // Set up some rate limiting and other important security settings.
    import '../imports/startup/server/security.js';

    // This defines all the collections, publications and methods that the application provides
    // as an API to the client.
    import '../imports/api/api.js';

然后在主服务器入口 `server/main.js` 导入这个启动模块。你可以看到，这里我们实际上并没有从这些文件中导入任何变量 - 我们导入它们是为了按照这个顺序执行。

**4.Meteor 伪全局导入**

为了向后兼容，Meteor 1.3 仍然为 Meteor 核心包以及应用程序中包含的其他 Meteor 包提供全局命名空间。你也仍然可以像以前版本的 Meteor 一样直接调用 `Meteor.publish` 等函数，而不需要先导入它们。然而，推荐的最佳做法是，在使用 Meteor "伪全局"之前，首先使用 `import { Name } from 'meteor/package’` 语法加载它们。比如说:

    import { Meteor } from 'meteor/meteor';
    import { EJSON } from 'meteor/ejson';

## 3.3 默认的文件加载顺序
尽管建议你在编写应用程序时使用 ES2015 模块和 `import/` 目录，但 Meteor 1.3 继续支持使用这些默认的加载顺序规则对文件急执行，以便向后兼容为 Meteor 1.2 及以前编写的应用程序。关于急执行、懒执行和懒加载之间的区别的，请参考 Stack Overflow 的这篇文章。

在应用程序中使用 import 时，你可以结合急执行和懒加载。当使用这些规则加载和执行一个文件时，任何导入语句都会按照它们在该文件中列出的顺序执行。

有几个加载顺序规则。它们按照下面给出的优先级，依次应用于应用程序中所有适用的文件：

- HTML模板文件总是在其他内容之前加载
- 以 `main.` 开头的文件最后加载
- 任何 `lib/` 目录内的文件都是接下来被加载
- 具有更深路径的文件接下来被加载
- 按整个路径的字母顺序加载文件

示例：

    nav.html
    main.html
    client/lib/methods.js
    client/lib/styles.js
    lib/feature/styles.js
    lib/collections.js
    client/feature-y.js
    feature-x.js
    client/main.js

例如，上面的文件是按照正确的加载顺序排列的。`main.html` 被加载在第二位，因为HTML模板总是先被加载，即使它以 `main.` 开头，因为规则1比规则2优先级高。然而，它将在 `nav.html` 之后被加载，因为规则2比规则5优先级高。

`client/lib/styles.js` 和 `lib/feature/styles.js` 在规则4之前有相同的加载顺序；然而，由于 `client` 按字母顺序排在 `lib` 之前，它将首先被加载。

> 你也可以使用 `Meteor.startup` 来控制代码在服务器和客户端的运行时间。

**1.特殊目录**

默认情况下，Meteor 应用程序目录中任何 JavaScript 文件都会被捆绑并同时在客户端和服务器上加载。然而，项目中文件和目录的名称会影响它们的加载顺序，加载的位置，以及其他一些特性。下面是一个被 Meteor 特别处理的文件和目录名称的列表:

- **imports**

  任何名为 `imports/` 的目录在任何地方都不会被加载，文件必须使用 `import` 导入。
  
- **node_modules**

  任何名为 `node_modules/` 的目录在任何地方都不会被加载。安装到 `node_modules` 目录中的 node.js 包必须使用 `import` 或在 `package.js` 中使用 `Npm.dependence` 导入。

- **client**

  任何名为 `client/` 的目录都不会在服务器上加载。这类似于将你的代码包装在 `if (Meteor.isClient) { … }`。 在生产模式下，所有在客户端加载的文件都会自动合并后被删除。在开发模式下，为了便于调试，JavaScript 和 CSS 文件不会被删除。为了保证生产和开发的一致性，CSS 文件仍然会被合并成一个文件，因为改变 CSS 文件的 URL 会影响文件中 URL 的处理方式。
  
> Meteor 应用程序中的HTML文件的处理方式与服务端的框架有很大的不同。Meteor 会扫描你目录中的所有HTML文件，寻找三个顶级元素：`<head>`, ` <body>` 和 `<template>`。`head` 和 `body` 部分被分别合并为一个单独的 head 和 body，并在初始页面加载时被传送到客户端。

- **server**

  任何名为 `server/` 的目录都不会在客户端加载。类似于用 `if (Meteor.isServer){ …}` 来包裹你的代，客户端永远不会收到这些代码。任何你不希望提供给客户端的敏感代码，例如包含密码或认证机制的代码，都应该保存在 `server/` 目录下。

  Meteor 收集你所有的 JavaScript 文件，除了 `client`、`public` 和 `private` 子目录下的任何文件，并将它们加载到一个 Node.js 服务器实例中。在Meteor 中，你的服务器代码在每个请求中以单线程运行，而不是Node 中典型的异步回调风格。
  
- **public**

  `public/` 顶层目录内的所有文件都以原样提供给客户端。当引用这些静态文件时，不要在URL中包含 `public/`，要把URL写成它们都在顶层的格式。例如，引用 `public/bg.png` 为 `<img src='/bg.png' />`。这是存放 `favicon.ico`、`robots.txt` 和其它类似文件最佳的地方。
  
- **private**

  `private/` 顶层目录内的所有文件只能从服务器代码中访问，并且可以通过静态API加载。这可以用于存放 私人数据文件 和 任何你不希望从外部访问的项目中的文件。
  
- **client/compatibility**

  这个文件夹是为了让那些依赖 `var` 声明的变量在被全局导出时与 JavaScript 库兼容。这个目录中的文件在执行时不会被包裹在一个新的变量范围内。这些文件会在其他客户端的 JavaScript 文件之前执行。
  
> 建议使用 npm 来获取第三方的 JavaScript 库，并使用 `import` 来控制文件何时被加载。

- **tests**

  `test/` 目录不会被加载到任何地方。使用这个目录存放任何使用Meteor 内置测试工具之外的测试运行器来运行的 测试代码。
  
以下目录也不会作为你的应用程序代码的一部分被加载：

- 以点开头的文件或目录，如 `.meteor` 和 `.git`
- `packages/`：用于本地软件包
- `cordova-build-override/`：用于高级的移动端 build 定制
- `programs`：历史遗留原因

**2.特殊目录以外的文件**

所有在特殊目录之外的 JavaScript 文件都会同时在客户端和服务器上加载。Meteor 提供了 `Meteor.isClient` 和 `Meteor.isServer` 变量，用来让你的代码判断是在客户端还是在服务器上运行从而改变其行为。

特殊目录之外的 CSS 和 HTML 文件只在客户端加载，不能从服务器代码中使用。

## 3.4 拆分成多个应用程序
如果你正在编写一个足够复杂的系统，把代码分割成多个应用程序是有意义的。例如，你可能想为 管理用户的UI 创建一个单独的应用程序（而不是全部通过网站管理的代码 检查权限，你可以检查一次），或者为应用程序的移动和桌面版本分离代码。

另一个非常常见的用例是将一个工作进程从你的主应用程序中分离出来，这样开销大的工作就不会因为被锁住一个网络服务器中而影响访问体验。

以这种方式拆分你的应用程序有一些好处：

- 如果你把特定类型的用户始终不会使用的代码分离出来，客户端 JavaScript 捆绑的代码就会明显变小。

- 你可以用不同的扩展设置来部署不同的应用程序，并以不同的方式保护它们（例如，可以只允许防火墙后面的用户访问 admin 应用程序）。

- 你可以允许你的组织中的不同团队独立处理不同的应用程序。

然而，以这种方式拆分你的代码有一些在使用之前就应该考虑的困难。

**1.代码共享**

主要的挑战是在不同的应用程序之间恰当地共享代码。处理这个问题的最简单方法是在不同的网络服务器上部署相同的应用程序，通过不同的设置控制行为。这种方法允许你部署具有不同扩展行为的不同版本的代码，但不能享受上面所说的大多数其他优势。

如果你想用独立的代码创建 Meteor 应用程序，你会有一些模块想在它们之间共享。如果这些模块是更广的范围可以使用的，你应该考虑将它们发布到一个包系统中，可以是 `npm`，也可以是 `Atmosphere`，这取决于代码是否是 Meteor 特有的。

如果这些代码是私有的，或者其他人不感兴趣，那么在两个应用程序中包含相同的模块通常是有意义的（你可以用私有的 npm 模块来做这个）。有几种方法可以做到这一点:

- 一个直接的方法是将公共代码作为两个应用程序的 git 子模块。
- 或者，如果你将两个应用程序都包含在一个仓库中，你可以使用符号链接将公共模块包含在两个应用程序中。

**2.数据共享**

另一个重要的考虑是如何在不同的应用程序之间共享数据。

最简单的方法是将两个应用程序都指向同一个 `MONGO_URL`，并允许两个应用程序直接从数据库中读取和写入。这种方法可行要感谢 Meteor 支持反应式数据库。当应用程序改变 MongoDB 中的一些数据时，由于 Meteor 的 livequery，连接到数据库的任何其他应用程序的用户将立即看到这些变化。

然而，在某些情况下，最好让一个应用程序成为主程序，并通过API控制其他应用程序对数据的访问。如果你想在不同的时间部署不同的应用程序，并且需要对数据的变化保守，那么这可以帮助你。

提供 server-server API 的最简单方法是直接使用 Meteor 内置的 DDP 协议。这与你的 Meteor 客户端从服务器获取数据的方式相同，但也可以用它来在不同的应用程序之间进行通信。你可以使用`DDP.connect()` 从一个"客户"服务器连接到主服务器，然后使用返回的连接对象来进行方法调用和从发布中读取。

**3.账号共享**

如果你有两个访问同一数据库的服务器，并且想让认证的用户在这两个服务器上进行 DDP 调用，可以使用一个连接上设置的令牌来登录另一个连接。

如果你的用户已经连接到服务器A，那么你可以使用 `DDP.connect()` 打开一个到服务器B的连接，并传入服务器A的令牌来认证服务器B。认证的代码看起来像这样：

    // This is server A's token as the default `Accounts` points at our server
    const token = Accounts._storedLoginToken();

    // We create a *second* accounts client pointing at server B
    const app2 = DDP.connect('url://of.server.b');
    const accounts2 = new AccountsClient({ connection: app2 });

    // Now we can login with the token. Further calls to `accounts2` will be authenticated
    accounts2.loginWithToken(token);

你可以在一个 [例子仓库](https://github.com/tmeasday/multi-app-accounts) 中看到这个架构的概念证明。

# 4 迁移到 Meteor 2.4

如何将你的应用程序迁移到 Meteor 2.4。

Meteor 2.4 中的大部分新功能都是直接在幕后应用的（以向后兼容的方式），或者是可选的。完整的变化，请参考 [changelog](http://docs.meteor.com/changelog.html)。

综上所述，有一些事情需要先做，以便给将来节省更多的时间。

**createIndex**

以前没有记录的 `_ensureIndex` 已经与 MongoDB 在命名上的重大变化相一致，现在可以作为 `createIndex` 使用。`_ensureIndex` 已被弃用，在开发中使用时会抛出一个警告。

**Email 2.2**

电子邮件包有一个功能更新。你现在可以用`Email.customTransport` 完全覆盖发送功能，或者 如果你使用了已知的服务，现在可以抛弃 `MAIL_URL` 环境变量，通过在 `settings.json` 文件中这样设置：

    {
      "packages": {
        "email": {
          "service": "Mailgun",
          "user": "postmaster@meteor.com",
          "password": "superDuperPassword"
        }
      }
    }

## 4.1 从2.3以前的版本迁移?

如果你要从比 Meteor 2.3 更早的版本迁移，可能会有一些重要的注意事项没有在本指南中列出（本指南特别涵盖了从 2.2 到 2.3）。请查看更早的迁移指南以了解细节:

- [Migrating to Meteor 2.3](https://guide.meteor.com/2.3-migration.html)(from 2.2)
- [Migrating to Meteor 2.2](https://guide.meteor.com/2.2-migration.html)(from 2.0)
- [Migrating to Meteor 2.0](https://guide.meteor.com/2.0-migration.html)(from 1.12)
