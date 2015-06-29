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
var House = {};
module.exports = House;
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

```css
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

我们想要：

1. 代码中提供相似功能的component只有一个

2. 别人可以方便地使用component，而不需要去了解其细节

3. 同样的数据，在所有components中只有一份

4. 更新数据即可该改变component的表现

###我在开发中遇到的一些问题

* 一个部件是否要独立写出来？(price-dragger.vm, HouseInfoCircle.js)

UI上的东西都很可能被重复使用，也应该被重复使用。并且以一个component的角度出发来写一个东西并不会比写一个一次性的东西要花费更多的时间。

* 一个部件虽然独立了出来，使用也没问题，但却没办法在一个页面中使用多次

$()是全局搜索，会对其他相同的部件产生影响

* 封装成独立的jquery对象或jquery plugin的方式虽好，但他们需要在js中手动生成大量DOM

这里只探讨（个人觉得）最有价值的几种：

* jquery only（或是其他jquery类型的库）

* Angular

* React

###Angular

google angular团队开发，主要负责人misko本身长期负责java上的开发和质量保证，Angular吸取了很多java（spring）的经验。

###React

facebook开发。

Instagram网站，阿里（支付宝，淘宝，天猫部分）, AirBnb等。链接：[Who use react?](https://github.com/facebook/react/wiki/Sites-Using-React)

特性：

0. React可能是对UI抽象最优秀的框架，这使得用React构建的部件非常易于分割和组合，数据变化非常容易被追踪和控制。

1. Virtual DOM提高性能，性能仅次于native。benchmark结果：

native js > react > jquery > angular

2. 很容易学习

3. 只是MVC中的V，不提供冗余的特性。

链接：[Removing User Interface Complexity, or Why React is Awesome](http://jlongster.com/Removing-User-Interface-Complexity,-or-Why-React-is-Awesome)

###探讨

先表明立场：jquery很好很强大，它简单易用，非常灵活，可以应对大量应用场景。同时其生态系统又丰富而完善，
大大小小的前端库都会基于jquery。

但：

* 原生javascript和jquery都不是声明式，他们需要大量进行DOM query

测试代码

在公司学习java时，我觉得（我们公司的）java代码很优秀，很适合生产环境，很好维护很好扩展。我觉得自己会产生这种想法的原因是，在扩展公司的代码时，很容易想到。结合Yahoo前端大牛Douglas Crokford说过的一句话：

##总结

0. 用Less取代css，用Less来提供本来是由velocity提供的css功能补足，将对样式（样式代码）的关注从vm中分离。

1. 划分js模块，缩小单个js文件代码的粒度，提高可复用性。

2. 将代码的重点放入到分离的静态js中（从而更好地划分模块和打包），尽可能只让vm进行高层次上的代码组合和数据，UI初始化，细节应该由js自行处理。

3. 由React构建UI，并保证两点：1. UI尽可能的无状态性（stateless，即在React中尽可能由props代替state）2. 将state尽可能地集成到顶层
