#babel那些事
### 一 babel到底是做什么的
> Babel is a JavaScript compiler
> 简单来说就是把 JavaScript 中 es2015+ 的新语法转化为 es5，让低端运行环境 (如浏览器和 node ) 能够认识并执行

### 二 Babel的工作原理
>babel是一个编译器compiler，但是叫转译器transpiler更准确，因为它只是把同种语言的高版本规则翻译成低版本规则，而不像编译器那样，输出的是另一种更低级的语言代码
>Babel的编译过程跟绝大多数其他语言的编译器大致同理，分为三个阶段：

1. 解析(parse)：使用babylon将代码字符串解析成抽象语法树ast 
2. 变换(transform)：使用babel-traverse, 利用配置好的 plugins/presets遍历ast，更新节点，转变为新的ast
3. 再建(generate)：使用babel-generator根据变换后的抽象语法树再生成代码字符串 

babel编译流程图
![编译流程图](http://on-img.com/chart_image/5b91e61ce4b06fc64ae8f146.png)

我们可以来看看下面这段代码
```
function abs(number) {
    if (number >= 0) {  
        return number;  
    } else {number; 
    }
}
```
[AST Explorer](https://astexplorer.net/) 可以帮助我们直观的认识AST的结构，

为了更加直观，可以用结构图表示如下
![!ast树](http://on-img.com/chart_image/5b932c6ee4b0534c9bd10d2a.png)
可以看到AST就是由一个个节点构成，每个节点都是源代码语法的一个标签。

解析这一步是如何生成抽象语法树的, 本人真心无能为力了, 给大家甩一个知乎上的一篇文章[Babel是如何读懂JS代码的](https://zhuanlan.zhihu.com/p/27289600)里面有讲解析器解析的过程,  涉及到编译器的部分，有兴趣的话，可以研究一下github上一个轻量级的编译器实现[the-super-tiny-compiler](https://github.com/jamiebuilds/the-super-tiny-compiler)

**此外,以下两点请注意**：
1.  babel本身不具备转码能力，转码都是通过插件完成的，如果不添加插件，那么经过babel后的代码和源代码是一样的
2.  babel只是转译新标准引入的语法，比如ES6的箭头函数转译成ES5的函数；而新标准引入的新的原生对象(Symbol)，部分原生对象新增的原型方法（数组的include的方法），新增的API等（如Proxy、Set等），这些babel是不会转译的。需要用户自行引入polyfill(后面会介绍)来解决,

### 三 使用方法
>Babel的配置文件是.babelrc，存放在项目的根目录下。使用Babel的第一步，就是配置这个文件。
```
{
  "presets": [],
  "plugins": []
}
```

插件和 preset 的配置项
简略情况下，插件和 preset 只要列出字符串格式的名字即可。但如果某个 preset 或者插件需要一些配置项(或者说参数)，就需要把自己先变成数组。第一个元素依然是字符串，表示自己的名字；第二个元素是一个对象，即配置对象
```
{"presets": [
    // 带了配置项，自己变成数组
    [
        // 第一个元素依然是名字
        "env",
        // 第二个元素是对象，列出配置项
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

## presets
> babel 本身不具有任何转化功能，它把转化的功能都分解到一个个 plugin 里面, 插件是在转换这一阶段起作用的。当我们不配置任何插件时，经过 babel 的代码和输入是相同的。
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
    + env (重点,后面讲)
    + react
    + flow
    + minify
- stage-x
    + Stage 0 - 稻草人: 只是一个想法，经过 TC39 成员提出即可。
    + Stage 1 - 提案: 初步尝试。
    + Stage 2 - 初稿: 完成初步规范。
    + Stage 3 - 候选: 完成规范和浏览器初步实现。
    + Stage 4 - 完成: 将被添加到下一年度发布。
- es201x, latest
    但因为 env 的出现，使得 es2016 和 es2017 都已经废弃。latest 是 env 的雏形，它是一个每年更新的 preset，目的是包含所有 es201x。但也是因为更加灵活的 env 的出现，已经废弃。
### env (重点)
 >env 的核心目的是通过配置得知目标环境的特点，然后只做必要的转换。例如目标浏览器支持 es2015，那么 es2015 这preset 其实是不需要的，于是代码就可以小一点(一般转化后的代码总是更长)，构建时间也可以缩短一些。
如果不写任何配置项，env 等价于 latest，也等价于 es2015 + es2016 + es2017 三个相加(不包含 stage-x 中的插件)。env 包含的插件列表维护在[这里](https://github.com/babel/babel-preset-env/blob/master/data/plugin-features.js)

### 四 工具
2. 构建工具的插件 (webpack 的 babel-loader, rollup 的 rollup-plugin-babel)
Babel的配置文件是.babelrc，存放在项目的根目录下
```
{
  "presets": [
    ["env", { "modules": false }],
    "stage-2"
  ],
  "plugins": ["transform-runtime"],
  "env": {
    "test": {
      "presets": ["env", "stage-2"],
      "plugins": [ "istanbul" ]
    }
  }
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
### 五 Babel包介绍
[babel包](https://github.com/babel/babel/tree/master/packages)
[在线运行环境](https://babeljs.io/en/repl#?babili=false&browsers=&build=&builtIns=false&spec=false&loose=false&code_lz=DYUwLgBAhhC8EAoEEo4D4IA8BUAmZC-AUEUA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=&prettier=false&targets=&version=6.26.0&envVersion=)
#### (一) 核心包
+ babel-core：babel转译器本身，提供了babel的转译API，如babel.transform等，用于对代码进行转译。像webpack 的babel-loader就是调用这些API来完成转译过程的。
+ babylon：js的词法解析器
+ babel-traverse：用于对AST（抽象语法树，想了解的请自行查询编译原理）的遍历，主要给plugin用
+ babel-generator：根据AST生成代码

babel-core的使用
```
var babel = require('babel-core');

// 字符串转码, transform方法的第一个参数是一个字符串，表示需要转换的代码，第二个参数是转换的配置对象
babel.transform('code();', options);
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
 
### (二) 功能包

#### 1. babel-loader

>前面已经介绍过了 babel-cli。但一些大型的项目都会有构建工具 (如 webpack 或 rollup) 来进行代码构建和压缩 (uglify)。理论上来说，我们也可以对压缩后的代码进行 babel 处理，但那会非常慢。我们需要在 uglify 之前就加入 babel 处理，所以就有了 babel 插入到构建工具内部这样的需求。以我们项目中的 webpack 为例，webpack 有 loader 的概念，因此就出现了 babel-loader。和 babel-cli 一样，babel-loader 也会读取 .babelrc 或者 package.json 中的 babel字段作为自己的配置，之后的内核处理也是相同。唯一比 babel-cli 复杂的是，它需要和 webpack 交互，因此需要在 webpack 这边进行配置。比较常见的如下：
```
module: {
  rules: [
    {
      test: /\.js$/,
      exclude: /(node_modules|bower_components)/,
      loader: 'babel-loader'
    }
  ]
}
```
如果想在这里传入 babel 的配置项，也可以把改成：

// loader: 'babel-loader' 改成如下：
rules: [{
  test: /\.js$/,
  loader: 'babel-loader',
  options: {
    // 配置项在这里
  }
}'
+ babel-types：用于检验、构建和改变AST树的节点
+ babel-template：辅助函数，用于从字符串形式的代码来构建AST树节点
+ babel-helpers：一系列预制的babel-template函数，用于提供给一些plugins使用
+ babel-code-frames：用于生成错误信息，打印出错误点源代码帧以及指出出错位置
+ babel-plugin-xxx：babel转译过程中使用到的插件，其中babel-plugin-transform-xxx是transform步骤使用的
+ babel-preset-xxx：transform阶段使用到的一系列的plugin
+ babel-polyfill：JS标准新增的原生对象和API的shim，实现上仅仅是core-js和regenerator-runtime两个包的封装
+ babel-runtime：功能类似babel-polyfill，一般用于library或plugin中，因为它不会污染全局作用域

### 2 babel-polyfill
  >Babel默认只转换新的JavaScript句法（syntax），而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如Object.assign）都不会转码。
  比如,ES6在Array对象上新增了Array.from方法。Babel就不会转码这个方法。如果想让这个方法运行，必须使用babel-polyfill，为当前环境提供一个垫片。
  Babel默认不转码的API非常多，详细清单可以查看babel-plugin-transform-runtime模块的[definitions.js](https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js)文件。

  ##演示##
  ```
  var  a = Object.assign({},{b:1})
  ```
  
 babel-polyfill是一个针对ES2015+环境的shim，实现上来说babel-polyfill包只是简单的把core-js和regenerator runtime包装了下，这两个包才是真正的实现代码所在
 使用时，在所有代码运行之前增加 require('babel-polyfill')。或者更常规的操作是在 webpack.config.js 中将 babel-polyfill 作为第一个 entry

+ 使用方法
1.  先安装包： npm install --save babel-polyfill
要确保在入口处导入polyfill，因为polyfill代码需要在所有其他代码前先被调用
2. 代码方式： import "babel-polyfill"
  webpack配置： module.exports = { entry: ["babel-polyfill", "./app/js"] };

+ babel-polyfill 主要有两个缺点：

1. 使用 babel-polyfill 会导致打出来的包非常大，因为 babel-polyfill 是一个整体，把所有方法都加到原型链上。比如我们只使用了 Array.from，但它把 Object.defineProperty 也给加上了，这就是一种浪费了。这个问题可以通过单独使用 core-js 的某个类库来解决，core-js 都是分开的。
2. babel-polyfill 会污染全局变量，给很多类的原型链上都作了修改，如果我们开发的也是一个类库供其他开发者使用，这种情况就会变得非常不可控。
因此在实际使用中，通常我们会倾向于使用 babel-plugin-transform-runtime。
### 3 babel-runtime 和 babel-plugin-transform-runtime (重点)
babel-plugin-transform-runtime插件依赖babel-runtime，babel-runtime是真正提供runtime环境的包；也就是说transform-runtime插件是把js代码中使用到的新原生对象和静态方法转换成对runtime实现包的引用，举个例子如下：

作者：mercurygear
链接：https://www.jianshu.com/p/e9b94b2d52e2
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。


```
// 输入的ES6代码
var sym = Symbol();
// 通过transform-runtime转换后的ES5+runtime代码 
var _symbol = require("babel-runtime/core-js/symbol");
var sym = (0, _symbol.default)();

作者：mercurygear
链接：https://www.jianshu.com/p/e9b94b2d52e2
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
```

### (三) 工具包

### 1 babel-cli

1 .  Babel提供babel-cli工具，用于命令行转码, 基本命令如下
```	
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
>babel-node
>babel-cli工具自带一个babel-node命令，提供一个支持ES6的REPL环境。它支持Node的REPL环境的所有功能，而且可以直接运行ES6代码。
它不用单独安装，而是随babel-cli一起安装。然后，执行babel-node就进入REPL环境。
打开cmd终端,输入`babel-node`进入PEPL环境,直接输入es6代码运行

### 2 babel-register
  babel-register模块改写require命令，为它加上一个钩子。此后，每当使用require加载.js、.jsx、.es和.es6后缀名的文件，就会先用Babel进行转码。


## babel 7.x
###babel7的变化
[babel7](https://babeljs.io/blog/2018/08/27/7.0.0)
我们关注一下 7.0 的变化(核心都没有变化)

		