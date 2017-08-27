UglifyJS中文文档
----------------

本文档译自[UglifyJS3文档](https://github.com/mishoo/UglifyJS2)。

此前翻译的[UglifyJS2中文文档](https://github.com/LiPinghai/UglifyJSDocCN/tree/UglifyJs2)已挪到本项目UglifyJS2分支。

**喜欢的话请收藏、给个赞/star吧！谢谢！**

转载请注明原文链接（https://github.com/LiPinghai/UglifyJSDocCN/blob/master/README.md ）与作者信息。

## 译序

由于webpack本身集成了UglifyJS插件（webpack.optimize.UglifyJsPlugin），其命令`webpack -p`即表示调用UglifyJS来压缩代码，还有不少webpack插件如`html-webpack-plugin`也会默认使用UglifyJS。因此我们其实经常要用到它，但UglifyJS本身配置较复杂/选项繁多，又没有中文文档，使用起来如坠云雾。鉴于此特翻译此文，谬误甚多，敬请斧正。

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
option      选项/配置
STDIN       标准输入，指在命令行中直接输入
STDOUT      标准输出
STDERR      标准错误输出
side effects函数副作用，即函数除了返回外还产生别的作用，比如改了全局变量
shebang     释伴（#!）
```

以下为正文：

UglifyJS 3
==========

UglifyJS 是一个js 解释器、最小化器、压缩器、美化器工具集（parser, minifier, compressor or beautifier toolkit）。

#### 注意:
- **`uglify-js@3` 的[API](#api-reference) 和 [CLI](#command-line-usage)已简化，不再向后兼容 [`uglify-js@2`](https://github.com/mishoo/UglifyJS2/tree/v2.x)**.
- **UglifyJS `2.x` 文档在[这里](https://github.com/mishoo/UglifyJS2/tree/v2.x)**.
- `uglify-js` 只支持 ECMAScript 5 (ES5).
- 假如希望压缩 ES2015+ (ES6+)代码，应该使用 [**uglify-es**](https://github.com/mishoo/UglifyJS2/tree/harmony)这个`npm` 包。 

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


# CLI使用
# Command line usage

```
 uglifyjs [input files] [options]
```

UglifyJS可以输入多文件。建议你先写输入文件，再传选项。UglifyJS会根据压缩选项，把文件放在队列中依次解释。所有文件都会在同一个全局域中，假如一个文件中的变量、方法被另一文件引用，UglifyJS会合理地匹配。

假如没有指定文件，UglifyJS会读取输入字符串（STDIN）。

如果你想要把选项写在文件名的前面，那要在二者之前加上双横线，防止文件名被当成了选项：
```
 uglifyjs --compress --mangle -- input.js
```

### CLI选项：
### Command line options

```
  -h, --help                  列出使用指南。
                              `--help options` 获取可用选项的详情。
  -V, --version               打印版本号。
  -p, --parse <options>       指定解析器配置选项:
                              `acorn`  使用 Acorn 来解析。
                              `bare_returns`  允许在函数外return。
                                              在压缩CommonJS模块或`.user.js `引擎调用被同步执行函数包裹的用户脚本 时会用到。
                              `expression`  不是解析文件，二是解析一段表达式 (例如解析JSON).
                              `spidermonkey`  输入文件是 SpiderMonkey
                                              AST 格式 (JSON).
  -c, --compress [options]    启用压缩（true/false）/指定压缩配置:
                              `pure_funcs`  传一个函数名的列表，当这些函数返回值没被利用时，该函数会被安全移除。
  -m, --mangle [options]       启用混淆（true/false）/指定混淆配置:
                              `reserved`  不被混淆的名字列表。
  --mangle-props [options]    混淆属性/指定压缩配置:
                              `builtins`  混淆那些与标准JS全局变量重复的名字。
                              `debug`  添加debug前缀和后缀。
                              `domprops`  混淆那些鱼DOM属性名重复的名字。
                              `keep_quoted`  只混淆没括起来的属性名。
                              
                              `regex`  只混淆匹配（该正则）的名字。
                              `reserved`  不需要混淆的名字的列表（即保留）。
  -b, --beautify [options]    是否美化输出（true/false）/指定输出配置：
                              `beautify`  默认是启用.
                              `preamble`  预设的输出文件头部。你可以插入一段注释，比如版权信息。它不会被解析，但sourcemap会因此调整。
                              `quote_style`  括号类型:
                                              0 - auto自动
                                              1 - single单引号
                                              2 - double双引号
                                              3 - original跟随原码
                              `wrap_iife`  把立即执行函数括起来。注意：你或许应禁用压缩配置中的`negate_iife`选项。 

 -o, --output <file>         输出文件路径 (默认 STDOUT). 指定 `ast` 或
                                `spidermonkey`的话分别是输出UglifyJS或SpiderMonkey AST。
    --comments [filter]         保留版权注释。默认像Google Closure那样，保留包含"@license"或"@preserve"这样JSDoc风格的注释。你可以传以下的参数：
                                - "all" 保留全部注释
                                - 一个合适的正则，如 `/foo/` 或 `/^!/`，保留匹配到的注释。 
                                注意，在启用压缩时，因为死代码被移除或压缩声明为一行，并非*所有*的注释都会被保留。
    --config-file <file>        从此JSON文件读取 `minify()` 配置。
    -d, --define <expr>[=value] 定义全局变量。
    --ie8                       支持IE8。
                                等同于在`minify()`的`compress`、 `mangle` 和 `output`配置设置`ie8: true`。UglifyJS不会默认兼容IE8。
    --keep-fnames               不要混淆、干掉的函数的名字。当代码依赖Function.prototype.name时有用。
    --name-cache <file>         用来保存混淆map的文件。
    --self                      把UglifyJS本身也构建成一个依赖包
                                (等同于`--wrap UglifyJS`)
    --source-map [options]      启用 source map（true/false）/指定sourcemap配置:
                                `base` 根路径，用于计算输入文件的相对路径。
                                `content`  输入sourcemap。假如的你要编译的JS是另外的源码编译出来的。
                                假如该sourcemap包含在js内，请指定"inline"。 
                                `filename`  输出文件的名字或位置。
                                `includeSources`  如果你要在sourcemap中加上源文件的内容作sourcesContent属性，就传这个参数吧。
                                `root`  此路径中的源码编译后会产生sourcemap.
                                `url`   如果指定此值，会添加sourcemap相对路径在`//#sourceMappingURL`中。
    --timings                   在STDERR显示操作运行时间。
    --toplevel                  压缩/混淆在最高作用域中声明的变量名。
    --verbose                   打印诊断信息。
    --warn                      打印警告信息。
    --wrap <name>               把所有代码包裹在一个大函数中。让“exports”和“global”变量有效。
                                你需要传一个参数来指定此模块的名字，以便浏览器引用。         

```

指定`--output` (`-o`)来明确输出文件，否则将在终端输出（STDOUT）


## CLI sourcemap选项
## CLI source map options

UglifyJS可以生成一份sourcemap文件，这非常有利于你调试压缩后的JS代码。传`--source-map --output output.js`来获取sorcemap文件（sorcemap会生成为`output.js.map`）。

额外选项：
- `--source-map filename=<NAME>` 指定sourcemap名字。
- `--source-map root=<URL>` 传一个源文件的路径。否则UglifyJS将假定已经用了HTTP`X-SourceMap`，并将省略`//＃sourceMappingURL=`指示。
- `--source-map url=<URL>` 指定生成sourcemap的路径。

例如：
```
    uglifyjs js/file1.js js/file2.js \
             -o foo.min.js -c -m \
             --source-map root="http://foo.com/src",url=foo.min.js.map
```
上述配置会压缩和混淆`file1.js`、`file2.js`，输出文件`foo.min.js` 和sourcemap`foo.min.js.map`，sourcemap会建立`http://foo.com/src/js/file1.js`、
`http://foo.com/src/js/file2.js`的映射。（实际上，sourcemap根目录是`http://foo.com/src`，所以相当于源文件路径是`js/file1.js`、`js/file2.js`）

### 关联sourcemap
### Composed source map

假如你的JS代码是用其他编译器（例如coffeescript）生成的，那么映射到JS代码就没什么用了，你肯定希望映射到CoffeeScript源码。UglifyJS有一个选项可以输入sourcemap，假如你有一个从CoffeeScript → 编译后JS的map的话，UglifyJS可以生成一个从CoffeeScript->压缩后JS的map映射到源码位置。

你可以传入 `--source-map content="/path/to/input/source.map"`或来尝试此特性，如果sourcemap包含在js内，则写 `--source-map content=inline`。

## CLI混淆选项
## CLI mangle options

你需要传入`--mangle` (`-m`)来使启用混淆功能。支持以下选项（用逗号隔开）：

- `toplevel` — 混淆在最高作用域中声明的变量名（默认disabled）

- `eval` - 混淆在`eval` 或 `with`作用域出现的变量名（默认disabled）

当启用混淆功能时，如果你希望保留一些名字不被混淆，你可以用`--mangle reserved` 声明一些名字（用逗号隔开）。例如：
```
 uglifyjs ... -m reserved=[$,require,exports]'
```
这样能防止`require`, `exports`和 `$`被混淆改变。

### CLI混淆属性名 (`--mangle-props`)
### CLI mangling property names (`--mangle-props`)

**警告：**这能会搞崩你的代码。混淆属性名跟混淆变量名不一样，是相互独立的。传入`--mangle-props`会混淆对象所有可见的属性名，除了DOM属性名和JS内置的类名。例如：
```js
// example.js
var x = {
    baz_: 0,
    foo_: 1,
    calc: function() {
        return this.foo_ + this.baz_;
    }
};
x.bar_ = 2;
x["baz_"] = 3;
console.log(x.calc());
```

混淆所有属性（除了JS内置的）:
```bash
$ uglifyjs example.js -c -m --mangle-props
```
```javascript
var x={o:0,_:1,l:function(){return this._+this.o}};x.t=2,x.o=3,console.log(x.l());
```

混淆除了 `reserved` （保留）外的所有属性:
```bash
$ uglifyjs example.js -c -m --mangle-props reserved=[foo_,bar_]
```
```javascript
var x={o:0,foo_:1,_:function(){return this.foo_+this.o}};x.bar_=2,x.o=3,console.log(x._());
```

混淆匹配`regex`（正则）的属性：
```bash
$ uglifyjs example.js -c -m --mangle-props regex=/_$/
```
```javascript
var x={o:0,_:1,calc:function(){return this._+this.o}};x.l=2,x.o=3,console.log(x.calc());
```

混用多个混淆属性选项：
```bash
$ uglifyjs example.js -c -m --mangle-props regex=/_$/,reserved=[bar_]
```
```javascript
var x={o:0,_:1,calc:function(){return this._+this.o}};x.bar_=2,x.o=3,console.log(x.calc());
```
为了混淆正常使用，我们默认避免混淆标准JS内置的名字（`--mangle-props builtins`可以强制混淆）。

 `tools/domprops.json` 里有一个默认的排除名单，包括绝大部分标准JS和多种浏览器中的DOM属性名。传入`--mangle-props domprops` 可以让此名单失效。

 可以用正则表达式来定义该混淆的属性名。例如`--mangle-props regex=/^_/`，只混淆下划线开头的属性。

 当你压缩多个文件时，为了保证让它们最终能同时工作，我们要让他们中同样的属性名混淆成相同的结果。传入`--name-cache
filename.json`，UglifyJS会维护一个共同的映射供他们复用。这个json一开始应该是空的，例如：

```bash
$ rm -f /tmp/cache.json  # start fresh
$ uglifyjs file1.js file2.js --mangle-props --name-cache /tmp/cache.json -o part1.js
$ uglifyjs file3.js file4.js --mangle-props --name-cache /tmp/cache.json -o part2.js
```

这样`part1.js` 和 `part2.js`会知晓对方混淆的属性名。

假如你把所有文件压缩成同一个文件，那就不需要启用名字缓存了。

#### 混淆没括起来的名字(`--mangle-props keep_quoted`)
### Mangling unquoted names (`--mangle-props keep_quoted`)


使用括号属性名 (`o["foo"]`)以保留属性名(`foo`)。这会让整个脚本中其余此属性的引用(`o.foo`)也不被混淆。例如：

```javascript
// stuff.js
var o = {
    "foo": 1,
    bar: 3
};
o.foo += o.bar;
console.log(o.foo);
```
```bash
$ uglifyjs stuff.js --mangle-props keep_quoted -c -m
```
```javascript
var o={foo:1,o:3};o.foo+=o.o,console.log(o.foo);
```

#### 调试属性名混淆
### Debugging property name mangling 

为了混淆属性时不至于完全分不清，你可以传入`--mangle-props debug`来调试。例如`o.foo`会被混淆成`o._$foo$_`。这让源码量大、属性被混淆时也可以debug，可以看清混淆会把哪些属性搞乱。

```bash
$ uglifyjs stuff.js --mangle-props debug -c -m
```
```javascript
var o={_$foo$_:1,_$bar$_:3};o._$foo$_+=o._$bar$_,console.log(o._$foo$_);
```

你可以用`--mangle-props-debug=XYZ`来传入自定义后缀。让`o.foo` 混淆成 `o._$foo$XYZ_`， 你可以在每次编译是都改变一下，来辨清属性名怎么被混淆的。一个小技巧，你可以每次编译时传随机数来模仿混淆操作（例如你更新了脚本，有了新的属性名），这有助于识别混淆时的出错。

# API参考
# API Reference

假如是通过NPM安装的，你可以在你的应用中这样加载UglifyJS：

```javascript
var UglifyJS = require("uglify-js");
```
这输出一个高级函数**`minify(code, options)`**，它能根据配置，实现多种最小化（即压缩、混淆等）。 `minify()`默认启用压缩和混淆选项。例子：
```javascript
var code = "function add(first, second) { return first + second; }";
var result = UglifyJS.minify(code);
console.log(result.error); // runtime error, or `undefined` if no error
console.log(result.code);  // minified output: function add(n,d){return n+d}
```
你可以通过一个对象（key为文件名，value为代码）来同时`最小化`多个文件：
```javascript
var code = {
    "file1.js": "function add(first, second) { return first + second; }",
    "file2.js": "console.log(add(1 + 2, 3 + 4));"
};
var result = UglifyJS.minify(code);
console.log(result.code);
// function add(d,n){return d+n}console.log(add(3,7));
```

`toplevel`选项例子：

```javascript
var code = {
    "file1.js": "function add(first, second) { return first + second; }",
    "file2.js": "console.log(add(1 + 2, 3 + 4));"
};
var options = { toplevel: true };
var result = UglifyJS.minify(code, options);
console.log(result.code);
// console.log(3+7);
```

`nameCache` 选项例子:
```javascript
var options = {
    mangle: {
        toplevel: true,
    },
    nameCache: {}
};
var result1 = UglifyJS.minify({
    "file1.js": "function add(first, second) { return first + second; }"
}, options);
var result2 = UglifyJS.minify({
    "file2.js": "console.log(add(1 + 2, 3 + 4));"
}, options);
console.log(result1.code);
// function n(n,r){return n+r}
console.log(result2.code);
// console.log(n(3,7));
```

你可以像下面这样把名字缓存保存在文件中：

```javascript
var cacheFileName = "/tmp/cache.json";
var options = {
    mangle: {
        properties: true,
    },
    nameCache: JSON.parse(fs.readFileSync(cacheFileName, "utf8"))
};
fs.writeFileSync("part1.js", UglifyJS.minify({
    "file1.js": fs.readFileSync("file1.js", "utf8"),
    "file2.js": fs.readFileSync("file2.js", "utf8")
}, options).code, "utf8");
fs.writeFileSync("part2.js", UglifyJS.minify({
    "file3.js": fs.readFileSync("file3.js", "utf8"),
    "file4.js": fs.readFileSync("file4.js", "utf8")
}, options).code, "utf8");
fs.writeFileSync(cacheFileName, JSON.stringify(options.nameCache), "utf8");
```

综合使用多种`minify()`选项的例子：
```javascript
var code = {
    "file1.js": "function add(first, second) { return first + second; }",
    "file2.js": "console.log(add(1 + 2, 3 + 4));"
};
var options = {
    toplevel: true,
    compress: {
        global_defs: {
            "@console.log": "alert"
        },
        passes: 2
    },
    output: {
        beautify: false,
        preamble: "/* uglified */"
    }
};
var result = UglifyJS.minify(code, options);
console.log(result.code);
// /* uglified */
// alert(10);"
```

生成警告提示：
```javascript
var code = "function f(){ var u; return 2 + 3; }";
var options = { warnings: true };
var result = UglifyJS.minify(code, options);
console.log(result.error);    // runtime error, `undefined` in this case
console.log(result.warnings); // [ 'Dropping unused variable u [0:1,18]' ]
console.log(result.code);     // function f(){return 5}
```

生成错误提示：
```javascript
var result = UglifyJS.minify({"foo.js" : "if (0) else console.log(1);"});
console.log(JSON.stringify(result.error));
// {"message":"Unexpected token: keyword (else)","filename":"foo.js","line":1,"col":7,"pos":7}
```
Note: unlike `uglify-js@2.x`, the `3.x` API does not throw errors. To
achieve a similar effect one could do the following:
```javascript
var result = UglifyJS.minify(code, options);
if (result.error) throw result.error;
```

## 最小化选项
## Minify options

- `warnings` (default `false`) — 传 `true`的话，会在`result.warnings`中返回压缩过程的警告。传 `"verbose"`获得更详细的警告。

- `parse` (default `{}`) — 如果你要指定额外的[解析配置parse options](#parse-options),传配置对象。

- `compress` (default `{}`) — 传`false`就完全跳过压缩。传一个对象来自定义 [压缩配置compress options](#compress-options)。

- `mangle` (default `true`) — 传 `false`就跳过混淆名字。传对象来指定[混淆配置mangle options](#mangle-options) (详情如下).

  - `mangle.properties` (default `false`) — 传一个对象来自定义[混淆属性配置mangle property options](#mangle-properties-options).

- `output` (default `null`) — 要自定义就传个对象来指定额外的 [输出配置output options](#output-options).  默认是压缩到最优化。

- `sourceMap` (default `false`) - 传一个对象来自定义
  [sourcemap配置source map options](#source-map-options).

- `toplevel` (default `false`) - 如果你要混淆（和干掉没引用的）最高作用域中的变量和函数名，就传`true`。

- `nameCache` (default `null`) - 如果你要缓存 `minify()`多处调用的经混淆的变量名、属性名，就传一个空对象`{}`或先前用过的`nameCache`对象。
注意:这是个可读/可写属性。`minify()`会读取这个对象的nameCache状态，并在最小化过程中更新，以便保留和供用户在外部使用。

- `ie8` (default `false`) - 传 `true` 来支持 IE8.


## 最小化配置的结构
## Minify options structure

```javascript
{
    warnings: false,
    parse: {
        // parse options
    },
    compress: {
        // compress options
    },
    mangle: {
        // mangle options

        properties: {
            // mangle property options
        }
    },
    output: {
        // output options
    },
    sourceMap: {
        // source map options
    },
    nameCache: null, // or specify a name cache object
    toplevel: false,
    ie8: false,
}
```
### sourcemap配置
### Source map options

这样生成sourcemap：
```javascript
var result = UglifyJS.minify({"file1.js": "var a = function() {};"}, {
    sourceMap: {
        filename: "out.js",
        url: "out.js.map"
    }
});
console.log(result.code); // minified output
console.log(result.map);  // source map
```
要注意，此时sourcemap并不会保存为一份文件，它只会返回在`result.map`中。
`sourceMap.url` 传入的值只用来在`result.code`中设置`//# sourceMappingURL=out.js.map` ，`filename` 的值只用来在sourcemap文件中设置 `file`属性(详情看 [规范][sm-spec])。

你可以把`sourceMap.url`设为`true` ，这样sourcemap会加在代码末尾。

你也可以指定sourcemap中的源文件根目录（sourceRoot）属性：

```javascript
var result = UglifyJS.minify({"file1.js": "var a = function() {};"}, {
    sourceMap: {
        root: "http://example.com/src",
        url: "out.js.map"
    }
});
```

如果你要压缩*从其他文件编译得来的*带一份sourcemap的JS文件，你可以用`sourceMap.content`参数：

```javascript
var result = UglifyJS.minify({"compiled.js": "compiled code"}, {
    sourceMap: {
        content: "content from compiled.js.map",
        url: "minified.js.map"
    }
});
// same as before, it returns `code` and `map`
```

如果你要用` X-SourceMap `请求头，你可以忽略 `sourceMap.url`。

## 解析配置
## Parse options

- `bare_returns` (default `false`) -- 支持在顶级作用域中 `return` 声明。
- `html5_comments` (default `true`)
- `shebang` (default `true`) -- 支持在第一行用 `#!command`

## 压缩配置
## Compress options

- `sequences`(default: true) -- 连续声明变量，用逗号隔开来。可以设置为正整数来指定连续声明的最大长度。如果设为`true` 表示默认`200`个，设为`false`或`0`则禁用。 `sequences`至少要是`2`,`1`的话等同于`true`（即`200`)。默认的sequences设置有极小几率会导致压缩很慢，所以推荐设置成`20`或以下。

- `properties` -- 用`.`来重写属性引用，例如`foo["bar"] → foo.bar`

- `dead_code` -- 移除没被引用的代码

- `drop_debugger` -- 移除 `debugger;` 

- `unsafe` (default: false) -- 使用 "unsafe"转换 (下面详述)

- `unsafe_comps` (default: false) -- 保留`<` 和 `<=`不被换成 `>` 和 `>=`。假如某些运算对象是用`get`或 `valueOf`object得出的时候，转换可能会不安全，可能会引起运算对象的改变。此选项只有当 `comparisons`和`unsafe_comps` 都设为true时才会启用。

- `unsafe_Func` (default: false) -- 当 `Function(args, code)`的`args` 和 `code`都是字符串时，压缩并混淆。

- `unsafe_math` (default: false) -- 优化数字表达式，例如`2 * x * 3` 变成 `6 * x`, 可能会导致不精确的浮点数结果。

- `unsafe_proto` (default: false) -- 把`Array.prototype.slice.call(a)` 优化成 `[].slice.call(a)`

- `unsafe_regexp` (default: false) -- 如果`RegExp` 的值是常量，替换成变量。

- `conditionals` -- 优化`if`等判断以及条件选择

- `comparisons` --  把结果必然的运算优化成二元运算，例如`!(a <= b) → a > b` (只有设置了 `unsafe_comps`时才生效)；尽量转成否运算。例如 `a = !b && !c && !d && !e → a=!(b||c||d||e)` 

- `evaluate` -- 尝试计算常量表达式

- `booleans` -- 优化布尔运算，例如 `!!a? b : c → a ? b : c`

- `typeofs` -- 默认 `true`.  转换 `typeof foo == "undefined"` 成 `foo === void 0`. 注意：如果要适配IE10或以下，由于已知的问题，推荐设成`false` 。

- `loops` -- 当`do`、`while` 、 `for`循环的判断条件可以确定是，对其进行优化。 

- `unused` -- 干掉没有被引用的函数和变量。（除非设置`"keep_assign"`，否则变量的简单直接赋值也不算被引用。）

- `toplevel` -- 干掉顶层作用域中没有被引用的函数 (`"funcs"`)和/或变量(`"vars"`) (默认是`false` , `true` 的话即函数变量都干掉)

- `top_retain` -- 当设了`unused`时，保留顶层作用域中的某些函数变量。(可以写成数组，用逗号隔开，也可以用正则或函数. 参考`toplevel`)

- `hoist_funs` -- 提升函数声明

- `hoist_vars` (default: false) -- 提升 `var` 声明 (默认是`false`,因为那会加大文件的size)

- `if_return` -- 优化 if/return 和 if/continue

- `inline` -- 包裹简单函数。 

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

- `side_effects` -- 默认 `true`. 传`false`禁用丢弃纯函数。如果一个函数被调用前有一段`/*@__PURE__*/` or `/*#__PURE__*/` 注释，该函数会被标注为纯函数。例如 `/*@__PURE__*/foo();`

## 混淆配置
## Mangle options

- `reserved` (default `[]`)。 传一个不需要混淆的名字的数组。 Example: `["foo", "bar"]`.

 - `toplevel` (default `false`)。混淆那些定义在顶层作用域的名字（默认禁用）。ß

 - `keep_fnames`(default `false`)。传`true`的话就不混淆函数名。对那些依赖`Function.prototype.name`的代码有用。延展阅读：`keep_fnames` [压缩配置](#compress-options).

 - `eval` (default `false`)。混淆那些在with或eval中出现的名字。

```javascript
// test.js
var globalVar;
function funcName(firstLongName, anotherLongName) {
    var myVariable = firstLongName +  anotherLongName;
}
```
```javascript
var code = fs.readFileSync("test.js", "utf8");

UglifyJS.minify(code).code;
// 'function funcName(a,n){}var globalVar;'

UglifyJS.minify(code, { mangle: { reserved: ['firstLongName'] } }).code;
// 'function funcName(firstLongName,a){}var globalVar;'

UglifyJS.minify(code, { mangle: { toplevel: true } }).code;
// 'function n(n,a){}var a;'
```

### 混淆属性的配置
### Mangle properties options

- `reserved` (default: `[]`) -- 不混淆在`reserved` 数组里的属性名.
- `regex` (default: `null`) -— 传一个正则，只混淆匹配该正则的属性名。 
- `keep_quoted` (default: `false`) -— 只混淆不在括号内的属性名.
- `debug` (default: `false`) -— 用原名字来组成混淆后的名字.
  传空字符串`""` 来启用,或者非空字符串作为debu后缀。（例如`"abc"`, `foo.bar`=>`foo.barabc`)
- `builtins` (default: `false`) -- 传 `true`的话，允许混淆内置的DOM属性名。不推荐使用。 

## 输出配置
## Output options

代码生成器默认会尽量输出最简短的代码。假如你要美化一下输出代码，可以传`--beautify` (`-b`)。你也可以传更多的参数来控制输出代码：

- `ascii_only` (default `false`) -- 忽略字符串和正则（导致非ascii字符失效）中的Unicode字符。
- `beautify` (default `true`) -- 是否美化输出代码。传`-b`的话就是设成true。假如你想生成最小化的代码同时又要用其他设置来美化代码，你可以设`-b beautify=false`。
- `bracketize` (default `false`) -- 永远在`if`, `for`,`do`, `while`, `with`后面加上大括号，即使循环体只有一句。
- `comments` (default `false`) -- 传 `true` 或 `"all"`保留全部注释,传 `"some"`保留部分，传正则 (例如 `/^!/`) 或者函数也行。
- `indent_level` (default 4) 缩进格数
- `indent_start` (default 0) -- 每行前面加几个空格
- `inline_script` (default `false`) -- 避免字符串中出现`</script`中的斜杠
- `keep_quoted_props` (default `false`) -- 如果启用，会保留对象属性名的引号。
- `max_line_len` (default 32000) -- 最大行宽（压缩后的代码）
- `space-colon` (default `true`) -- 在冒号后面加空格
- `preamble` (default `null`) -- 如果要传的话，必须是字符串。它会被加在输出文档的前面。sourcemap会随之调整。例如可以用来插入版权信息。
- `preserve_line` (default `false`) -- 传 `true` 就保留空行，但只在`beautify` 设为`false`时有效。ß
- `quote_keys` (default `false`) -- 传`true`的话会在对象所有的键加上括号
- `quote_style` (default `0`) -- 影响字符串的括号格式（也会影响属性名和指令）。
- `0` -- 倾向使用双引号，字符串里还有引号的话就是单引号。
- `1` -- 永远单引号
- `2` -- 永远双引号
- `3` -- 永远是本来的引号
- `semicolons` (default `true`) -- 用分号分开多个声明。如果你传`false`,则总会另起一行，增强输出文件的可读性。（gzip前体积更小，gzip后稍大一点点）
- `shebang` (default `true`) -- 保留开头的 shebang `#!` (bash 脚本)
- `width` (default 80) -- 仅在美化时生效，设定一个行宽让美化器尽量实现。这会影响行中文字的数量（不包括缩进）。当前本功能实现得不是非常好，但依然让美化后的代码可读性大大增强。

- `wrap_iife` (default `false`) --传`true`的话，把立即执行函数括起来。 更多详情看这里
  [#640](https://github.com/mishoo/UglifyJS2/issues/640) 

# 综合应用
# Miscellaneous

### 保留版权告示或其他注释

你可以传入`--comments`让输出文件中保留某些注释。默认时会保留JSDoc-style的注释（包含"@preserve","@license" 或 "@cc_on"（为IE所编译））。你可以传入`--comments all`来保留全部注释，或者传一个合法的正则来保留那些匹配到的注释。例如`--comments /^!/`会保留`/*! Copyright Notice */`这样的注释。 

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

### `unsafe``compress`配置
### The `unsafe` `compress` option


在某些刻意营造的案例中，启用某些转换**有可能**会打断代码的逻辑，但绝大部分情况下是安全的。你可能会想尝试一下，因为这毕竟会减少文件体积。以下是某些例子：

- `new Array(1, 2, 3)` 或 `Array(1, 2, 3)` → `[ 1, 2, 3 ]`
- `new Object()` → `{}`
- `String(exp)` 或 `exp.toString()` → `"" + exp`
- `new Object/RegExp/Function/Error/Array (...)` → 我们干掉用`new`的
- `void 0` → `undefined` (假如作用域中有一个变量名叫"undefined";我们这么做是因为变量名会被混淆成单字符）

### 编译条件语句
### Conditional compilation

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
这样构建你的代码：
```
  uglifyjs build/defines.js js/foo.js js/bar.js... -c
```
UglifyJS会注意到这些常量。因为它们无法改变，所以它们会被认为是没被引用而被照样干掉。如果你用`const`声明，构建后还会被保留。如果你的运行环境低于ES6、不支持`const`，请用`var`声明加上`reduce_vars`设置（默认启用）来实现。

#### 编译条件语句API

你也可以通过程序API来设置编译配置。其中有差别的是一个压缩器属性`global_defs`：
```javascript
var result = UglifyJS.minify(fs.readFileSync("input.js", "utf8"), {
    compress: {
        dead_code: true,
        global_defs: {
            DEBUG: false
        }
    }
});
```
在`global_defs`配`"@"`前缀的表达式，UglifyJS才会替换成语句表达式：
```javascript
UglifyJS.minify("alert('hello');", {
    compress: {
        global_defs: {
            "@alert": "console.log"
        }
    }
}).code;
// returns: 'console.log("hello");'
```
否则会替换成字符串:
```javascript
UglifyJS.minify("alert('hello');", {
    compress: {
        global_defs: {
            "alert": "console.log"
        }
    }
}).code;
// returns: '"console.log"("hello");'
```

### 使用`minify()`获得原生UglifyJS ast
### Using native Uglify AST with `minify()`

```javascript
// 例子: 只解析代码，获得原生Uglify AST

var result = UglifyJS.minify(code, {
    parse: {},
    compress: false,
    mangle: false,
    output: {
        ast: true,
        code: false  // optional - faster if false
    }
});

// result.ast 即是原生 Uglify AST
```
```javascript
// 例子: 输入原生 Uglify AST，接着把它压缩并混淆，生成代码和原生ast

var result = UglifyJS.minify(ast, {
    compress: {},
    mangle: {},
    output: {
        ast: true,
        code: true  // 可选，false更快
    }
});

// result.ast 是原生 Uglify AST
// result.code 是字符串格式的最小化后的代码
```

### 使用 Uglify AST
### Working with Uglify AST


可以通过[`TreeWalker`](http://lisperator.net/uglifyjs/walk)和[`TreeTransformer`](http://lisperator.net/uglifyjs/transform)分别横截（？transversal）和转换原生AST。 

### ESTree/SpiderMonkey AST

UglifyJS有自己的抽象语法树格式；为了某些[现实的原因](http://lisperator.net/blog/uglifyjs-why-not-switching-to-spidermonkey-ast/)
我们无法在内部轻易地改成使用SpiderMonkey AST。但UglifyJS现在有了一个可以输入SpiderMonkeyAST的转换器。
例如[Acorn][acorn] ，这是一个超级快的生成SpiderMonkey AST的解释器。它带有一个实用的迷你CLI，能解释一个文件、把AST转存为JSON并标准输出。可以这样用UglifyJS来压缩混淆：
```
    acorn file.js | uglifyjs --spidermonkey -m -c
```
 `-p --spidermonkey`选项能让UglifyJS知道输入文件并非JavaScript，而是SpiderMonkey AST生成的JSON代码。这事我们不用自己的解释器，只把AST转成我们内部AST。

### 使用 Acorn 来解释代码
### Use Acorn for parsing


更有趣的是，我们加了 `-p --acorn`选项来使用Acorn解释所有代码。如果你传入这个选项，UglifyJS会`require("acorn")`

Acorn确实非常快（650k代码原来要380ms，现在只需250ms），但转换Acorn产生的SpiderMonkey树会额外花费150ms。所以总共比UglifyJS自己的解释器还要多花一点时间。

[acorn]: https://github.com/ternjs/acorn
[sm-spec]: https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k

### Uglify快速最小化模式
### Uglify Fast Minify Mode

很少人知道，对大多数js代码而言，其实移除空格和混淆符号已经占了减少代码体积之中到的95%--不必细致地转换。简单地禁用`压缩compress`能加快UglifyJS的构建速度三四倍。我们可以比较一下
[`butternut`](https://www.npmjs.com/package/butternut)和只使用`混淆mangle`的模式的Uglify的压缩速度与gzip大小：
[`butternut`](https://www.npmjs.com/package/butternut):

| d3.js | minify size | gzip size | minify time (seconds) |
| --- | ---: | ---: | ---: |
| original | 451,131 | 108,733 | - |
| uglify-js@3.0.24 mangle=false, compress=false | 316,600 | 85,245 | 0.70 |
| uglify-js@3.0.24 mangle=true, compress=false | 220,216 | 72,730 | 1.13 |
| butternut@0.4.6 | 217,568 | 72,738 | 1.41 |
| uglify-js@3.0.24 mangle=true, compress=true | 212,511 | 71,560 | 3.36 |
| babili@0.1.4 | 210,713 | 72,140 | 12.64 |

在CLI中，这样启用快速最小化模式:
```
uglifyjs file.js -m
```

API这样用:
```js
UglifyJS.minify(code, { compress: false, mangle: true });
```
