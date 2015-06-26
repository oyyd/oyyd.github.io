#107 Room Front-End Refactor Data

##原则

1. 不改变后端架构。

2. 能平稳过度（新方式能与当前的开发模式协同进行）。

3. 尽可能选择学习代价最小的方式。

##关于引入Node的解释

1. 引入Node并不会改变当前系统架构，它是前端开发环境的需求用来进行编译打包工作
，并不影响其他工种，其他人和线上环境也不需要安装。

2. 前端开发人员进行开发几乎不需要额外学习Node（只需要懂Node的模块）。

##压缩代码（js, css）

###Why?

1. 减少js，css文件大小（uglifyjs可减少30%-40%左右的js文件大小）

2. 别人很难看懂压缩过的js代码

##js模块

###Why?

__<script>（global）:__

1. IO阻塞（外部文件）

2. 需要按照代码的依赖关系，依次顺序引入代码（实际上就是人肉管理代码依赖）。

3. 不注意分割js文件的话，文件代码容易变得很长，不利于维护；文件分割过细会导致文件请求数量变大（Yahoo前端优化的要点之一：减少请求文件数量）。

###可选解决办法

####两类模块选择

__Commonjs:__

Commonjs做法近似一般后端脚本语言。

优点：简洁，解决依赖问题。

问题：浏览器环境不同于其他环境，获取任何模块都需要发请求，不打包的话没办法解决IO阻塞问题。

```js
var User = require('path/to/User.js');// node module
```

可用库：Node的模块, Seajs（淘宝）,

__AMD:__

AMD会根据依赖自动异步加载脚本，即互相不依赖的脚本会被同时加载，以减轻IO阻塞带来的问题。

可用库：RequireJs

```js
require(['scripts/config', 'dependency'], function() {
    // start
});
```

问题：AMD模块稍微繁杂，而且不解决

####主流做法（推荐）

__Commonjs + 打包：__

（PS1：AMD的特点是异步加载，但打包的话就没有这一特点了，所以这里不提AMD + 打包）

打包好处：可以大量减少请求文件的数量

打包常用工具：webpack, browserifyjs

##预编译css

###Why?

0. Less百分百兼容css，我们完全可以不改变现有css代码。

1. 我们需要一些常见的，但css本身没有的功能，如：

__变量:__

参见我们的_style.vm文件。

___更加直观的层级，css代码更容易维护:__

是要继续：

```css
.component{
  width: 100px;
  height: 100px;
}
.component .header{
  color: red;
}
.component .header .title{
  font-size: 30px;
}
.component .header .footer{
  ...
}
```

还是要？：

```less
 @size = 100px;
 @mainColor = red;
 @bigFont = 30px;
 .component{
  width: @size;
  height: @size;
  .header{
    color: @mainColor;
    h1.title{
      font-size:@bigFont;
    }
  }
  .footer{
    //...
  }
 }
```

发散：对于css和less而言，如果要改变component的size的话，各需要改变多少代码？如果我们不得不改变顶级的component的话，各需要改多少地方？

__import:__

```less
@import "../component_style.less";
```

2. Less非常简单，几乎不需要额外学习。

3. 配合预编译，我们可以把代码从vm文件中分离出来，让vm文件专注于业务。
（部分vm文件大小可以分离出500行以上的css代码，至少不会看得烦）

4. 优雅：为了减少css被意外作用在class上，我们常常需要写很长的前缀来限制css的作用范围。而使用less可以减少这种痛苦。

5. 减少java渲染vm文件时的性能消耗。

（PS：因为使用CSS预编译工具有大量的好处，而几乎不会带来负面作用，这种做法终将成为主流，所以这里我也不多赘述）

##前端开发模式的讨论

这里只探讨（个人觉得）最有价值的几种：

* jquery only（或是其他jquery类型的库）

* Angular

* React

Angular：google angular团队开发，主要负责人misko本身长期负责java上的开发和质量保证，Angular吸取了很多java（spring）的经验。

React：facebook开发。Instagram网站，阿里（支付宝部分，天猫部分）

先表明立场：jquery很好很强大，它简单易用，非常灵活，可以应对大量应用场景。同时其生态系统又丰富而完善，
大大小小的前端库都会基于jquery。

但灵活的代码问题就是难以维护。接下来我会参杂其他一些大牛的文章来探讨一下这个问题。

测试代码

在公司学习java时，我觉得（我们公司的）java代码很优秀，很适合生产环境，很好维护很好扩展。我觉得自己会产生这种想法的原因是，在扩展公司的代码时，很容易想到。结合Yahoo前端大牛Douglas Crokford说过的一句话：

