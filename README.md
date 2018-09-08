# babel那些事儿

## 目录
* [一 babel是什么](#一-babel是什么)
* [二 Babel的工作原理](#二.Babel的工作原理)
* [三 使用方法](#三-使用方法)
  * .babelrc
  * presets
  * env
  * 与构建工具集成
* [四 babel包介绍](#四-babel包介绍)
  * [(一) 核心包](#(一)-核心包) 
      * [babel-core](#babel-core)
  * [(二) 功能包](#(二)-功能包)
      * [babel-polyfill](#babel-polyfill)
      * [babel-runtime 和 babel-plugin-transform-runtime](babel-runtime-和-babel-plugin-transform-runtime)
  * [(三) 工具包](#(三)-工具包)
      * [babel-cli](#babel-cli)
      * [babel-node](#babel-node)
      * [babel-register](#babel-register)
* [六 bable7.x](#babel-7.x)

----
### 一 babel是什么
> Babel is a JavaScript compiler

简单来说就是把 JavaScript 中 es2015+ 的新语法转化为 es5，让低端运行环境 (如浏览器和 node ) 能够认识并执行

### 二 Babel的工作原理
babel是一个编译器compiler，但是叫转译器transpiler更准确，因为它只是把同种语言的高版本规则翻译成低版本规则，而不像编译器那样，输出的是另一种更低级的语言代码
Babel的编译过程跟绝大多数其他语言的编译器大致同理，分为三个阶段：

1. 解析(parse)：使用babylon将代码字符串解析成抽象语法树ast 
2. 变换(transform)：使用babel-traverse, 利用配置好的 plugins/presets遍历ast，更新节点，转变为新的ast
3. 再建(generate)：使用babel-generator根据变换后的抽象语法树再生成代码字符串 

babel编译流程图
![编译流程图](http://on-img.com/chart_image/5b91e61ce4b06fc64ae8f146.png "编译流程图")

我们来了解一下抽象语法树
[AST Explorer](https://astexplorer.net/)是一个在线实时编译工具，可以将源代码解析成抽象语法树, 帮助我们更直观的认识AST的结构，编译器选择babylon6,运行下面的的代码
```javascript
function abs(number) {
    if (number >= 0) {  
        return number;  
    } else {number; 
    }
}
```

为了更加直观，可以用结构图表示如下,可以看到AST就是由一个个节点构成，每个节点都是源代码语法的一个标签。
![!ast树](http://on-img.com/chart_image/5b932c6ee4b0534c9bd10d2a.png)

解析这一步是如何生成抽象语法树的, 给大家甩一个知乎上的一篇文章[Babel是如何读懂JS代码的](https://zhuanlan.zhihu.com/p/27289600)里面有讲解析器解析的过程,  涉及到编译器的部分，有兴趣的话，可以研究一下github上一个轻量级的编译器实现[the-super-tiny-compiler](https://github.com/jamiebuilds/the-super-tiny-compiler)

**此外,以下两点请注意**：
1.  babel本身不具备转码能力，转码都是通过插件完成的，如果不添加插件，那么经过babel后的代码和源代码是一样的
```javascript
let x = (a) => a*2;
```
2.  babel只是转译新标准引入的语法，比如ES6的箭头函数转译成ES5的函数；而新标准引入的新的原生对象(Symbol)，部分原生对象新增的原型方法（数组的include的方法），新增的API等（如Proxy、Set等），这些babel是不会转译的。需要用户自行引入polyfill(后面会介绍)来解决。
```javascript
let obj = Object.assign({},{a:1});
```

### 三 使用方法

Babel的配置文件是.babelrc，存放在项目的根目录下。使用Babel的第一步，就是配置这个文件。
```javascript
{
  "presets": [],
  "plugins": []
}
```

插件和 preset 的配置项
简略情况下，插件和 preset 只要列出字符串格式的名字即可。但如果某个 preset 或者插件需要一些配置项(或者说参数)，就需要把自己先变成数组。第一个元素依然是字符串，表示自己的名字；第二个元素是一个对象，即配置对象
```javascript
{"presets": [
    // 带了配置项，自己变成数组
    [
        // 第一个元素是preset的名称
        "env",
        // 第二个元素是对象，列出该preset的配置项
        {
          "module": false
        }
    ],

    // 不带配置项，直接列出名字
    "stage-2"
]}
```
执行顺序
+ Plugin 会运行在 Preset 之前。
+ Plugin 会从前到后顺序执行。
+ Preset 的顺序则 刚好相反(从后向前)。
+ preset 的逆向顺序主要是为了保证向后兼容，因为大多数用户的编写顺序是 ['es2015', 'stage-0']。这样必须先执行 + stage-0 才能确保 babel 不报错。

#### presets
 babel 本身不具有任何转化功能，它把转化的功能都分解到一个个 plugin 里面, 插件是在转换这一阶段起作用的。当我们不配置任何插件时，经过 babel 的代码和输入是相同的。
那么如何配置插件呢，比如 es2015 是一套规范，包含如下二十多个转译插件,
+ check-es2015-constants
+ transform-es2015-arrow-functions
+ transform-es2015-block-scoped-functions
+ transform-es2015-block-scoping
+ transform-es2015-classes
+ transform-es2015-computed-properties
+ transform-es2015-destructuring
+ transform-es2015-duplicate-keys
+ transform-es2015-for-of
+ transform-es2015-function-name
+ transform-es2015-literals
+ transform-es2015-modules-commonjs
+ transform-es2015-object-super
+ transform-es2015-parameters
+ transform-es2015-shorthand-properties
+ transform-es2015-spread
+ transform-es2015-sticky-regex
+ transform-es2015-template-literals
+ transform-es2015-typeof-symbol
+ transform-es2015-unicode-regex
+ transform-regenerator

如果一个个添加然后安装，就非常麻烦, babel官方就提供了插件合集, 省去了开发者配置的麻烦，这就叫presets,分为以下几种
 - 官方
    + **env**
    + react
    + flow
    + minify
- stage-x
    + Stage 0 - 稻草人(strawman): 只是一个想法，经过 TC39 成员提出即可。
    + Stage 1 - 提案(proposal): 初步尝试。
    + Stage 2 - 初稿(draft): 完成初步规范。
    + Stage 3 - 候选(candidate): 完成规范和浏览器初步实现。
- es201x, latest
    但因为 env 的出现，使得 es2016 和 es2017 都已经废弃。latest 是 env 的雏形，它是一个每年更新的 preset，目的是包含所有 es201x。但也是因为更加灵活的 env 的出现，已经废弃。
#### env
 env 的核心目的是通过配置得知目标环境的特点，然后只做必要的转换。例如目标浏览器支持 es2015，那么 es2015 这个preset 其实是不需要的，于是代码就可以小一点(一般转化后的代码总是更长)，构建时间也可以缩短一些。
如果不写任何配置项，env 等价于 latest，也等价于 es2015 + es2016 + es2017 三个相加(不包含 stage-x 中的插件)。env 包含的插件列表维护在[env插件列表][env-plugin-list]

#### 与构建工具集成
babel一般是集成到构建工具里面使用的，这时需要安装构建工具的插件 (webpack 的 babel-loader, rollup 的 rollup-plugin-babel)，以目前使用最频繁的打包工具的webpack为例

配置文件.babelrc如下，存放在项目的根目录下
```
{
  "presets": [
    ["env", { "modules": false }],
    "stage-2"
  ],
  "plugins": ["transform-runtime"]
}

```
然后webpack对应的配置
```
 module: {
    rules: [
      {
        test: /\.js$/,
        use: ['babel-loader'],
        exclude: /node_modules/,
      }
    ]
  }
```
### 四 Babel包介绍
[babel包](https://github.com/babel/babel/tree/master/packages)  
[在线运行repl环境](https://babeljs.io/en/repl#?babili=false&browsers=&build=&builtIns=false&spec=false&loose=false&code_lz=DYUwLgBAhhC8EAoEEo4D4IA8BUAmZC-AUEUA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=&prettier=false&targets=&version=6.26.0&envVersion=)

#### (一) 核心包
+ babel-core：babel转译器本身，提供了babel的转译API，如babel.transform等，用于对代码进行转译。像webpack 的babel-loader就是调用这些API来完成转译过程的。
+ babylon：js的词法解析器
+ babel-traverse：用于对AST的遍历，主要给plugin用
+ babel-generator：根据AST生成最终的代码

babel-core的使用
```javascript
var babel = require('babel-core');

// 字符串转码, transform方法的第一个参数是一个字符串，表示需要转换的代码，第二个参数是转换的配置对象
babel.transform('code', options);
// => { code, map, ast }

// 文件转码（异步）
babel.transformFile('filename.js', options, function(err, result) {
  result; // => { code, map, ast }
});

// 文件转码（同步）
babel.transformFileSync('filename.js', options);
// => { code, map, ast }

// Babel AST转码
babel.transformFromAst(ast, code, options);
// => { code, map, ast }
```
 **babel-core演示**
 new built-ins, static methods, instance methods, and generator functions
 
### (二) 功能包
+ babel-types：用于检验、构建和改变AST树的节点
+ babel-template：辅助函数，用于从字符串形式的代码来构建AST树节点
+ babel-helpers：一系列预制的babel-template函数，用于提供给一些plugins使用
+ babel-code-frames：用于生成错误信息，打印出错误点源代码帧以及指出出错位置
+ babel-plugin-xxx：babel转译过程中使用到的插件，其中babel-plugin-transform-xxx是transform步骤使用的
+ babel-preset-xxx：transform阶段使用到的一系列的plugin
+ **babel-polyfill**：JS标准新增的原生对象和API的shim，实现上仅仅是core-js和regenerator-runtime两个包的封装
+ **babel-runtime**：功能类似babel-polyfill，一般用于library或plugin中，因为它不会污染全局作用域

###  babel-polyfill
>Babel includes a polyfill that includes a custom regenerator runtime and core-js.

  前面说过，Babel默认只转换新的JavaScript句法(syntax)，而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的静态方法（比如Object.assign,Array.from, Array.isArray,Array.of), 还有实例方法如Array.prototye.inclueds等，Babel都不会转码。 Babel默认不转码的API清单可以查看babel-plugin-transform-runtime模块的[definitions.js][definitions.js]文件。
  如果想让这些API和方法运行，必须使用babel-polyfill，为当前环境提供一个垫片。
  拿inclueds来说，IE11仍不支持该方法(去[caniuse](https://caniuse.com/#search=includes查看兼容性),在IE11浏览器的控制台里面运行下面的代码会报如下错误
  ```javascript
  var arr = [1,2,3]
  arr.includeds(1)
  // 对象不支持“includes”属性或方法
  ```
  我们先来看一下polyfill, 下面代码实现了一个includes方法的pollyfill
  ```javascript
 if(!Array.prototype.includeds) {
    Object.defineProperty(Array.prototype, 'includeds', {
      enumarable: false,
      writable: false,
      configurable: true,
      value: function(arg) {
        if(this == null) {
            throw new TypeError('"this" is null or not defined');
        }
        var len = this.length;
        if(!len) {
          return false
        } else {
            return this.indexOf(arg) > -1;
        }
      }
    })
  }
  ```
上面这段代码的意思是，如果目标环境中已经存在includeds, 什么都不做，如果没有就在 Array 的原型中定义一个,这便是polyfill 的意义。babel-polyfill 同理
虽说浏览器的特性对新javascript的语法规范支持状况千差万别，但其实可以提炼出两类：

  + 浏览器都有，只是不同语法的区别；
  + 有的浏览器有，有的浏览器没有。 

babel 编译过程处理第一种情况 - 统一语法的形态，通常是高版本语法编译成低版本的，比如 ES6 语法编译成 ES5 或 ES3。而 babel-polyfill 处理第二种情况 - 让目标浏览器支持所有特性，不管它是全局的，还是原型的，或是其它。这样，通过 babel-polyfill，不同浏览器在特性支持上就站到同一起跑线上。其实babel-polyfill就是一个针对ES2015+环境的shim，实现上来说babel-polyfill包只是简单的把core-js和regenerator runtime包装了下，这两个包才是真正的实现代码所在

+ 集成到webpack中的方法
1. 先安装包： npm install --save babel-polyfill, 因为polyfill会在源代码之前运行，所以需要安装成dependencies而不是devDependencies
2. 在所有代码运行之前增加 require('babel-polyfill')，有以下两种方式
   + 在程序入口文件app.js的顶部引用(因为polyfill代码需要在所有其他代码前先被调用): import "babel-polyfill"
   + 或者在webpack配置的entry里面第一个引入： module.exports = { entry: ["babel-polyfill", "./app/js"] };

但是，从上面很容易看出, babel-polyfill有两个主要缺点：

1. 使用 babel-polyfill 会导致打出来的包非常大，因为 babel-polyfill 是一个整体，把所有方法都加到原型链上。我们并不会用到所有的方法，这就造成了浪费
2. babel-polyfill 修改原型链，会污染全局变量，如果我们开发的也是一个类库供其他开发者使用，这种情况就会变得非常不可控。
因此在实际使用中,更好的选择是babel-runtime

### babel-runtime 和 babel-plugin-transform-runtime

> babel-runtime is a library that contain's Babel modular runtime helpers and a version of regenerator-runtime.

上面是官方定义，尽快我看得懂每一个单词，但是连起来

![什么鬼][what-hell]

我们去看看babel-runtime的包吧，package.json 里没有 main 字段，用法肯定不是 require('babel-runtime')和import那样, 我们时常在项目中看到 .babelrc 中使用 babel-plugin-transform-runtime，而 package.json 中的 dependencies (注意不是 devDependencies) 又包含了 babel-runtime，那这两个是不是成套使用的呢？他们又起什么作用呢？

babel 会转换 js 语法，之前已经提过了。以 Object.assign举例，IE不支持 Object.assign，此时，要想兼容的话，我们有两个方案
1. 引入babel-polyfill, 这个当然能解决问题，但是弊端很大，污染全局变量，代码庞大
2. 引入该语法插件plugin-transfrom-object-assign来现实特定的转换
使用第二种方法来编译下面的代码`code0`
```javascript
Object.assign({}, {a:1})
```
执行demo5的`npm run build1`(babel index.js --out-file compile1.js --plugins=transform-object-assign)生成compile1.js如下代码`code1`
```javascript
var _extends = Object.assign || function (target) { for (var i = 1; i < arguments.length; i++) { var source = arguments[i]; for (var key in source) { if (Object.prototype.hasOwnProperty.call(source, key)) { target[key] = source[key]; } } } return target; };

var object = _extends({}, { a: 1 });
```
没错，的确解决了兼容性的问题，但是新的问题来了，如果你的项目里有多少个文件用了 Object.assign，_extends 辅助函数会出现多少次，为了避免代码重复，我们必选要把这个方法分离出去, 改成引用的形式，babel-plugin-transform-runtime就是来做这些工作的，执行demo5的`npm run build2`(babel index.js --out-file compile2.js --plugins=transform-runtime)变可生成下面代码`code2`
```javascript
import _extends from "babel-runtime/helpers/extends";//从直接定义改为引用，这样就不会重复定义了。
_extends({}, {a:1});
```
从结果可以看出，定义方法改成了引用，那重复定义就变成了重复引用，就不存在代码重复的问题了。

上面的babel-runtime就是这些方法的集合处，也因此，在使用 babel-plugin-transform-runtime 的时候必须把 babel-runtime 当做依赖。
babel-runtime，它内部集成了
1. core-js: 转换一些内置类 (Promise, Symbols等等) 和静态方法 (Array.from 等)。绝大部分转换是这里做的。在代码中使用这些内置类和静态方法时自动引入。
2. regenerator: 作为 core-js 的拾遗补漏，主要是 generator/yield 和 async/await 两组的支持。当代码中有使用 generators/async 时自动引入。
3. helpers,如上面的 _extends 就是其中之一，其他还有如 jsx, classCallCheck 等等。在代码中有内置的 helpers 使用时(如上面的代码code1)移除定义，并插入引用(于是就变成了code2)。

**总结**：
1.  babel-polyfill 与 babel-runtime 的区别，前者改造目标浏览器，让你的浏览器拥有本来不支持的特性；后者改造你的代码，让你的代码能在所有目标浏览器上运行，但不改造浏览器, babel-polyfill比babel-runtime多了对包含高版本 js 中类型的实例方法 (例如 [1,2,3].includes(1))的支持。
2. babel-plugin-transform-runtime插件依赖babel-runtime，babel-runtime是真正提供runtime环境的包；也就是说transform-runtime插件是把js代码中使用到的新原生对象和静态方法转换成对runtime实现包的引用。

不懂的话让灵魂画手再帮你们一次

### (三) 工具包

### babel-cli
1 .  Babel提供babel-cli工具，用于命令行转码, 基本命令如下
```node	
// 转码结果输出到标准输出
$ npx babel example.js
// 转码结果写入一个文件
// --out-file 或 -o 参数指定输出文件
$ npx  babel example.js --out-file compiled.js
// 或者
$ npx  babel example.js -o compiled.js
// 整个目录转码
// --out-dir 或 -d 参数指定输出目录
$ npx  babel src --out-dir lib
// 或者
$ npx   babel src -d lib
// -s 参数生成source map文件
$npx   babel src -d lib -s
```	
**babel-cli使用演示**

### babel-node
babel-cli工具自带一个babel-node命令，提供一个支持ES6的REPL环境。它支持Node的REPL环境的所有功能，而且可以直接运行ES6代码。
它不用单独安装，而是随babel-cli一起安装。然后，执行babel-node就进入REPL环境。
打开cmd终端,输入`babel-node`进入PEPL环境,直接输入es6代码运行

### babel-register
  babel-register模块改写require命令，为它加上一个钩子。此后，每当使用require加载.js、.jsx、.es和.es6后缀名的文件，就会先用Babel进行转码。


### babel 7.x
### babel7的变化
[babel7](https://babeljs.io/blog/2018/08/27/7.0.0)
我们关注一下 7.0 的变化(核心都没有变化)
最后用一张脑图来总结这篇文章
![文章脑图](http://on-img.com/chart_image/57246562e4b0d554ee4ffffb.png)
----
[env-plugin-list]:https://github.com/babel/babel-preset-env/blob/master/data/plugin-features.js
[definitions.js]:https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js
[what-hell]:http://5b0988e595225.cdn.sohucs.com/images/20170713/2df27dfe14be40ce8da5710535db4043.png