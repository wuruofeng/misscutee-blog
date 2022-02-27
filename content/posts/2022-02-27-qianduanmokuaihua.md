---
title: 前端模块化的前世今生
author: RuoFeng
date: 2022-02-27
tags: ["tec","FrontEnd"]
slug: front-end-modularity

---
> 前端模块化是前端工程化所不可或缺的重要一环。

# 前言

​	与十年之前只用jQuery就可以解决一切的情况完全不同的是，如今的前端生态可谓是遍地开花，百花齐放。各种框架层出不穷，像什么大名鼎鼎的Vue，React，~~Angular~~ ，还有一堆耳熟能详的UI框架，比如Antd，ElementUI等等，这些项目背后的工程文件大多都是数以千计的零碎代码，而在我们自己的项目中如果需要使用他们，只需要一个<kbd>script</kbd>标签或者一个<kbd>import</kbd>即可将这些模块一整个的导入，十分的方便。这便是前端的模块化。  

​	一个全球通行的技术产物是一定要有一个严格的规范的，小柴当年也曾接触到过诸如AMD、CMD、ES Module等各式各样的规范，但是未曾想过这些规范之间的区别，也更加不知道当我们在项目中require或者import的背后到底发生了什么。本文着重讲解一下模块化的规范与演进，希望诸君阅读后能对前端模块化有个比较全面的认知～



# 一、模块化的内涵

## 1.为什么需要模块化？

​	小柴写了一段代码，用来计算班级的平均分，小哈也写了一个计算班级成绩指标的方法，小哈想在自己的方法中引用小柴的代码，这时候该怎么做呢？

### 1.1 无模块化

```html
<script>
  // 小柴计算平均分的代码
  function calAverageScore() {
    // ...
  };
  // 小哈的代码
  function calTotalScore() {
    // ...
    calAverageScore();
  }
</script>

或

<script type="text/javascript" src="calAverageScore.js"></script>
<script type="text/javascript" src="calTotalScore.js"></script>
```

在无模块化的时代，开发者想要引用别人的代码要么只能把别人的代码全盘拷到自己的代码里面，要么借助Ajax等工具去引入一些JS文件，这样一来会有两个问题：

- 容易造成全局变量的污染
- 如果有依赖关系，那么依赖关系的管理将会是一场灾难

大家的方法难免会定义一些全局变量，当第三方进行整合的时候发现两个方法定义了同一个全局变量的就悲剧了，需要自己手动改，这无疑是徒增烦恼。第二点是因为假设A的代码依赖B，而B的代码又依赖C，这就要求开发者按顺序管理好这些代码文件，文件少还好说，假设文件的规模达到了上百个，那就完犊子了。

### 1.2 早期的解决方式

- **命名空间**

```javascript
let myModule = {
  data: 'chaichaiCoding',
  foo() {
    console.log(`foo() ${this.data}`)
  },
  bar() {
    console.log(`bar() ${this.data}`)
  }
}
```

命名空间的作用便是将全局变量收敛到一个对象中去，这样子就能解决全局变量污染的问题，但是问题也很明显，就是内部变量是能被外部更改的，数据不安全。这时候聪明的人们就想出了闭包的方式。

- **闭包(IIFE匿名函数的立即执行)**

```javascript
(function(window, $) {
  let data = 'chaichaiCoding'
  function foo() {
  }
  function bar() {
    otherFun() // 内部调用
  }
  function otherFun() {
    // 内部私有的函数
    console.log('otherFun()')
  }
  // 挂载到window上
  window.myModule = { foo, bar }
})(window, jQuery)

// IIFE增强，一方面能引用其他库作为依赖，一方面由于闭包的特性，保证内部变量不被外部更改
```

这样就达到了模块封装的目的，大名鼎鼎的jQuery便是用这种方式。综上，我们总结出前端模块化的一些要点如下：

![image-20220226131625471](https://chaichai-coding-1256181968.cos.ap-chengdu.myqcloud.com/typora/image-20220226131625471.png)

# 二、模块化的规范

![image-20220226131902837](https://chaichai-coding-1256181968.cos.ap-chengdu.myqcloud.com/typora/image-20220226131902837.png)

模块的呈现形式无非就是模块的定义和使用两方面。接下来的篇幅柴柴将会按顺序讲解CommonJS(CJS)，AMD，CMD以及最新的ES Module(ESM)之间到底有什么不同。

## 1.CommonJS与Node 的那些事儿

得益于Node的诞生，JavaScript也可以写服务端的代码。在服务端而言模块化是必不可缺少的，于是，CommonJS规范便应运而生。



>- Nodejs的模块化能一种成熟的姿态出现离不开CommonJS的规范的影响
>
>- 在服务器端CommonJS能以一种寻常的姿态写进各个公司的项目代码中，离不开Node的优异表现
>
>- Node并非完全按照规范实现，针对模块规范进行了一定的取舍，同时也增加了少许自身特性
>
>  ​																													--朴灵《深入浅出Nodejs》

### 

在CommonJS中，每一个文件都被视为独立的模块。例如，假设一个名为foo.js的文件:

```javascript
const circle = require('./circle.js');
console.log(`The area of a circle of radius 4 is ${circle.area(4)}`);
```

在第一行，`foo.js` 加载了与 `foo.js` 位于同一目录中的模块 `circle.js`。

以下是 `circle.js` 的内容：

```javascript
const { PI } = Math;

exports.area = (r) => PI * r ** 2;

exports.circumference = (r) => 2 * PI * r;
```

模块 `circle.js` 已导出函数 `area()` 和 `circumference()`。 通过在特殊的 `exports` 对象上指定额外的属性，将函数和对象添加到模块的根部。这样一来我们就能导入计算圆周长和面积的模块，从而实现在自己的模块中计算相关信息的功能。在浏览器中我们可以借助诸如Browserify或者WebPack等打包工具把各种require的文件打到一个bundle里面去供浏览器调用。

## 2.AMD与RequreJS 的那些事儿

上述我们谈到Node中的CommonJS的规范，但是有一个很重要的点就是，这种规范天生就不适用于浏览器，因为它是同步的。在服务端各类文件都在本地，受限制的是硬盘的读写速度，而在浏览器端，浏览器端每加载一个文件，要发网络请求去取，如果网速慢，就非常耗时，浏览器就要一直等 `require` 返回，就会一直卡在那里，阻塞后面代码的执行，从而阻塞页面渲染，使得页面出现假死状态。这时候受限制的就是网速了。于是，应声而出的**AMD(Asynchronous Module Definition，异步模块加载机制)**便出现了。

![image-20220226150639568](https://chaichai-coding-1256181968.cos.ap-chengdu.myqcloud.com/typora/image-20220226150639568.png)

来看看 AMD 规范的实现：

```html
<script src="require.js"></script>
<script src="a.js"></script>
```

首先我们引入 `require.js` 工具库，这个库提供了定义模块、加载模块等功能。它提供了一个全局的 `define` 函数用来定义模块。所以在引入 `require.js` 文件后，再引入的其它文件，都可以使用 `define` 来定义模块。

```js
define(id?, dependencies?, factory)
```

- id：可选参数，用来定义模块的标识。

- dependencies：可选参数，是一个**数组**，表示当前模块的依赖，如果没有依赖可以不传。

- factory：工厂方法，模块初始化后执行的回调函数或对象。如果为函数，它应该只被执行一次，返回值便是模块要导出的值。如果是对象，此对象应该为模块的输出值。

  

  所以模块A可以这么定义

```js
// a.js
define(function(){
    var PI = 3.14159
    return {
        PI,
        getArea: (r) => PI * r ** 2
    }
})
// b.js
define(['a.js'], function(a){
    var name = 'chaichaiCoding'
    console.log(a.PI) // '3.14159'
    console.log(a.getArea(4)) 
    return {
        name,
    }
})
```

它采用异步方式加载模块，也就是说，模块的加载需要等待入参中dependencies这个数组对应的所有模块加载好后才执行回调函数，模块的加载不影响它后面语句的运行。

RequireJS 的基本思想是，通过 define 方法，将代码定义为模块。当这个模块被 `require` 时，它开始加载它依赖的模块，当所有依赖的模块加载完成后，开始执行回调函数。与下文柴柴所要说的的SeaJs不同，他是**依赖前置**的。

## 3.CMD与SeaJS 的那些事儿

和 AMD 类似，CMD 是 Sea.js 在推广过程中对模块定义的规范化产出。Sea.js 是阿里的玉伯写的。它的诞生在 RequireJS 之后，玉伯觉得 AMD 规范是异步的，模块的组织形式不够自然和直观。于是他在追求能像 CommonJS 那样的书写形式。于是就有了 CMD 。

来看看 CMD 规范的实现：

```html
<script src="sea.js"></script>
<script src="a.js"></script>
```

与`require.js`类似，我们引入`sea.js`的工具库。

```js
// 所有模块都通过 define 来定义
define(function(require, exports, module) {
  // 通过 require 引入依赖
  var a = require('xxx')
  var b = require('yyy')
  // 通过 exports 对外提供接口
  exports.doSomething = ...
  // 或者通过 module.exports 提供整个接口
  module.exports = ...
})
// a.js
define(function(require, exports, module){
    var name = 'chaichaiCoding'
    var PI = 3.14159
    exports.name = name
    exports.getArea = (r) => PI * r ** 2
})
// b.js
define(function(require, exports, module){
    var name = 'xiaoha'
    var r = 4
    var a = require('a.js')

    console.log(a.name) // 'chaichaiCoding'
    console.log(a.getArea(r)) //  计算半径为4的面积

    exports.name = name
    exports.getArea = () => a.getArea(r)
})
```

> **Sea.js 可以像 CommonsJS 那样同步的形式书写模块代码的秘诀在于： 当 b.js 模块被 `require` 时，b.js 加载后，Sea.js 会扫描 b.js 的代码，找到 `require` 这个关键字，提取所有的依赖项，然后加载，等到依赖的所有模块加载完成后，执行回调函数，此时再执行到 `require('a.js')` 这行代码时，a.js 已经加载好在内存中了**

## 4.ESM的那些事儿

如前面所述，CommonJS和AMD都是运行时加载。ES6在语言规格层面上实现了模块功能，是编译时加载，完全可以取代现有的CommonJS和AMD规范，可以成为浏览器和服务器通用的模块解决方案。这里关于ES6模块我们项目里使用非常多，所以详细讲解。

来看看 ESM 规范的实现：

```js
// a.js

export const name = 'ChaiChaiCoding'
const PI = 3.14159
export function getArea (r) {
    return PI * r ** 2
}

//等价于
const name = 'ChaiChaiCoding'
const PI = 3.14159
export function getArea (r) {
    return PI * r ** 2
}
export {
    PI,
    getArea
}
```

使用 `export` 命令定义了模块的对外接口以后，其他 JavaScript 文件就可以通过 `import` 命令加载这个模块。

```js
// b.js
import { PI, getArea } from 'a.js'
const area = getArea(4)
console.log(area) // PI * 4 ** 2

// 等价于
import * as a from 'a.js'
console.log(a.PI) // '3.14159'
const area = getArea(4)
console.log(area) // PI * r ** 2
```

除了指定加载某个输出值，还可以使用整体加载，即用星号（*）指定一个对象，所有输出值都加载在这个对象上面。

ES6模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。所以说ES6是**编译时加载**，不同于CommonJS的运行时加载(实际加载的是一整个对象)，ES6模块不是对象，而是通过export命令显式指定输出的代码，输入时也采用静态命令的形式：

```js
//ES6模块
import { basename, dirname, parse } from 'path';

//CommonJS模块
let { basename, dirname, parse } = require('path');
```

以上这种写法与CommonJS的模块加载有什么不同？

- 当require path模块时，其实 CommonJS会将path模块运行一遍，并返回一个对象，并将这个对象缓存起来，这个对象包含path这个模块的所有API。以后无论多少次加载这个模块都是取这个缓存的值，也就是第一次运行的结果，除非手动清除。
- ES6会从path模块只加载3个方法，其他不会加载，这就是编译时加载。ES6可以在编译时就完成模块加载，当ES6遇到import时，不会像CommonJS一样去执行模块，而是生成一个动态的只读引用，当真正需要的时候再到模块里去取值，所以ES6模块是动态引用，并且不会缓存值。

# 三、总结

1.CommonJS是为了解决Node中的模块化问题而提出的。

2.AMD是为了解决浏览器中无法像服务端同步加载所提出的异步规范，有个很重要的特点就是**依赖前置**。

3.CMD是国内大佬为了像CommonJS那样的书写方式，提出的一种规范，有个很重要的特点就是**就近原则**。

4.上述三种规范都是运行时，而只有ESM是在编译时。**完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案**。





欢迎搜索关注**柴柴爱Coding**微信公众号，这里有  ~~**免费的学习资源、全方位的进阶路线、各岗位面试资源、程序设计源码**~~   一只会Coding的柴柴等你哦~