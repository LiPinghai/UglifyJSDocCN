# UglifyJS中文文档
译序
-----------
本文档译自[UglifyJS2文档](https://github.com/mishoo/UglifyJS2)。

由于webpack本身集成了UglifyJS插件（webpack.optimize.UglifyJsPlugin），其命令`webpack -p`即表示调用UglifyJS来压缩代码，还有不少webpack插件如`html-webpack-plugin`也会默认使用UglifyJS。因此我们其实经常要用到它，但UglifyJS2本身配置较复杂/选项繁多，又没有中文文档，使用起来如坠云雾。鉴于此特翻译此文，谬误甚多，敬请斧正。转载请注明[出处](https://github.com/LiPinghai/UglifyJSDocCN)。

**喜欢的话请收藏、给个赞吧！谢谢！**

词典：
```
parse       解释
compress    压缩
mangle      混淆
beautify    美化
minify      最小化
CLI         命令行工具
sourcemap   编译后代码对源码的映射，用于网页调试
AST         抽象语法树
name        名字，包括变量名、函数名、属性名
toplevel    顶层作用域
unreachable 不可达代码
option      选项
STDIN       标准输入，指在命令行中直接输入
STDOUT      标准输出
STDERR      标准错误输出
side effects函数副作用，即函数除了返回外还产生别的作用，比如改了全局变量
```

以下为正文：

UglifyJS 2
==========

UglifyJs 是一个js 解释器、最小化器、压缩器、美化器工具集（parser, minifier, compressor or beautifier toolkit）。

这个网页是命令行使用的文档，要看API和内部文档请到[作者的网站](http://lisperator.net/uglifyjs/)。
另外还有个[在线demo](http://lisperator.net/uglifyjs/#demo)(FF、chrome，safari可能也行)

#### Note:
- `uglify-js`的发行版本只支持ES5，如果你要压缩ES6+代码请使用[兼容](#harmony)开发分支
-  Node7有个已知的性能倒退问题——运行`uglify-js`两次导致很慢

安装
-------

首先确认一直你已经安装了最新的[node.js](http://nodejs.org/)(装完后或许需要重启一下电脑)

用NPM安装CLI：
```
npm install uglify-js -g
```
用NPM下载给程序使用:
```
npm install uglify-js
```
用Git下载：
```
git clone git://github.com/mishoo/UglifyJS2.git
cd UglifyJS2
npm link 
```

使用
------
```
 uglifyjs [input files] [options]
```

UglifyJS2可以输入多文件。建议你先写输入文件，再传选项。UglifyJS会根据压缩选项，把文件放在队列中依次解释。所有文件都会在同一个全局域中，假如一个文件中的变量、方法被另一文件引用，UglifyJS会合理地匹配。

假如你不要输入文件，而是要输入字符串（STDIN），那就把文件名换成一个横线（-）

如果你想要把选项写在文件名的前面，那要在二者之前加上双横线，防止文件名被当成了选项：
```
 uglifyjs --compress --mangle -- input.js
```
以下是可用的选项：
```
  --source-map                  指定输出的文件产生一份sourcemap 
  --source-map-root             此路径中的源码编译后会产生sourcemap
  --source-map-url              放在//#sourceMappingURL的sourcemap路径.  默认是 
                                --source-map传入的值.
  --source-map-include-sources  如果你要在sourcemap中加上源文件的内容作为sourcesContent属性，
                                就传这个参数吧。
  --source-map-inline           把sourcemap以base64格式附在输出文件结尾
  --in-source-map               输入sourcemap。假如的你要编译的JS是另外的源码编译出来的。
                                假如该sourcemap包含在js内，请指定"inline"。
  --screw-ie8                   是否要支持IE6/7/8。UglifyJS默认不兼容IE。
  --support-ie8                 是否要支持IE6/7/8，等同于在`compress`, `mangle` 和
                                 `output`选项中都设置`screw_ie8: false`
  --expr                        编译一个表达式，而不是编译一段代码（编译JSON时用）
  -p, --prefix                  忽略sourcemap中源码的前缀。例如`-p 3`会干掉文件名前面3层目录
                                以及保证路径是相对路径。你也可以指定`-p relative`,让UglifyJS
                                自己计算输出文件、sourcemap与源码之间的相对路径。
  -o, --output                  输出文件，默认标准输出(STDOUT)
  -b, --beautify                美化输出/指定输出 选项
  -m, --mangle                  Mangle的名字，或传入一个mangler选项.
  -r, --reserved                mangle的例外，不包含在mangling的名字
  -c, --compress                是否启用压缩功能（true/fasle），或者传一个压缩选项对象, 例如 
                                `-c 'if_return=false,pure_funcs=["Math.pow","console.log"]'`，
                                `-c`不带参数的话就是用默认的压缩设置。
  -d, --define                  全局定义
  -e, --enclose                 所有代码嵌入到一个大方法中，传入参数为配置项
  --comments                    保留版权注释。默认保留Google Closure那样的，保留JSDoc-style、
                                包含"@license" 或"@preserve"字样的注释。你也可以传下面的参数:
                                - "all" 保留所有注释
                                - 正则（如`/foo/`、`/^!/`）保留匹配到的。要注意，如果启用了压
                                缩，因为会移除不可达代码以及压缩连续声明，因此不是*所有*注释都能
                                保留下来。
  --preamble                    在输出文件开头插入的前言。你可以插入一段注释，例如版权信息。
                                这些不会被编译，但sourcemap会改成当前的样子。
  --stats                       在STDERR中显示操作运行时间。
  --acorn                       用 Acorn解析。
  --spidermonkey                假如输入文件是 SpiderMonkey AST 格式(像JSON).
  --self                        把UglifyJS2本身也构建成一个依赖包
                                (等同于`--wrap=UglifyJS --export-all`)
  --wrap                        所有代码嵌入到一个大函数中,让"exports"和"global"变量有效，
                                你需要传入一个参数指定模块被浏览器引入时的名字。
  --export-all                  只当`--wrap`时有效,告诉UglifyJS自动把代码暴露到全局。
  --lint                        显示一些可视警告
  -v, --verbose                 Verbose
  -V, --version                 打印版本号.
  --noerr                       不要为-c,-b 或 -m选项中出现未知选项而抛出错误。
  --bare-returns                允许返回函数的外部。当最小化CommonJs模块和Userscripts时，
                                可能匿名函数会被.user.js引擎调用立即执行（IIFE）
  --keep-fnames                 不要混淆、干掉的函数的名字。当代码依赖Function.prototype.name时有用。
  --reserved-file               要保留的文件的名字
  --reserve-domprops            保留（绝大部分？）DOM的属性，当--mangle-props
  --mangle-props                混淆属性，默认是`0`.设置为`true`或`1`则会混淆所有属性名。
                                设为`unquoted`或 `2`则只混淆不在引号内的属性。`2`时也会让
                                `keep_quoted_props` 美化选项生效，保留括号内的属性；让压缩选项
                                的`properties`失效，阻止覆写带点号（.）的属性。你可以通过在命令
                                中明确设置来覆写它们。
  --mangle-regex                混淆正则，只混淆匹配到的属性名。
  --name-cache                  用来保存混淆map的文件
  --pure-funcs                  假如返回值没被调用则可以安全移除的函数。 
                                例如`--pure-funcs Math.floor console.info`(需要设置 `--compress`)
```

指定`--output` (`-o`)来明确输出文件，否则将在终端输出（STDOUT）


## sourcemap选项
## Source map options

UglifyJS2可以生成一份sourcemap文件，这对调试你压缩后的JS代码非常有用。传`--source-map output.js.map`（完整路径）来获取sorcemap文件。

另外，你可能要设置`--source-map-root`传入源码所在的根目录。为了防止出现整个路径，你可以用`--prefix` (`-p`)指定干掉几层soucemap中路径的前缀。

例如：
```
uglifyjs /home/doe/work/foo/src/js/file1.js \
          /home/doe/work/foo/src/js/file2.js \
          -o foo.min.js \
          --source-map foo.min.js.map \
          --source-map-root http://foo.com/src \
          -p 5 -c -m
```
上述配置会压缩和混淆`file1.js`、`file2.js`，输出文件`foo.min.js` 和sourcemap`foo.min.js.map`，sourcemap会建立`http://foo.com/src/js/file1.js`、
`http://foo.com/src/js/file2.js`的映射。（实际上，sourcemap根目录是`http://foo.com/src`，所以相当于源文件路径是`js/file1.js`、`js/file2.js`）

### 关联sourcemap

假如你的JS代码是用其他编译器（例如coffeescript）生成的，那么映射到JS代码就没什么用了，你肯定希望映射到CoffeeScript源码。UglifyJS有一个选项可以输入sourcemap，假如你有一个从CoffeeScript → 编译后JS的map的话，UglifyJS可以生成一个从CoffeeScript->压缩后JS的map映射到源码位置。

你可以传入 `--in-source-map /path/to/input/source.map`来尝试此特性，如果sourcemap包含在js内，则写`--in-source-map inline` 。通常输入的sourcemap会指向源代码生成的JS，所以你可以忽略不写输入文件。

## 混淆选项
## Mangler options

你需要传入`--mangle` (`-m`)来使启用混淆功能。支持用逗号隔开选项：

- `toplevel` — 混淆在最高作用域中声明的变量名（默认disabled）

- `eval` - 混淆在`eval` 或 `with`作用域出现的变量名（默认disabled）

当启用混淆功能时，如果你希望保留一些名字不被混淆，你可以用`--reserved` (`-r`) 声明一些名字，用逗号隔开。例如：
```
 uglifyjs ... -m -r '$,require,exports'
```
防止`require`, `exports`和 `$`被混淆改变。

### 混淆属性名 (`--mangle-props`)

**警告：**这能会搞崩你的代码。混淆属性名跟混淆变量名不一样，是相互独立的。传入`--mangle-props`会混淆对象所有可见的属性名。例如：
```js
var x = {
  foo: 1
};

x.bar = 2;
x["baz"] = 3;
x[condition ? "moo" : "boo"] = 4;
console.log(x.something());
```

上面代码中，`foo`, `bar`, `baz`, `moo` 、 `boo`会被替换成单字符名字，`something()`则不变。

为了合理地使用，我们应该避免混淆一些JS标准的名字。比如，如果你代码中有`x.length = 10`，那`length`就将被混淆，不管这是在对象中还是访问数组的长度，它都被干掉。为了避免这种情况，你可以用 `--reserved-file`来输入一个文件，里面包含不参与混淆的名字，变量名或属性名都行。就像下面这样：
```js
{
  "vars": [ "define", "require", ... ],
  "props": [ "length", "prototype", ... ]
}
```

`--reserved-file` 可以是文件名数组（用逗号隔开，你也可以传多个`--reserved-file`），在上面例子中的名字将被排除在混淆中。
 `tools/domprops.json` 里有一个默认的排除名单，包括绝大部分标准JS和多种浏览器中的DOM属性名。传入`--reserve-domprops` 可以读取此名单生效。

 你也可以用正则表达式来定义一些应该被混淆的属性名。例如`--mangle-regex="/^_/"`，会只混淆以下划线开始的属性名。

 当你压缩多个文件时，为了保证让它们最终能同时工作，我们要让他们中同样的属性名混淆成相同的结果。传入`--name-cache
filename.json`，UglifyJS会维护一个共同的映射供他们复用。这个json一开始应该是空的，例如：

```
rm -f /tmp/cache.json  # start fresh
uglifyjs file1.js file2.js --mangle-props --name-cache /tmp/cache.json -o part1.js
uglifyjs file3.js file4.js --mangle-props --name-cache /tmp/cache.json -o part2.js
```

现在，`part1.js` 和 `part2.js`会知晓对方混淆的属性名。

假如你把所有文件压缩成同一个文件，那就不需要启用名字缓存了。

#### 混淆括号中的名字(`--mangle-props=unquoted`或 `--mangle-props=2`)

使用括号属性名 (`o["foo"]`)以保留属性名(`foo`)。这会让整个脚本中其余此属性的引用(`o.foo`)也不被混淆。例如：

```
$ echo 'var o={"foo":1, bar:3}; o.foo += o.bar; console.log(o.foo);' | uglifyjs --mangle-props=2 -mc
var o={"foo":1,a:3};o.foo+=o.a,console.log(o.foo);
```
#### 调试属性名混淆

为了混淆属性时不至于完全糊涂，你可以传入`--mangle-props-debug`来调试。例如`o.foo`会被混淆成`o._$foo$_`。这让源码大量、属性被混淆时也可以debug，可以看清混淆会把哪些属性搞乱。

你可以用`--mangle-props-debug=XYZ`来传入自定义后缀。让`o.foo` 混淆成 `o._$foo$XYZ_`， 你可以在每次编译是都改变一下，来辨清属性名怎么被混淆的。一个小技巧，你可以每次编译时传随机数来模仿混淆操作（例如你更新了脚本，有了新的属性名），这有助于识别混淆时的出错。

## 压缩器选项
## Compressor options

你要传入 `--compress` (`-c`)来启用压缩功能。你可以用逗号隔开选项。选项的形式为`foo=bar`,或者就`foo`（后者等同于你要设为`true`，相当于`foo=true`的缩写）。

- `sequences`(默认true) -- 连续声明变量，用逗号隔开来。可以设置为正整数来指定连续声明的最大长度。如果设为`true` 表示默认`200`个，设为`false`或`0`则禁用。 `sequences`至少要是`2`,`1`的话等同于`true`（即`200`)。默认的sequences设置有极小几率会导致压缩很慢，所以推荐设置成`20`或以下。

- `properties` -- 用`.`来重写属性引用，例如`foo["bar"] → foo.bar`

- `dead_code` -- 移除没被引用的代码

- `drop_debugger` -- 移除 `debugger;` 

- `unsafe` (默认 false) -- 使用 "unsafe"转换 (下面详述)

- `unsafe_comps` (默认 false) -- 保留`<` 和 `<=`不被换成 `>` 和 `>=`。假如某些运算对象是用`get`或 `valueOf`object得出的时候，转换可能会不安全，可能会引起运算对象的改变。此选项只有当 `comparisons`和`unsafe_comps` 都设为true时才会启用。

- `unsafe_math` (默认 false) -- 优化数字表达式，例如`2 * x * 3` 变成 `6 * x`, 可能会导致不精确的浮点数结果。

- `unsafe_proto` (默认 false) -- 把`Array.prototype.slice.call(a)` 优化成 `[].slice.call(a)`

- `conditionals` -- 优化`if`等判断以及条件选择

- `comparisons` --  把结果必然的运算优化成二元运算，例如`!(a <= b) → a > b` (只有设置了 `unsafe_comps`时才生效)；尽量转成否运算。例如 `a = !b && !c && !d && !e → a=!(b||c||d||e)` 

- `evaluate` -- 尝试计算常量表达式

- `booleans` -- 优化布尔运算，例如 `!!a? b : c → a ? b : c`

- `loops` -- 当`do`、`while` 、 `for`循环的判断条件可以确定是，对其进行优化。 

- `unused` -- 干掉没有被引用的函数和变量。（除非设置`"keep_assign"`，否则变量的简单直接赋值也不算被引用。）

- `toplevel` -- 干掉顶层作用域中没有被引用的函数 (`"funcs"`)和/或变量(`"vars"`) (默认是`false` , `true` 的话即函数变量都干掉)

- `top_retain` -- 当设了`unused`时，保留顶层作用域中的某些函数变量。(可以写成数组，用逗号隔开，也可以用正则或函数. 参考`toplevel`)

- `hoist_funs` -- 提升函数声明

- `hoist_vars` (默认 false) -- 提升 `var` 声明 (默认是`false`,因为那会加大文件的size)

- `if_return` -- 优化 if/return 和 if/continue

- `join_vars` -- 合并连续 `var` 声明

- `cascade` -- 弱弱地优化一下连续声明, 将 `x, x` 转成 `x`，`x = something(), x` 转成 `x = something()`

- `collapse_vars` -- 当 `var` 和 `const` 单独使用时尽量合并

- `reduce_vars` -- 优化某些变量实际上是按常量值来赋值、使用的情况。

- `warnings` -- 当删除没有用处的代码时，显示警告

- `negate_iife` -- 当立即执行函数（IIFE）的返回值没用时，取消之。避免代码生成器会插入括号。

- `pure_getters` -- 默认是 `false`. 如果你传入`true`，UglifyJS会假设对象属性的引用（例如`foo.bar` 或 `foo["bar"]`）没有函数副作用。

- `pure_funcs` -- 默认 `null`. 你可以传入一个名字的数组，UglifyJS会假设这些函数没有函数副作用。**警告：**假如名字在作用域中重新定义，不会再次检测。例如`var q = Math.floor(a/b)`，假如变量`q`没有被引用，UglifyJS会干掉它，但 `Math.floor(a/b)`会被保留，没有人知道它是干嘛的。你可以设置`pure_funcs: [ 'Math.floor' ]` ，这样该函数会被认为没有函数副作用，这样整个声明会被废弃。在目前的执行情况下，会增加开销（压缩会变慢）。

- `drop_console` -- 默认 `false`.  传`true`的话会干掉`console.*`函数。如果你要干掉特定的函数比如`console.info` ，又想删掉后保留其参数中的副作用，那用`pure_funcs`来处理吧。

- `expression` -- 默认 `false`。传`true`来保留终端语句中没有"return"的完成值。例如在bookmarklets。

- `keep_fargs` -- 默认`true`。阻止压缩器干掉那些没有用到的函数参数。你需要它来保护某些依赖`Function.length`的函数。

- `keep_fnames` -- 默认 `false`。传 `true`来防止压缩器干掉函数名。对那些依赖`Function.prototype.name`的函数很有用。延展阅读：`keep_fnames` [混淆选项](#mangle).

- `passes` -- 默认 `1`。运行压缩的次数。在某些情况下，用一个大于1的数字参数可以进一步压缩代码大小。注意：数字越大压缩耗时越长。

- `keep_infinity` -- 默认 `false`。传`true`以防止压缩时把`1/0`转成`Infinity`，那可能会在chrome上有性能问题。

### `unsafe`选项

在某些刻意营造的案例中，启用某些转换**有可能**会打断代码的逻辑，但绝大部分情况下是安全的。你可能会想尝试一下，因为这毕竟会减少文件体积。以下是某些例子：

- `new Array(1, 2, 3)` 或 `Array(1, 2, 3)` → `[ 1, 2, 3 ]`
- `new Object()` → `{}`
- `String(exp)` 或 `exp.toString()` → `"" + exp`
- `new Object/RegExp/Function/Error/Array (...)` → 我们干掉用`new`的
- `typeof foo == "undefined"` → `foo === void 0`
- `void 0` → `undefined` (假如作用域中有一个变量名叫"undefined";我们这么做是因为变量名会被混淆成单字符）

### 编译条件语句

Uglify会假设全局变量都是常量(不管是否在局部域中定义了)，你可以用`--define` (`-d`)来实现定义全局变量。例如你传`--define DEBUG=false`，UglifyJS会在输出中干掉下面代码：
```javascript
if (DEBUG) {
console.log("debug stuff");
}
```

你可以像`--define env.DEBUG=false`这样写嵌套的常量。

在干掉那些永否的条件语句以及不可达代码时，UglifyJS会给出警告。现在没有选项可以禁用此特性，但你可以设置 `warnings=false` 来禁掉*所有*警告。

另一个定义全局常量的方法是，在一个独立的文档中定义，再引入到构建中。例如你有一个这样的`build/defines.js`：
```javascript
const DEBUG = false;
const PRODUCTION = true;
// 等等
```
构建使用这样写：
```
  uglifyjs build/defines.js js/foo.js js/bar.js... -c
```
UglifyJS会注意到这些常量。因为它们无法改变，所以它们会被认为是没被引用而被照样干掉。如果你用`const`声明，构建后还会被保留。如果你的运行环境低于ES6、不支持`const`，请用`var`声明加上`reduce_vars`设置（默认启用）来实现。

#### 编译条件语句API

你也可以通过程序API来设置编译配置。其中有差别的是一个压缩器属性`global_defs`：
```js
uglifyJS.minify([ "input.js"], {
  compress: {
      dead_code: true,
      global_defs: {
          DEBUG: false
      }
  }
});
```
## 美化器选项（Beautifier options）

代码生成器默认会输出尽量简短的代码。假如你想美化一下输出代码，请设置`--beautify` (`-b`)。你可以传入更多可选的选项参数来控制代码生成：

- `beautify` (默认 `true`) -- 是否美化输出代码。传`-b`的话就是设成true。假如你想生成最小化的代码同时又要用其他设置来美化代码，你可以设`-b beautify=false`。
- `indent-level` (默认 4) 缩进格数
- `indent-start` (默认 0) -- 每行前面加几个空格
- `quote-keys` (默认 `false`) -- 传`true`的话会在对象所有的键加上括号
- `space-colon` (默认 `true`) -- 在冒号后面加空格
- `ascii-only` (默认 `false`) -- 避免Unicode字符在字符串/正则中出现（非ascii字符会变不合法）。
- `inline-script` (默认 `false`) -- 避免字符串中出现`</script`中的斜杠
- `width` (默认 80) -- 仅在美化时生效，设定一个行宽让美化器尽量实现。这会影响行中文字的数量（不包括缩进）。当前本功能实现得不是非常好，但依然让美化后的代码可读性大大增强。
- `max-line-len` (默认 32000) -- 最大行宽（压缩后的代码）
- `bracketize` (默认 `false`) -- 永远在`if`, `for`,`do`, `while`, `with`后面加上大括号，即使循环体只有一句。
- `semicolons` (默认 `true`) -- 用分号分开多个声明。如果你传`false`,则总会另起一行，增强输出文件的可读性。（gzip前体积更小，gzip后稍大一点点）
- `preamble` (默认 `null`) -- 如果要传的话，必须是字符串。它会被加在输出文档的前面。sourcemap会随之调整。例如可以用来插入版权信息。
- `quote_style` (默认 `0`) -- 影响字符串的括号格式（也会影响属性名和指令）。
- `0` -- 倾向使用双引号，字符串里还有引号的话就是单引号。
- `1` -- 永远单引号
- `2` -- 永远双引号
- `3` -- 永远是本来的引号
- `keep_quoted_props` (默认 `false`) -- 如果启用，会保留属性名的引号。

### 保留版权告示和其他注释

你可以传入`--comments`让输出文件中保留某些注释。默认时会保留JSDoc-style的注释（包含"@preserve","@license" 或 "@cc_on"（为IE所编译））。你可以传入`--comments all`来保留全部注释，或者传一个合法的正则来保留那些匹配到的注释。例如`--comments '/foo|bar/'`会保留那些包含"foo" 或 "bar"的注释。 

注意，无论如何，总会有些注释在某些情况下会丢失。例如：
```javascript
function f() {
	/** @preserve Foo Bar */
	function g() {
	  // this function is never called
	}
	return something();
}
```
即使里面带有"@preserve"，注释依然会被丢弃。因为内部的函数`g`（注释所依附的抽象语法树节点）没有被引用、会被压缩器干掉。

书写版权信息（或其他需要在输出文件中保留的信息）的最安全位置是全局节点。

## 对SpiderMonkey的支持

UglifyJS2有自己的抽象语法树格式；为了某些[现实的原因](http://lisperator.net/blog/uglifyjs-why-not-switching-to-spidermonkey-ast/)
我们无法在内部轻易地改成使用SpiderMonkey抽象语法树(AST)。但UglifyJS现在有了一个可以输入SpiderMonkeyAST的转换器。
例如[Acorn][acorn] ，这是一个超级快的生成SpiderMonkey AST的解释器。它带有一个实用的迷你CLI，能解释一个文件、把AST转存为JSON并标准输出。可以这样用UglifyJS来压缩混淆：
```
    acorn file.js | uglifyjs --spidermonkey -m -c
```
 `--spidermonkey`选项能让UglifyJS知道输入文件并非JavaScript，而是SpiderMonkey AST生成的JSON代码。这事我们不用自己的解释器，只把AST转成我们内部AST。


### 使用 Acorn 来解释代码

更有趣的是，我们加了 `--acorn`选项来使用Acorn解释所有代码。如果你传入这个选项，UglifyJS会`require("acorn")`

Acorn确实非常快（650k代码原来要380ms，现在只需250ms），但转换Acorn产生的SpiderMonkey树会额外花费150ms。所以总共比UglifyJS自己的解释器还要多花一点时间。

### 使用 UglifyJS 转换 SpiderMonkey AST

现在你可以像使用其他中间工具一样使用UglifyJS将JS抽象语法树转换为SpiderMonkey格式。

例如：
```javascript
function uglify(ast, options, mangle) {
  // 把SpiderMonkey AST 转成中间格式
  var uAST = UglifyJS.AST_Node.from_mozilla_ast(ast);

  // 压缩
  uAST.figure_out_scope();
  uAST = UglifyJS.Compressor(options).compress(uAST);

  // 混淆 (可选)
  if (mangle) {
    uAST.figure_out_scope();
    uAST.compute_char_frequency();
    uAST.mangle_names();
  }

  // 转回 SpiderMonkey AST
  return uAST.to_mozilla_ast();
}
```

[原博文](http://rreverser.com/using-mozilla-ast-with-uglifyjs/)有更多细节。

API参考
---------
假如是通过NPM安装的，你可以这样在你的应用中加载UglifyJS：
```javascript
var UglifyJS = require("uglify-js");
```
它会输出很多模块，但我在此只介绍一下涉及解释、混淆和压缩的基础代码。按(1)
解释, (2) 压缩, (3) 混淆, (4) 生成输出代码的顺序。

### 简易使用模式

`minify`是一个顶级的、单独、包含所有步骤的方法。如果你不需要进一步自定义的话，你应该会喜欢使用它。

例子:
```javascript
var result = UglifyJS.minify("/path/to/file.js");
console.log(result.code); // 最小化输出
// 假如你不想传一个文件名，而是要传入一段代码
var result = UglifyJS.minify("var b = function () {};", {fromString: true});
```
你也可以压缩多个文件：
```javascript
var result = UglifyJS.minify([ "file1.js", "file2.js", "file3.js" ]);
console.log(result.code);
```
这样生成一份sourcemap：
```javascript
var result = UglifyJS.minify([ "file1.js", "file2.js", "file3.js" ], {
	outSourceMap: "out.js.map"
});
console.log(result.code); // 最小化输出
console.log(result.map);
```

你也可以用一个带fromString选项的对象来要生成sourcemap：
```javascript
var result = UglifyJS.minify({"file1.js": "var a = function () {};"}, {
  outSourceMap: "out.js.map",
  outFileName: "out.js",
  fromString: true
});
```
要注意，此时sourcemap并不会保存为一份文件，它只会返回在`result.map`中。`outSourceMap` 的值只用来在`result.code`中设置`//# sourceMappingURL=out.js.map` ，`outFileName` 的值只用来在sourcemap文件中设置 `file`属性。

sourcemap（查阅[规格][sm-spec]）中的`file`属性会优先使用 `outFileName` ，假如没有，会从`outSourceMap`中推导（就是去掉`'.map'`）。

你可以把`sourceMapInline`设为`true` ，这样sourcemap会加在代码末尾。

你也可以指定sourcemap中的源文件根目录（sourceRoot）属性：
```javascript
var result = UglifyJS.minify([ "file1.js", "file2.js", "file3.js" ], {
	outSourceMap: "out.js.map",
	sourceRoot: "http://example.com/src"
});
```

如果你要压缩*从其他文件编译得来的*带一份sourcemap的JS文件，你可以用`inSourceMap`参数：

```javascript
var result = UglifyJS.minify("compiled.js", {
	inSourceMap: "compiled.js.map",
	outSourceMap: "minified.js.map"
});
// 跟之前一样，返回 `code`和 `map`
```

如果你要输入的sourcemap并非一份单独文件，你可以在对象参数中设置`inSourceMap`参数：
```javascript
var result = UglifyJS.minify("compiled.js", {
	inSourceMap: JSON.parse(my_source_map_string),
	outSourceMap: "minified.js.map"
});
```
只有在需要`outSourceMap`时， `inSourceMap` 才会被用到（否则就没用咯）。

要设置sourcemap的url的话，请用 `sourceMapUrl`选项。

如果你要用` X-SourceMap `请求头，你可以把`sourceMapUrl`选项设为false。
`outSourceMap`的默认设置：

```javascript
var result = UglifyJS.minify([ "file1.js" ], {
  outSourceMap: "out.js.map",
  sourceMapUrl: "localhost/out.js.map"
});
```

其他选项：

- `warnings` (默认 `false`) — 传`true`来现实压缩器的警告

- `fromString` (默认 `false`) — 传`true`的话，你可以输入JS源码，而不是文件名。

- `mangle` (默认 `true`) — 传`false`来跳过混淆步骤，或者传一个对象来特定指明混淆选项（下面详述）。

- `mangleProperties` (默认 `false`) — 传一个对象来自定义指明混淆对象属性的选项。

- `output` (默认 `null`) — 如果你要进一步指定[输出选项][codegen],请传一个对象。默认是压缩到最优化。

- `compress` (默认 `{}`) — 传`false`的话就跳过整个压缩步骤。自定义的话请传一个[压缩选项][compressor]对象。

- `parse` (默认 {}) — 如果你要进一步自定义解释步骤请传一个[解释选项][parser]对象（不是所有选项都有效....下面再说）。

#### 混淆

- `except` - 传一个应该排除在混淆之外的标识的数组。

 - `toplevel` — 混淆那些定义在顶层作用域的名字（默认禁用）。

 - `eval` — 混淆那些在with或eval中出现的名字（默认禁用）。

 - `keep_fnames` -- 默认`false`。传`true`的话就不混淆函数名。对那些依赖`Function.prototype.name`的代码有用。延展阅读：`keep_fnames` [压缩选项](#compressor-options).


例子：
```javascript
  //tst.js
  var globalVar;
  function funcName(firstLongName, anotherLongName)
  {
    var myVariable = firstLongName +  anotherLongName;
  }

  UglifyJS.minify("tst.js").code;
  // 'function funcName(a,n){}var globalVar;'

  UglifyJS.minify("tst.js", { mangle: { except: ['firstLongName'] } }).code;
  // 'function funcName(firstLongName,a){}var globalVar;'

  UglifyJS.minify("tst.js", { mangle: { toplevel: true } }).code;
  // 'function n(n,a){}var a;'
  ```

#### 混淆属性名选项

 - `regex` — 传一个正则，*仅混淆*匹配到的名字。（与`--mangle-regex` CLI参数选项关联）
 - `ignore_quoted` – 只混淆*非括号中*的属性名（与`--mangle-props 2` CLI 参数选项关联）

 - `debug` – 让混淆后的名字与原名字有关。与`--mangle-props-debug` CLI 参数选项关联）。默认是`false`。传一个空字符串来启用，或者传一个非空字符串来添加后缀。

### 高级使用模式

如果`minify`函数太简单不能满足你的需求，下面这些API信息有更多的细节详情：

#### 解释器
```javascript
var toplevel_ast = UglifyJS.parse(code, options);
```

`options` 是可选的，要传的话就必须传个对象。下面这些是有效的属性：

- `strict` — 禁用自动添加分号，禁止数组、对象末尾还有逗号。
- `bare_returns` — 允许函数返回外部。（与 `--bare-returns` CLI参数选项关联，对`minify` `parse`选项对象也有效。）
- `filename` — 输入的文件名。
- `toplevel` — 一个 `toplevel` 节点。 (就是之前调用`parse`返回的)

后面两个选项是当你要最小化多个文件成一个文件（以及正确的sourcemap）时有用。我们的CLI会像这样处理：
```javascript
var toplevel = null;
files.forEach(function(file){
	var code = fs.readFileSync(file, "utf8");
	toplevel = UglifyJS.parse(code, {
		filename: file,
		toplevel: toplevel
	});
});
```

完成后，我们就在`toplevel`这个大AST里包含了我们的所有文件，每一份都带着正确的来源信息。

#### 作用域信息

UglifyJS包含一个作用域分析器，你可以在压缩、混淆前手动调用。基本上，它添加了AST中的节点在哪里被命名、被引用了多少次、是否全局的、是否在`eval` 或`with`中声明等等。我们将讨论除此之外的，那些在你对AST进行任何操作前必须知道的重要事项：
```javascript
toplevel.figure_out_scope()
```

#### 压缩

就如这样：
```javascript
var compressor = UglifyJS.Compressor(options);
var compressed_ast = compressor.compress(toplevel);
```

`options`可以不要。之前的“压缩器选项“中已经讲过可以填什么。默认选项对大多数脚本来说应该都是最佳的。

压缩器是破坏性的，所以不要依赖那些源树`toplevel`。

#### 混淆

压缩之后再调用一次`figure_out_scope`是个好做法（因为压缩过程可能会干掉一些没用的、不可达的代码，改变标识的数量和位置），你也可以选择在Gzip（统计不可混淆的词中字符的使用频率）后调用。例如：

```javascript
compressed_ast.figure_out_scope();
compressed_ast.compute_char_frequency();
compressed_ast.mangle_names();
```

#### 生成输出代码

AST节点带一个`print`方法，用来生成输出流。基本上，要生成代码你只要这么做：

```javascript
var stream = UglifyJS.OutputStream(options);
compressed_ast.print(stream);
var code = stream.toString(); // 这就是你最小化后的代码
```
又或者这样缩写：
```javascript
var code = compressed_ast.print_to_string(options);
```

通常情况下`options`是可选的。输出流可以接收一堆选项参数，绝大多数在”美化选项“中有阐述。我们所关心的是`source_map` 和 `comments`选项。

#### 在输出代码中保留注释

你需要传入`comments`选项来保留某些注释。你可以传正则表达式（以`/`包裹或正则对象）、布尔值或函数。也可以传字符串`all` 或 `some`，`some`等同于CLI中`--comments`不带任何参数。如果你传正则，只有匹配到的注释会被保留。注意，匹配的主体不包括 `//` 或 `/*`。如果你传函数，每遇到树中的注释都会调用一下，传入两个参数，一是注释所依附的节点，二是注释标识本身。

注释标识有如下属性：

- `type`: 单行注释是"comment1"，多行注释 "comment2"。
- `value`: 注释体本身。
- `pos` 和 `endpos`: 注释在源码中出现的起始位置/结束位置（从0开始索引）。
- `line` 和 `col`: 注释在源码中出现的行和列。
- `file` — 源码的文件名
- `nlb` — 在源码中，如果注释前有一空行或注释另起新一行的话是`true`

你的函数返回`true`的话就保留注释，其他返回值都代表false。

####  生成sourcemap

你需要在调用`print`时传`source_map`参数。`source_map`参数需要是`SourceMap`对象（在[source-map][source-map]库顶部有个小框框里说了）。

例子：
```javascript
var source_map = UglifyJS.SourceMap(source_map_options);
var stream = UglifyJS.OutputStream({
	...
	source_map: source_map
});
compressed_ast.print(stream);

var code = stream.toString();
var map = source_map.toString(); // 输出json格式sourcemap
```

`source_map_options`（可选）包含以下属性:

- `file`: 被输出的、sourcemap所映射的JS的文件名
- `root`:  `sourceRoot` 属性 (详看 [规格][sm-spec])
- `orig`:  "original source map"，方便你想让sourcemap映射到生成JS的源码上。此参数可以只是字符串或json，也可以是包含源码sourcemap的json对象。

  [acorn]: https://github.com/ternjs/acorn
  [source-map]: https://github.com/mozilla/source-map
  [sm-spec]: https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit
  [codegen]: http://lisperator.net/uglifyjs/codegen
  [compressor]: http://lisperator.net/uglifyjs/compress
  [parser]: http://lisperator.net/uglifyjs/parser

#### 兼容版
#### Harmony

如果你想使用能最小化ES6+的实验性质的[兼容](https://github.com/mishoo/UglifyJS2/commits/harmony)分支，请在你的`package.json` 文件中加上下面代码：
```
"uglify-js": "git+https://github.com/mishoo/UglifyJS2.git#harmony"
```
或者直接安装兼容实验版UglifyJS：
```
npm install --save-dev uglify-js@github:mishoo/UglifyJS2#harmony
```
更多细节请看 [#448](https://github.com/mishoo/UglifyJS2/issues/448) 
