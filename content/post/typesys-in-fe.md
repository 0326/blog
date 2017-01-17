+++
date = "2016-09-23T01:42:50+08:00"
title = "类型系统-前端进化的里程碑"
description = "TypeScript或许只是现在，类型系统一定是未来"
draft = false
+++


大半夜的JavaScript Weekly发来贺电：[TypeScript 2.0 Final Released！](https://blogs.msdn.microsoft.com/typescript/2016/09/22/announcing-typescript-2-0/#comment-26185)

没错，继Angular2发布之后，TypeScript今天也发布了2.0版本，这不禁让我浮想一番。如果要说TS和JS最明显的差别，我想一定是Type System，所以今天我们就聊聊类型系统在前端发展历程中，到底扮演了怎样的角色。

## 历史斗争
如果要你把PV上百万级别的Web Application用一门在10天内撸出来的编程语言来开发，我想你肯定不会放心的。然而事实上我们现在都是这样干的，JS已经成为了最流行的编程语言。JS现在所承担的使命已经完全超出了当年设计的初衷，虽然TC39一直在填坑，并且发展到如今的ES6已经相当成熟了，但仍然留下了一些历史包袱，并不能改变这是一门动态弱类型脚本语言的实质。

因此在前端工程化不断壮大的过程中，为了避免踩坑，人类同JS最佳编码实践方式展开了旷日持久的战争。

最开始，大家都只是取其精华，去其糟粕，如《JavaScript语言精粹》一书所说：你们只需要用我说的就好了，其他的垃圾都不要学，并且千万不要在项目里面用。

一般情况下每个公司都会出一套最佳实践的编码规范，程序员需要统一代码风格，按约定编写代码。但规范的约束力很低，结果在项目赶着上线的情况下还是写出了翔一样的代码，所以更好的方式是用工具来规范代码，发现一些潜在问题，通过工具来强制约定编码。比如JSLint，JSHint，以及ESLint，都是设定了一系列编码约定，让你避免写出一些糟糕的代码。

另外一种思路，就是抛弃使用JS作为开发语言，或者只是把他当成“JVM”,然后采用另外一种设计更加严谨，不容易采坑的语言来编程，比如CoffeeScript和TypeScript,开发完后再转译成JS来运行。

如果觉得这种方式过于激进，那么可以采用渐进的方式，比如Flow。Flow可以在开发时对代码进行静态类型分析，用写强类型的方式来写弱类型的JS。实质上这有很多好处：
1. 强制声明类型，IDE和编辑器可以通过静态类型分析发现代码隐藏缺陷，同时也能够提供更强大的自动补全，智能代码提示和纠错，达到Java/C++级别的开发体验。
2. 可避免类型隐式转换带来的消耗，提高运行效率。实际上JS引擎在运行时很大的开销都花在类型分析上。
3. 可读性/可维护性增强。一眼就能看出这个变量是String还是Number，代码维护也更清晰，并且通过注释工具生成的代码注释也会更加详细，后面换人维护时也更容易上手。

这些优势，其实都是类型系统所带来的强类型语言所具有的开发优势，无论是在开发体验还是后期项目维护上，都要优于目前的JavaScript。

接下来，我们就以渐进的方式，来感受一下类型系统带给我们的好处。

## 类型系统

### Flow.js
很多情况下我们都是在维护项目，不可能为了增加类型检查来修改老的项目代码。Flow可以在不修改代码的情况下，通过注释的方式来进行静态类型分析，这为我们提供了一个很好的过渡方式。你可以随时在任一个项目里面集成Flow。
```javascript
/*
* @flow 
* 只需要在文件头部添加flow注释，Flow就会认为这个文件需要静态分析并检查
*/

function foo(x) {
  return x * 10;
}

// 这样调用Flow就会给出错误提示：string和number类型不兼容
foo('Hello, world!'); 
```
这种无侵入式的集成，可以检测出一些比较低级的错误，如果要支持更多强大的分析，就需要写侵入代码了，比如手动类型注释：
```javascript
/* 
* @flow 
* var : [type] 指定变量类型
*/

function add(num1: number, num2: number): number {
  return num1 + num2;
}

// 这样调用就会报错，因为参数2已经被声明为number了
var x: number = add(3, '0');
```
这样的代码是不能直接运行的，还是需要Flow工具转译成原生JS才能执行。这种方式就更适合新的项目，一旦新项目直接集成了Flow套餐，就可以直接使用Flow支持的更多功能，并且配合IDE给出更好的开发体验。

以Mac下的VSC为例，首先安装本地Flow环境：
```shell
brew update
brew install flow
```
然后在VSC中安装启用vscode-flow插件,  ⌘+' 打开用户配置，禁用VSC自带的JavaScript校验功能(设置javascript.validate.enable为 false)，并设置好flow的安装目录：
![图片描述][1]

剩下的套路就跟Babel，ESLint一样了，在项目根目录下面建立一个.flowconfig文件，配置一些校验规则：
![图片描述][2]


vscode-flow插件检测到.flowconfig配置后就会启动flow服务去实时分析项目代码，当你开发的时候就能感受到比原生编辑器更加强大的自动补全和智能提示了。比如当你require一个util模块时，flow能分析出util模块内结构，并且当你调用util方法不当时给出提示：
![图片描述][3]


以上只是介绍简单流程，并且还是无侵入式的校验，如果再加上手动类型声明的话，还能提供更多功能。





### TypeScript

TS的做法更彻底，如果有一个全新的项目可以自由选择技术方案的话，我一定会选TypeScript而不是Flow.js。可惜的是，在公司里面大部分时候都依赖公司自身的技术体系，在做技术选型的时候都要依赖团队的技术栈。就比如大家都用ES6，你选择TypeScript，那么之后别人来维护你的代码成本就非常高，除非你能煽动整个团队，整个集团使用：）一般情况下这是不可能的，我想这也是TS难以普及的重要原因。

但是，这并不妨碍TypeScript成为一门优雅的前端开发语言。ES6有的它都有，ES6没有他也有（泛型/枚举/类型推导等只有强类型语言才有的一些特性），而这些特性恰恰更加适合日益壮大的工程化的前端，适合编写出可维护性代码。再配合微软自家的VSC，开发体验妥妥的：
![图片描述][4]

至于TypeScript 2.0带来了哪些新特性，请直接戳GitHub：
https://github.com/Microsoft/TypeScript/wiki/What%27s-new-in-TypeScript

## 未来趋势
前几日GitHub 发布了2016开源报告，JavaScript众望所归的荣登榜首，让众前端激动不已：
![图片描述][5]

然而让我意外的不是排在第一的JavaScript，而是最后的TypeScript：
![图片描述][6]

![图片描述][7]

看这增长趋势，微软是要协TypeScript在开源之路上越走越远了。


私认为，无论最后是不是TypeScript，类型系统都带来了更好的开发体验，代码质量，代码可读性和可维护性，这正是一个大型或长期项目所必须的，也是现在和未来的前端工程所需要的。所以实在是没有不学的理由，如果你觉得TypeScript像极了C#更适合后端程序员，那么学习它或许是你迈向全栈的一小步哈哈。


  [1]: //segmentfault.com/img/bVDu2Y
  [2]: //segmentfault.com/img/bVDsq4
  [3]: //segmentfault.com/img/bVDu4k
  [4]: //segmentfault.com/img/bVDvaS
  [5]: //segmentfault.com/img/bVDsql
  [6]: //segmentfault.com/img/bVDsqn
  [7]: //segmentfault.com/img/bVDsqo