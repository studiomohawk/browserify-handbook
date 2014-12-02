# introduction

This document covers how to use [browserify](http://browserify.org) to build
modular applications.

本ドキュメントでは[browserify](http://browserify.org)を使った、モジュール・アプリケーションの作り方について紹介します。

[![cc-by-3.0](http://i.creativecommons.org/l/by/3.0/80x15.png)](http://creativecommons.org/licenses/by/3.0/)

browserify is a tool for compiling
[node-flavored](http://nodejs.org/docs/latest/api/modules.html) commonjs modules
for the browser.

browserifyとは[Nodeで採用されている](http://nodejs.org/docs/latest/api/modules.html)、CommonJSモジュールをブラウザで利用できるようにコンパイルを行うツールです。

You can use browserify to organize your code and use third-party libraries even
if you don't use [node](http://nodejs.org) itself in any other capacity except
for bundling and installing packages with npm.

npmを使ってパッケージのインストール、またはバンドルする以外は[Node](http://nodejs.org)そのものを利用していなくても、browserifyを利用することで、自分のコードや第三者が作成したライブラリを構造化することができるようになります。

The module system that browserify uses is the same as node, so
packages published to [npm](https://npmjs.org) that were originally intended for
use in node but not browsers will work just fine in the browser too.

browserifyのモジュール・システムはNodeのそれと同じです。そのため、[npm](https://npmjs.org)で公開されているパッケージで元々はブラウザでではなく、Node.jsで利用されることを想定しているものであっても、ブラウザ上でも動作させることができます。

Increasingly, people are publishing modules to npm which are intentionally
designed to work in both node and in the browser using browserify and many
packages on npm are intended for use in just the browser.
[npm is for all javascript](http://maxogden.com/node-packaged-modules.html),
front or backend alike.

多くの人がNodeとブラウザの両方で動作するモジュールをnpmで公開し初めていたり、npm上にブラウザでしか動作しないパッケージも増えてきています。
それはフロントエンドも、バックエンドも関係なく[npmはすべてのJavaScriptのためにあります](http://maxogden.com/node-packaged-modules.html)。

# table of contents

- [introduction](#introduction)
- [table of contents](#table-of-contents)
- [node packaged manuscript](#node-packaged-manuscript)
- [node packaged modules](#node-packaged-modules)
  - [require](#require)
  - [exports](#exports)
  - [bundling for the browser](#bundling-for-the-browser)
  - [how browserify works](#how-browserify-works)
  - [how node_modules works](#how-node_modules-works)
  - [why concatenate](#why-concatenate)
- [development](#development)
  - [source maps](#source-maps)
    - [exorcist](#exorcist)
  - [auto-recompile](#auto-recompile)
  - [using the api directly](#using-the-api-directly)
  - [grunt](#grunt)
  - [gulp](#gulp)
- [builtins](#builtins)
  - [Buffer](#Buffer)
  - [process](#process)
  - [global](#global)
  - [__filename](#__filename)
  - [__dirname](#__dirname)
- [transforms](#transforms)
  - [writing your own](#writing-your-own)
- [package.json](#package.json)
  - [browser field](#browser-field)
  - [browserify.transform field](#browserifytransform-field)
- [finding good modules](#finding-good-modules)
  - [module philosophy](#module-philosophy)
- [organizing modules](#organizing-modules)
  - [avoiding ../../../../../../..](#avoiding-)
  - [non-javascript assets](#non-javascript-assets)
  - [reusable components](#reusable-components)
- [testing in node and the browser](#testing-in-node-and-the-browser)
  - [testing libraries](#testing-libraries)
  - [code coverage](#code-coverage)
  - [testling-ci](#testling-ci)
- [bundling](#bundling)
  - [saving bytes](#saving-bytes)
  - [standalone](#standalone)
  - [external bundles](#external-bundles)
  - [ignoring and excluding](#ignoring-and-excluding)
  - [browserify cdn](#browserify-cdn)
- [shimming](#shimming)
  - [browserify-shim](#browserify-shim)
- [partitioning](#partitioning)
  - [factor-bundle](#factor-bundle)
  - [partition-bundle](#partition-bundle)
- [compiler pipeline](#compiler-pipeline)
  - [build your own browserify](#build-your-own-browserify)
  - [labeled phases](#labeled-phases)
    - [deps](#deps)
      - [insert-module-globals](#insert-module-globals)
    - [json](#json)
    - [unbom](#unbom)
    - [syntax](#syntax)
    - [sort](#sort)
    - [dedupe](#dedupe)
    - [label](#label)
    - [emit-deps](#emit-deps)
    - [debug](#debug)
    - [pack](#pack)
    - [wrap](#wrap)
  - [browser-unpack](#browser-unpack)
- [plugins](#plugins)
  - [using plugins](#using-plugins)
  - [authoring plugins](#authoring-plugins)

# node packaged manuscript

You can install this handbook with npm, appropriately enough. Just do:

```
npm install -g browserify-handbook
```

Now you will have a `browserify-handbook` command that will open this readme
file in your `$PAGER`. Otherwise, you may continue reading this document as you
are presently doing.

# node packaged modules

Before we can dive too deeply into how to use browserify and how it works, it is
important to first understand how the
[node-flavored version](http://nodejs.org/docs/latest/api/modules.html)
of the commonjs module system works.

browserifyの利用方法やどのように動作するのかについて詳しく紹介する前に、[Nodeで採用されている](http://nodejs.org/docs/latest/api/modules.html)、CommonJSモジュールについて理解する必要があります。

## require

In node, there is a `require()` function for loading code from other files.

Nodeでは、`require()`関数を使って他のファイルからコードを呼び出すことができます。

If you install a module with [npm](https://npmjs.org):

以下の様に[npm](https://npmjs.org)からモジュールをインストールすると、

```
npm install uniq
```

Then in a file `nums.js` we can `require('uniq')`:

`nums.js`というファイルから`require('uniq')`とすることができます。

```
var uniq = require('uniq');
var nums = [ 5, 2, 1, 3, 2, 5, 4, 2, 0, 1 ];
console.log(uniq(nums));
```

The output of this program when run with node is:

Nodeでこのプログラムを実行すると以下の様に出力されます。

```
$ node nums.js
[ 0, 1, 2, 3, 4, 5 ]
```

You can require relative files by requiring a string that starts with a `.`. For
example, to load a file `foo.js` from `main.js`, in `main.js` you can do:

`require()`関数の引数を文字列`.`から開始することで相対パスのファイルを呼び出すこともできます。例として`foo.js`を`main.js`から呼び出すとして、`main.js`で以下の様にすることができます。

``` js
var foo = require('./foo.js');
console.log(foo(4));
```

If `foo.js` was in the parent directory, you could use `../foo.js` instead:

`foo.js`が親ディレクトリにある場合、代わりに`../foo.js`という様にも記述できます。

``` js
var foo = require('../foo.js');
console.log(foo(4));
```

or likewise for any other kind of relative path. Relative paths are always
resolved with respect to the invoking file's location.

どんな種類の相対パスにも同じ事が言えます。相対パスは常に実行するファイルから見たパスとなります。

Note that `require()` returned a function and we assigned that return value to a
variable called `uniq`. We could have picked any other name and it would have
worked the same. `require()` returns the exports of the module name that you
specify.

`require()`は関数を戻り値とし、その値を変数`uniq`に代入している点に注目してください。`uniq`以外の変数名にしてももちろん同じように動作します。`require()`は公開を指定したモジュール名を戻します。

How `require()` works is unlike many other module systems where imports are akin
to statements that expose themselves as globals or file-local lexicals with
names declared in the module itself outside of your control. Under the node
style of code import with `require()`, someone reading your program can easily
tell where each piece of functionality came from. This approach scales much
better as the number of modules in an application grows.

`require()`の挙動は他のモジュール・システムとは異なり、インポートするということは、モジュールそのものが宣言する、制御することができない
グローバル、あるいはファイル毎のローカル構文スコープが公開される
ということと似ています。  
Node方式のインポートである`require()`は、誰かが自分の書いたプログラムを読む際に、それぞれの機能がどこから来たのかわかりやすくしてくれます。このアプローチはアプリケーションが大きくなるにつれて、モジュールが増えれば増えるほどスケールしやすくなります。

## exports

To export a single thing from a file so that other files may import it, assign
over the value at `module.exports`:

他のファイルからインポートできるようにするために、あるファイルから1つの機能を公開する方法として、値を`module.exports`に代入する方法があります。

``` js
module.exports = function (n) {
    return n * 111
};
```

Now when some module `main.js` loads your `foo.js`, the return value of
`require('./foo.js')` will be the exported function:

こうすることで、`main.js`モジュールが`foo.js`を読み込む際に、`require('./foo.js')`の戻り値は公開された関数となります。

``` js
var foo = require('./foo.js');
console.log(foo(5));
```

This program will print:

このプログラムは以下を結果を表示します。

```
555
```

You can export any kind of value with `module.exports`, not just functions.

For example, this is perfectly fine:

`module.exports`は関数だけではなく、どんな値でも公開することができます。  
例として以下のようにすることも何ら問題がありません。

``` js
module.exports = 555
```

and so is this:

もちろん、以下の場合も問題ありません。

``` js
var numbers = [];
for (var i = 0; i < 100; i++) numbers.push(i);

module.exports = numbers;
```

There is another form of doing exports specifically for exporting items onto an
object. Here, `exports` is used instead of `module.exports`:

オブジェクトに値を公開するための他の方法もあります。`module.exports`の代わりに`exports`を利用している点に注目してください。

``` js
exports.beep = function (n) { return n * 1000 }
exports.boop = 555
```

This program is the same as:

このプログラムは以下と同じになります。

``` js
module.exports.beep = function (n) { return n * 1000 }
module.exports.boop = 555
```

because `module.exports` is the same as `exports` and is initially set to an
empty object.

Note however that you can't do:

`module.exports`は`exports`と同じものですが、`module.exports`では空のオブジェクトに対して値をセットしています。

しかし、以下の様にはできない点に注意してください。

``` js
// this doesn't work
exports = function (n) { return n * 1000 }
```

because the export value lives on the `module` object, and so assigning a new
value for `exports` instead of `module.exports` masks the original reference. 

Instead if you are going to export a single item, always do:

公開する値は`module`オブジェクトにあるため、`module.exports`ではなく、`exports`に対して新しい値を代入してしまうと、元々の参照を隠してしまうことになるためです。

1つのアイテムを公開したい場合は、必ず以下の様にします。

``` js
// instead
module.exports = function (n) { return n * 1000 }
```

If you're still confused, try to understand how modules work in
the background:

もしまだ混乱している場合、モジュールがシステムの裏側でどのように動作しているのかを理解するようにしてみてはどうでしょうか?

``` js
var module = {
  exports: {}
};

// If you require a module, it's basically wrapped in a function
// モジュールを`require()`する場合、基本的には関数で囲む、ということを意味します
(function(module, exports) {
  exports = function (n) { return n * 1000 };
}(module, module.exports))

console.log(module.exports); // it's still an empty object :(
// まだ空のオブジェクトです :(
```

Most of the time, you will want to export a single function or constructor with
`module.exports` because it's usually best for a module to do one thing.

ほとんどの場合、`module.exports`を使って、ある1つの関数、またはコンストラクタを公開するはずです。なぜなら、モジュールはある1つのことだけをするべきだからです。

The `exports` feature was originally the primary way of exporting functionality
and `module.exports` was an afterthought, but `module.exports` proved to be much
more useful in practice at being more direct, clear, and avoiding duplication.

In the early days, this style used to be much more common:

`exports`は元々は機能を公開するための主な機構で、`module.exports`は後付でした。しかし、`module.exports`はより直接的で、明確で、重複を避けることができ、実践において高い利便性を持っています。

Nodeの黎明期には以下の様なスタイルをよく見かけることができました。

foo.js:

``` js
exports.foo = function (n) { return n * 111 }
```

main.js:

``` js
var foo = require('./foo.js');
console.log(foo.foo(5));
```

but note that the `foo.foo` is a bit superfluous. Using `module.exports` it
becomes more clear:

`foo.foo`は少々余分です。そこで`module.exports`を利用するとより明確にすることができます。

foo.js:

``` js
module.exports = function (n) { return n * 111 }
```

main.js:

``` js
var foo = require('./foo.js');
console.log(foo(5));
```

## bundling for the browser

To run a module in node, you've got to start from somewhere.

In node you pass a file to the `node` command to run a file:

まずは千里の道も一歩から、Nodeでモジュールを実行するためには、次の様に、`node`コマンドにファイルを渡します。

```
$ node robot.js
beep boop
```

In browserify, you do this same thing, but instead of running the file, you
generate a stream of concatenated javascript files on stdout that you can write
to a file with the `>` operator:

browserifyでも同じ事をします。ただ、ファイルを実行する代わりに、stdoutに結合したJavaScriptを生成します。そうすることで、`>`オペレータを使ってファイルに書き込めるようになります。

```
$ browserify robot.js > bundle.js
```

Now `bundle.js` contains all the javascript that `robot.js` needs to work.
Just plop it into a single script tag in some html:

これで、`bundle.js`には`robot.js`を実行するのに必要な全てのJavaScriptが含まれることになります。  
あとはただどこかのHTML内のスクリプトタグからぽいっとリンクするだけです。

``` html
<html>
  <body>
    <script src="bundle.js"></script>
  </body>
</html>
```

Bonus: if you put your script tag right before the `</body>`, you can use all of
the dom elements on the page without waiting for a dom onready event.

おまけ: スクリプトタグを`</body>`の直前に設置すると、DOMの`onReady`イベントを待つ必要なく、すべてのDOM要素を利用できます。

There are many more things you can do with bundling. Check out the bundling
section elsewhere in this document.

バンドリングでできることは他にも数多くあります。詳しくは本ドキュメントのどこかにあるバンドリングについてのセクションを参照してください。

## how browserify works

Browserify starts at the entry point files that you give it and searches for any
`require()` calls it finds using
[static analysis](http://npmjs.org/package/detective)
of the source code's
[abstract syntax tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree).

まず、Browserifyは渡されたエントリー・ポイントとなるファイルのソースコードの[抽象構文木](https://ja.wikipedia.org/wiki/%E6%8A%BD%E8%B1%A1%E6%A7%8B%E6%96%87%E6%9C%A8)の[静的解析](http://npmjs.org/package/detective)を行い、`require()`を見つけ出します。

For every `require()` call with a string in it, browserify resolves those module
strings to file paths and then searches those file paths for `require()` calls
recursively until the entire dependency graph is visited.

それからBrowserifyは、`require()`の引数として渡した文字列のファイルパスからモジュールを見つけ、さらに、そのモジュール内に記述してある`require()`でも同様の処理を行います。この処理はすべての依存グラフを解決するまで繰り返されます。

Each file is concatenated into a single javascript file with a minimal
`require()` definition that maps the statically-resolved names to internal IDs.

それぞれのファイルは解析し解決したモジュール名を内部的なIDに紐付け、最低限必要な`require()`の定義と共に、1つのJavaScriptファイルに結合されます。

This means that the bundle you generate is completely self-contained and has
everything your application needs to work with a pretty negligible overhead.

こうすることで生成したバンドルは、完全に自己完結し、ごくわずかなオーバーヘッドで、アプリケーションとして動作するのに必要なすべてことを実行できるようになります。

For more details about how browserify works, check out the compiler pipeline
section of this document.

Browserifyがどのように動作するのかについてより詳しい情報が必要な場合は、本ドキュメント内のコンパイラ・パイプラインのセクションを参照してください。

## how node_modules works

node has a clever algorithm for resolving modules that is unique among rival
platforms.

Instead of resolving packages from an array of system search paths like how
`$PATH` works on the command line, node's mechanism is local by default.

If you `require('./foo.js')` from `/beep/boop/bar.js`, node will
look for `./foo.js` in `/beep/boop/foo.js`. Paths that start with a `./` or
`../` are always local to the file that calls `require()`.

If however you require a non-relative name such as `require('xyz')` from
`/beep/boop/foo.js`, node searches these paths in order, stopping at the first
match and raising an error if nothing is found:

```
/beep/boop/node_modules/xyz
/beep/node_modules/xyz
/node_modules/xyz
```

For each `xyz` directory that exists, node will first look for a
`xyz/package.json` to see if a `"main"` field exists. The `"main"` field defines
which file should take charge if you `require()` the directory path.

For example, if `/beep/node_modules/xyz` is the first match and
`/beep/node_modules/xyz/package.json` has:

```
{
  "name": "xyz",
  "version": "1.2.3",
  "main": "lib/abc.js"
}
```

then the exports from `/beep/node_modules/xyz/lib/abc.js` will be returned by
`require('xyz')`.

If there is no `package.json` or no `"main"` field, `index.js` is assumed:

```
/beep/node_modules/xyz/index.js
```

If you need to, you can reach into a package to pick out a particular file. For
example, to load the `lib/clone.js` file from the `dat` package, just do:

```
var clone = require('dat/lib/clone.js')
```

The recursive node_modules resolution will find the first `dat` package up the
directory hierarchy, then the `lib/clone.js` file will be resolved from there.
This `require('dat/lib/clone.js')` approach will work from any location where
you can `require('dat')`.

node also has a mechanism for searching an array of paths, but this mechanism is
deprecated and you should be using `node_modules/` unless you have a very good
reason not to.

The great thing about node's algorithm and how npm installs packages is that you
can never have a version conflict, unlike most every other platform. npm
installs the dependencies of each package into `node_modules`.

Each library gets its own local `node_modules/` directory where its dependencies
are stored and each dependency's dependencies has its own `node_modules/`
directory, recursively all the way down.

This means that packages can successfully use different versions of libraries in
the same application, which greatly decreases the coordination overhead
necessary to iterate on APIs. This feature is very important for an ecosystem
like npm where there is no central authority to manage how packages are
published and organized. Everyone may simply publish as they see fit and not
worry about how their dependency version choices might impact other dependencies
included in the same application.

You can leverage how `node_modules/` works to organize your own local
application modules too. See the `avoiding ../../../../../../..` section for
more.

## why concatenate

Browserify is a build step that runs on the server. It generates a single bundle
file that has everything in it.

Browserifyはサーバ上で実行されるビルド・ステップです。すべての機能が詰まった1つのバンドルファイルを生成します。

Here are some other ways of implementing module systems for the browser and what
their strengths and weaknesses are:

ブラウザ内でモジュールシステムを実装する他の方法について、そして、それらの強みと弱みについて、ここから見ていきましょう。

### window globals

Instead of a module system, each file defines properties on the window global
object or develops an internal namespacing scheme.

モジュールシステムの代わりに、それぞれのファイルにグローバルオブジェクトのプロパティを定義したり、内部的な名前空間スキームを開発する方法があります。

This approach does not scale well without extreme diligence since each new file
needs an additional `<script>` tag in all of the html pages where the
application will be rendered. Further, the files tend to be very order-sensitive
because some files need to be included before other files the expect globals to
already be present in the environment.

この手法でスケールするのは、相当の努力が必要になります。  
新しいファイルを作成する度に、アプリケーションを動作させるすべてのHTMLページに追加の`<script>`タグが必要になります。さらにそれらのファイルは、グローバルオブジェクトへの定義が存在することが必須になるため、どこかのファイルを呼び出す前に別のファイルを呼び出す必要があったりと、読み込み順に対して細心の注意が必要になります。

It can be difficult to refactor or maintain applications built this way.
On the plus side, all browsers natively support this approach and no server-side
tooling is required.

この方法でビルドしてしまうと、リファクタを行うのにも、アプリケーションをメンテナンスしつづけることも難しくなりやすいでしょう。  
利点としては、この方法はすべてのブラウザでサポートされ、サーバサイド側のツールも必要としない点が上げられます。

This approach tends to be very slow since each `<script>` tag initiates a
new round-trip http request.

また、この方法は`<script>`一つ一つが新たなHTTPリクエストのラウンド・トリップを必要とするため、ネットワーク・パフォーマンスも悪くなる傾向になります。

### concatenate

Instead of window globals, all the scripts are concatenated beforehand on the
server. The code is still order-sensitive and difficult to maintain, but loads
much faster because only a single http request for a single `<script>` tag needs
to execute.

グローバルオブジェクトの手法の代わりに、すべてのスクリプトをサーバが読み込む前に結合する方法もあります。コードそのものは、読み込み順に対して注意が必要なままで、メンテナンスも難しいままですが、HTTPリクエストは1つでよくなるため、ネットワーク・パフォーマンスは改善します。

Without source maps, exceptions thrown will have offsets that can't be easily
mapped back to their original files.

しかし、Source mapsを利用していない場合、例外処理が投げられた場合に、オリジナルファイルと異なる行番号を知らせるため、エラーの発見が難しくなります。

### AMD

Instead of using `<script>` tags, every file is wrapped with a `define()`
function and callback. [This is AMD](http://requirejs.org/docs/whyamd.html). 

`<script>`の代わりに、すべてのファイルを`define()`関数とコールバックで囲む方法が[AMDです](http://requirejs.org/docs/whyamd.html)。

The first argument is an array of modules to load that maps to each argument
supplied to the callback. Once all the modules are loaded, the callback fires.

第一引数にはコールバックに渡される引数にそれぞれマップされる読み込むモジュールの配列を指定します。  
すべてのモジュールが読み込まれるとコールバックが実行されます。

``` js
define(['jquery'] , function ($) {
    return function () {};
});
```

You can give your module a name in the first argument so that other modules can
include it.

名前を付けたモジュールを第一引数に渡すこともできます。そうすることで、他のモジュールも含めることができます。

There is a commonjs sugar syntax that stringifies each callback and scans it for
`require()` calls
[with a regexp](https://github.com/jrburke/requirejs/blob/master/require.js#L17).

CommonJS風に記述するための糖衣構文も用意されています。ここでは[正規表現](https://github.com/jrburke/requirejs/blob/master/require.js#L17)を使って、それぞれのコールバック内の`require()`をマッチさせています。

Code written this way is much less order-sensitive than concatenation or globals
since the order is resolved by explicit dependency information.

このように書かれたコードは、明示的な依存関係の情報を使って呼び出しの順序を解決するため、単純な結合や、グローバル・プロパティを使った方法と比べるとはるかに呼び出し順についての懸念が薄れます。

For performance reasons, most of the time AMD is bundled server-side into a
single file and during development it is more common to actually use the
asynchronous feature of AMD.

パフォーマンスを向上させるため、大抵AMDはサーバサイドで1つのファイルにバンドルされ利用されます。開発中はAMDが持つ非同期の機能を利用することが多いです。

### bundling commonjs server-side

If you're going to have a build step for performance and a sugar syntax for
convenience, why not scrap the whole AMD business altogether and bundle
commonjs? With tooling you can resolve modules to address order-sensitivity and
your development and production environments will be much more similar and less
fragile. The CJS syntax is nicer and the ecosystem is exploding because of node
and npm.

パフォーマンスを向上させるためにビルド・ステップを必要とし、利便性の向上のために糖衣構文があるのなら、どうしてAMDの代わりに、CommonJSでバンドルしないのでしょうか?
ツールを使ってモジュールの依存関係を解消し、読み込み順の問題に対応することもできるし、開発とプロダクションの環境を似たものに、そしてより強固にすることも可能です。
CommonJSのシンタックスの方が素敵ですし、Nodeとnpmのおかげでエコシスエムは爆発的に成長しています。

You can seamlessly share code between node and the browser. You just need a
build step and some tooling for source maps and auto-rebuilding.

Nodeとブラウザのコードをシームレスに共有することもできます。ビルド・ステップと、Souce mapsと自動再ビルドに必要ないくつかのツールが必要になるだけです。

Plus, we can use node's module lookup algorithms to save us from version
mismatch insanity so that we can have multiple conflicting versions of different
required packages in the same application and everything will still work. To
save bytes down the wire you can dedupe, which is covered elsewhere in this
document.

さらに、Nodeが持つモジュール・ルックアップのアルゴリズムを利用し、バージョンのミスマッチに心乱すこともなくなるでしょう。同じアプリケーション内で複数の異なるバージョンの衝突し合うパッケージが存在したとしても、問題なく動作するわけです。

# development

Concatenation has some downsides, but these can be very adequately addressed
with development tooling.

ファイルの結合にはいくつかの問題点がありますが、それらは開発ツールを使うことで適切に対処できます。

## source maps

Browserify supports a `--debug`/`-d` flag and `opts.debug` parameter to enable
source maps. Source maps tell the browser to convert line and column offsets for
exceptions thrown in the bundle file back into the offsets and filenames of the
original sources.

BrowserifyでSource Mapsを有効化するには`--debug`または`-d`フラグ、そして`opts.debug`パラメタを利用します。Source Mapsを利用することで結合後のファイルに対して元となるソースのファイル名や例外処理が発生した行番号などを関連づけることができます。

The source maps include all the original file contents inline so that you can
simply put the bundle file on a web server and not need to ensure that all the
original source contents are accessible from the web server with paths set up
correctly.

### exorcist

The downside of inlining all the source files into the inline source map is that
the bundle is twice as large. This is fine for debugging locally but not
practical for shipping source maps to production. However, you can use
[exorcist](https://npmjs.org/package/exorcist) to pull the inline source map out
into a separate `bundle.map.js` file:

``` sh
browserify main.js --debug | exorcist bundle.js.map > bundle.js
```

## auto-recompile

Running a command to recompile your bundle every time can be slow and tedious.
Luckily there are many tools to solve this problem.

毎回バンドルを生成するコマンドを叩くは時間がかかるし、面倒です。しかし、この問題に対する解決を行うツールがいくつもあります。

### [watchify](https://npmjs.org/package/watchify)

You can use `watchify` interchangeably with `browserify` but instead of writing
to an output file once, watchify will write the bundle file and then watch all
of the files in your dependency graph for changes. When you modify a file, the
new bundle file will be written much more quickly than the first time because of
aggressive caching.

`watchify`と`browserify`とは交互に用いることができますが、`watchify`では1度だけファイルを出力するのではなく、バンドルファイルを生成するのに加えて、依存グラフ上にあるすべてのファイルの変更を監視します。それらのファイルを編集すると新しいバンドルファイルが生成されますが、積極的にキャッシュをしているためファイル生成を早く行うことができます。

You can use `-v` to print a message every time a new bundle is written:

`-v`パラメタを渡すと新しいバンドルが生成される度にメッセージを表示できます。

```
$ watchify browser.js -d -o static/bundle.js -v
610598 bytes written to static/bundle.js  0.23s
610606 bytes written to static/bundle.js  0.10s
610597 bytes written to static/bundle.js  0.14s
610606 bytes written to static/bundle.js  0.08s
610597 bytes written to static/bundle.js  0.08s
610597 bytes written to static/bundle.js  0.19s
```

Here is a handy configuration for using watchify and browserify with the
package.json "scripts" field:

次の設定はwatchifyとbrowserifyをpackage.jsonの"scripts"で利用できるようにするものです。

``` json
{
  "build": "browserify browser.js -o static/bundle.js",
  "watch": "watchify browser.js -o static/bundle.js --debug --verbose",
}
```

To build the bundle for production do `npm run build` and to watch files for
during development do `npm run watch`.

プロダクション用にバンドルを生成するのには`npm run build`を利用し、開発中にファイルの状態を監視するには、`npm run watch`を利用します。

[Learn more about `npm run`](http://substack.net/task_automation_with_npm_run).

### [beefy](https://www.npmjs.org/package/beefy)

If you would rather spin up a web server that automatically recompiles your code
when you modify it, check out [beefy](http://didact.us/beefy/).

Just give beefy an entry file:

ファイルの修正をした際に自動的にバンドルのビルドを実行するウェブサーバを必要としているのであれば、[beefy](http://didact.us/beefy/)をチェックして見てください。

単にbeefyに対してファイルを渡してあげればいいだけです。


```
beefy main.js
```

and it will set up shop on an http port.

こうすることで、HTTPポートを開設できます。

### browserify-middleware, enchilada

If you are using express, check out
[browserify-middleware](https://www.npmjs.org/package/browserify-middleware)
or [enchilada](https://www.npmjs.org/package/enchilada).

Expressを利用している場合は、[browserify-middleware](https://www.npmjs.org/package/browserify-middleware)か[enchilada](https://www.npmjs.org/package/enchilada)をチェックしてみてください。

They both provide middleware you can drop into an express application for
serving browserify bundles.

両者はExpressアプリケーションでbrowserifyバンドルを利用するためのミドルウェアです。

## using the api directly

You can just use the API directly from an ordinary `http.createServer()` for
development too:

開発用であれば、`http.createServer()`でAPIを直接利用する方法もあります。

``` js
var browserify = require('browserify');
var http = require('http');

http.createServer(function (req, res) {
    if (req.url === '/bundle.js') {
        res.setHeader('content-type', 'application/javascript');
        var b = browserify(__dirname + '/main.js').bundle();
        b.on('error', console.error);
        b.pipe(res);
    }
    else res.writeHead(404, 'not found')
});
```

## grunt

If you use grunt, you'll probably want to use the
[grunt-browserify](https://www.npmjs.org/package/grunt-browserify) plugin.

Gruntを採用しているのであれば、[grunt-browserify](https://www.npmjs.org/package/grunt-browserify)プラグインを利用できます。

## gulp

If you use gulp, you should use the browserify API directly.

Here is
[a guide for getting started](http://viget.com/extend/gulp-browserify-starter-faq)
with gulp and browserify.

Here is a guide on how to [make browserify builds fast with watchify using
gulp](https://github.com/gulpjs/gulp/blob/master/docs/recipes/fast-browserify-builds-with-watchify.md)
from the official gulp recipes.

Gulpを採用している場合はbrowserifyのAPIを直接利用してください。
Gulpでbrowserifyを利用する場合の[ガイド](http://viget.com/extend/gulp-browserify-starter-faq)はこちらです。

# builtins

In order to make more npm modules originally written for node work in the
browser, browserify provides many browser-specific implementations of node core
libraries:

ブラウザでもNode用に書かれたnpmモジュールを動作させるために、browserifyはいくつかのNodeコアモジュールのブラウザ実装版を提供しています。

* [assert](https://npmjs.org/package/assert)
* [buffer](https://npmjs.org/package/buffer)
* [console](https://npmjs.org/package/console-browserify)
* [constants](https://npmjs.org/package/constants-browserify)
* [crypto](https://npmjs.org/package/crypto-browserify)
* [domain](https://npmjs.org/package/domain-browser)
* [events](https://npmjs.org/package/events)
* [http](https://npmjs.org/package/http-browserify)
* [https](https://npmjs.org/package/https-browserify)
* [os](https://npmjs.org/package/os-browserify)
* [path](https://npmjs.org/package/path-browserify)
* [punycode](https://npmjs.org/package/punycode)
* [querystring](https://npmjs.org/package/querystring)
* [stream](https://npmjs.org/package/stream-browserify)
* [string_decoder](https://npmjs.org/package/string_decoder)
* [timers](https://npmjs.org/package/timers-browserify)
* [tty](https://npmjs.org/package/tty-browserify)
* [url](https://npmjs.org/package/url)
* [util](https://npmjs.org/package/util)
* [vm](https://npmjs.org/package/vm-browserify)
* [zlib](https://npmjs.org/package/browserify-zlib)

events, stream, url, path, and querystring are particularly useful in a browser
environment.

eventsやstream、url、path、そしてquerystringはブラウザ環境でも特に便利なモジュールです。

Additionally, if browserify detects the use of `Buffer`, `process`, `global`,
`__filename`, or `__dirname`, it will include a browser-appropriate definition.

さらに、browserifyは`Buffer`、`process`、`global`、
`__filename`、 または`__dirname`を発見すると、ブラウザ環境において最適な定義をインクルードします。

So even if a module does a lot of buffer and stream operations, it will probably
just work in the browser, so long as it doesn't do any server IO.

モジュールがbufferやstreamの操作を行っていたとしても、サーバーに対して読み込み、書き込みを行っていないのであれば、ブラウザでもおそらく動作するでしょう。

If you haven't done any node before, here are some examples of what each of
those globals can do. Note too that these globals are only actually defined when
you or some module you depend on uses them.

Nodeの経験がない方は以下にいくつかの例を紹介します。これらのコアモジュールも他のモジュールと同様に、自らのモジュールか、依存しているモジュールで利用されている場合にのみ定義されるものであることに注意してください。

## [Buffer](http://nodejs.org/docs/latest/api/buffer.html)

In node all the file and network APIs deal with Buffer chunks. In browserify the
Buffer API is provided by [buffer](https://www.npmjs.org/package/buffer), which
uses augmented typed arrays in a very performant way with fallbacks for old
browsers.

NodeではすべてのファイルとネットワークのAPIはBufferチャンクを扱います。BrowserifyではBuffer APIは[buffer](https://www.npmjs.org/package/buffer)モジュールから供給されています。このモジュールはパフォーマンスの向上と古いブラウザへのフォールバックを行った型付き配列を利用しています。

Here's an example of using `Buffer` to convert a base64 string to hex:

以下は`Buffer`を利用してbase64の文字列をhexに変換する例です。

```
var buf = Buffer('YmVlcCBib29w', 'base64');
var hex = buf.toString('hex');
console.log(hex);
```

This example will print:

例は以下の出力を行います。

```
6265657020626f6f70
```

## [process](http://nodejs.org/docs/latest/api/process.html#process_process)

In node, `process` is a special object that handles information and control for
the running process such as environment, signals, and standard IO streams.

Nodeにおける`process`は、環境や、シグナル、標準入出力などの実行中のプロセスに関する情報とその操作を制御する特別なオブジェクトです。

Of particular consequence is the `process.nextTick()` implementation that
interfaces with the event loop.

`process.nextTick()`の実装におけるイベント・ループを使ったインターフェイスはとりわけ重要です。

In browserify the process implementation is handled by the
[process module](https://www.npmjs.org/package/process) which just provides
`process.nextTick()` and little else.

Browserifyにおける`process`の実装は[process module](https://www.npmjs.org/package/process)で制御しています。現時点では`process.nextTick()`以外の実装はほとんどしていません。

Here's what `process.nextTick()` does:

以下で`process.nextTick()`が何をしているかを例示します。

```
setTimeout(function () {
    console.log('third');
}, 0);

process.nextTick(function () {
    console.log('second');
});

console.log('first');
```

This script will output:

このスクリプトは以下を出力します。

```
first
second
third
```

`process.nextTick(fn)` is like `setTimeout(fn, 0)`, but faster because
`setTimeout` is artificially slower in javascript engines for compatibility reasons.

`process.nextTick(fn)`は`setTimeout(fn, 0)`と同じですが、`setTimeout`は互換性を保つためJavaScriptのエンジン側で人為的に遅延をさせているため、`process.nextTick(fn)`のほうが高速です。

## [global](http://nodejs.org/docs/latest/api/all.html#all_global)

In node, `global` is the top-level scope where global variables are attached
similar to how `window` works in the browser. In browserify, `global` is just an
alias for the `window` object.

Nodeにおける`global`はグローバル変数を格納する最上レベルのスコープとなります。これはブラウザにおける`window`と似ています。Browserifyでは、`global`は`window`オブジェクトのエイリアスです。

## [__filename](http://nodejs.org/docs/latest/api/all.html#all_filename)

`__filename` is the path to the current file, which is different for each file.

`__filename`は現在のファイルのパスを示し、ファイルによって異なります。

To prevent disclosing system path information, this path is rooted at the
`opts.basedir` that you pass to `browserify()`, which defaults to the
[current working directory](https://en.wikipedia.org/wiki/Current_working_directory).

システム全体のパス情報を公開してしまわないように、このパスは`browserify()`に渡す、`opts.basedir`をルートとします。なお、`opts.basedir`はデフォルトでは[カレントディレクトリ](https://ja.wikipedia.org/wiki/%E3%82%AB%E3%83%AC%E3%83%B3%E3%83%88%E3%83%87%E3%82%A3%E3%83%AC%E3%82%AF%E3%83%88%E3%83%AA)となります。

If we have a `main.js`:

`main.js`に以下を記述します。

``` js
var bar = require('./foo/bar.js');

console.log('here in main.js, __filename is:', __filename);
bar();
```

and a `foo/bar.js`:

そして`foo/bar.js`には以下を、

``` js
module.exports = function () {
    console.log('here in foo/bar.js, __filename is:', __filename);
};
```

then running browserify starting at `main.js` gives this output:

それから、browserifyに`main.js`に渡し、実行すると次のような出力になります。

```
$ browserify main.js | node
here in main.js, __filename is: /main.js
here in foo/bar.js, __filename is: /foo/bar.js
```

## [__dirname](http://nodejs.org/docs/latest/api/all.html#all_dirname)

`__dirname` is the directory of the current file. Like `__filename`, `__dirname`
is rooted at the `opts.basedir`.

`__dirname`は現在のファイルが格納されているディレクトリを示します。`__filename`と同様に`opts.basedir`をルートパスとします。

Here's an example of how `__dirname` works:

`__dirname`の動作の例を示します。

main.js:

``` js
require('./x/y/z/abc.js');
console.log('in main.js __dirname=' + __dirname);
```

x/y/z/abc.js:

``` js
console.log('in abc.js, __dirname=' + __dirname);
```

output:

出力:

```
$ browserify main.js | node
in abc.js, __dirname=/x/y/z
in main.js __dirname=/
```

# transforms

Instead of browserify baking in support for everything, it supports a flexible
transform system that are used to convert source files in-place.

browserifyにすべての機能を実装する代わりに、柔軟なtransformシステムをサポートし、ソースファイルをインプレースで変形します。

This way you can `require()` files written in coffee script or templates and
everything will be compiled down to javascript.

こうすることでCoffeeScriptで書かれたファイルやテンプレートを`require()`しても、最終的にJavaScriptにコンパイルすることができます。

To use [coffeescript](http://coffeescript.org/) for example, you can use the
[coffeeify](https://www.npmjs.org/package/coffeeify) transform.
Make sure you've installed coffeeify first with `npm install coffeeify` then do:

[CoffeeScript](http://coffeescript.org/)の場合には
[coffeeify](https://www.npmjs.org/package/coffeeify)が利用できます。`npm install coffeeify`としてcoffeeifyをインストールしてから、以下の様に実行します。

```
$ browserify -t coffeeify main.coffee > bundle.js
```

or with the API you can do:

APIの場合は以下の様になります。

```
var b = browserify('main.coffee');
b.transform('coffeeify');
```

The best part is, if you have source maps enabled with `--debug` or
`opts.debug`, the bundle.js will map exceptions back into the original coffee
script source files. This is very handy for debugging with firebug or chrome
inspector.

もっとも便利な点は、`--debug`オプションか`opts.debug`を通じてsource mapsを利用可能にしている場合、bundle.jsで発生した例外処理などは元々のCoffeeScriptにマップされます。FirebugやChromeの開発者ツールなどを使ってのデバッグに非常に便利です。

## writing your own

Transforms implement a simple streaming interface. Here is a transform that
replaces `$CWD` with the `process.cwd()`:

Transformは単純なストリーミングのインターフェイスを通じて実装できます。以下は`$CWD`を`process.cwd()`に変換する例です。

``` js
var through = require('through2');

module.exports = function (file) {
    return through(function (buf, enc, next) {
        this.push(buf.toString('utf8').replace(/\$CWD/g, process.cwd());
        next();
    });
};
```

The transform function fires for every `file` in the current package and returns
a transform stream that performs the conversion. The stream is written to and by
browserify with the original file contents and browserify reads from the stream
to obtain the new contents.

Transform関数はパッケージ内すべての`file`に対して実行され、変換を行うTransformストリームを返します。そのストリームはbrowserifyにオリジナルのファイルの内容とともに書き込まれ、browserifyが変換後の新しいコンテンツを取得します。

Simply save your transform to a file or make a package and then add it with
`-t ./your_transform.js`.

作成したTransformは単純にファイルとして保存するか、パッケージ化し、`-t ./your_transform.js`のように利用することができます。

For more information about how streams work, check out the
[stream handbook](https://github.com/substack/stream-handbook).

ストリームがどのように動作するかについて詳しくは[stream handbook](https://github.com/substack/stream-handbook)を参照してください。

# package.json

## browser field

You can define a `"browser"` field in the package.json of any package that will
tell browserify to override lookups for the main field and for individual
modules.

package.jsonに`"browser"`という領域を定義することで、browserifyに対して、個別のモジュール、またはメインの領域にある指定を上書きすることができます。

If you have a module with a main entry point of `main.js` for node but have a
browser-specific entry point at `browser.js`, you can do:

例えば、`main.js`というファイルをモジュールのメインエントリポイントとしている場合に、ブラウザ用のエントリポイントとして`browser.js`を指定する場合には以下の様にすることができます。

``` json
{
  "name": "mypkg",
  "version": "1.2.3",
  "main": "main.js",
  "browser": "browser.js"
}
```

Now when somebody does `require('mypkg')` in node, they will get the exports
from `main.js`, but when they do `require('mypkg')` in a browser, they will get
the exports from `browser.js`.

こうすることで、`require('mypkg')`とnodeで行うと、`main.js`からexportsが得られ、ブラウザで`require('mypkg')`とすると、`browser.js`を得ることになります。

Splitting up whether you are in the browser or not with a `"browser"` field in
this way is greatly preferrable to checking whether you are in a browser at
runtime because you may want to load different modules based on whether you are
in node or the browser. If the `require()` calls for both node and the browser
are in the same file, browserify's static analysis will include everything
whether you use those files or not.

`"browser"`領域を使ってブラウザ用のモジュールであるかを判定する方法は、nodeとブラウザで異なるモジュールを呼び出すためにも非常におすすめの方法です。

`require()`の実行がnodeとブラウザで同じファイルを参照する場合、browserifyの静的解析はそれらのファイルを使うかどうかに関わらずすべてのファイルに対して実行されます。

You can do more with the "browser" field as an object instead of a string.

For example, if you only want to swap out a single file in `lib/` with a
browser-specific version, you could do:

`"browser"`領域では文字列以外にオブジェクトを扱うこともできます。

``` json
{
  "name": "mypkg",
  "version": "1.2.3",
  "main": "main.js",
  "browser": {
    "lib/foo.js": "lib/browser-foo.js"
  }
}
```

or if you want to swap out a module used locally in the package, you can do:

`lib/`ディレクトリ内のある1ファイルだけをブラウザ専用として扱い対場合には以下のようにします。

``` json
{
  "name": "mypkg",
  "version": "1.2.3",
  "main": "main.js",
  "browser": {
    "fs": "level-fs-browser"
  }
}
```

You can ignore files (setting their contents to the empty object) by setting
their values in the browser field to `false`:

browser領域で`false`を指定することでファイルを無視(コンテンツを空のオブジェクトにする)することもできます。

``` json
{
  "name": "mypkg",
  "version": "1.2.3",
  "main": "main.js",
  "browser": {
    "winston": false
  }
}
```

The browser field *only* applies to the current package. Any mappings you put
will not propagate down to its dependencies or up to its dependents. This
isolation is designed to protect modules from each other so that when you
require a module you won't need to worry about any system-wide effects it might
have. Likewise, you shouldn't need to wory about how your local configuration
might adversely affect modules far away deep into your dependency graph.

browser領域はパッケージ *のみ* に適用されます。そのパッケージに依存しているパッケージや、そのパッケージが依存しているパッケージにマッピングが伝播することはありません。こうすることでシステム全体に及ぶような影響からモジュールを保護するようにしています。同様にこのローカルな設定が依存関係グラフの奥深くまで悪い影響をおよぼすようなことを心配しなくてもよくなります。

## browserify.transform field

You can configure transforms to be automatically applied when a module is loaded
in a package's `browserify.transform` field. For example, we can automatically
apply the [brfs](https://npmjs.org/package/brfs) transform with this
package.json:

`browserify.transform`領域ではモジュールが呼び出される際に自動的にトランスフォームを設定できます。以下のpackage.jsonの例では[brfs](https://npmjs.org/package/brfs)トランスフォームを自動的に実行できます。

``` json
{
  "name": "mypkg",
  "version": "1.2.3",
  "main": "main.js",
  "browserify": {
    "transform": [ "brfs" ]
  }
}
```

Now in our `main.js` we can do:

こうすることで`main.js`で以下のようにできます。

``` js
var fs = require('fs');
var src = fs.readFileSync(__dirname + '/foo.txt', 'utf8');

module.exports = function (x) { return src.replace(x, 'zzz') };
```

and the `fs.readFileSync()` call will be inlined by brfs without consumers of
the module having to know. You can apply as many transforms as you like in the
transform array and they will be applied in order.

モジュールのユーザは`fs.readFileSync()`はbrfsによってインライン化されることを知る必要がありません。この配列内でいくつでもトランスフォームを利用することができ、配列の順番で処理されます。

Like the `"browser"` field, transforms configured in package.json will only
apply to the local package for the same reasons.

`"browser"`領域と同じ理由でpackage.json内のtransforms領域での設定はローカルパッケージ内にのみ適用されます。

### configuring transforms

Sometimes a transform takes configuration options on the command line. To apply these
from package.json you can do the following.

トランスフォームは時としてコマンドラインで設定オプションを利用しますが、package.jsonから設定するには以下のようにします。

**on the command line**
**コマンドラインの例**

```
browserify -t coffeeify \
	   -t [ browserify-ngannotate --ext .coffee ] \
	   index.coffee > index.js
```

**in package.json**
**package.jsonの例**
``` json
"browserify": {
  "transform": [
    "coffeeify",
    ["browserify-ngannotate", {"ext": ".coffee"}]
  ]
}
```


# finding good modules

Here are [some useful heuristics](http://substack.net/finding_modules)
for finding good modules on npm that work in the browser:

ブラウザで動作するnpmにあるモジュールを見つけ出すための[役に立ついくつかのヒューリスティック](http://substack.net/finding_modules)を紹介します。

* I can install it with npm

* npmでインストールすることができること

* code snippet on the readme using require() - from a quick glance I should see
how to integrate the library into what I'm presently working on

* READMEにあるコード例でrequire()が利用されていること。(ぱっと見で自分自身が開発しているコードにそのライブラリをどうやって連携されるかがわかる)

* has a very clear, narrow idea about scope and purpose

* 明確で、制限されたスコープと目的を持っていること

* knows when to delegate to other libraries - doesn't try to do too many things itself

* 他のライブラリに委譲するタイミングを見極めていること。（そのライブラリ自身で多くのことをやろうとしていないこと)

* written or maintained by authors whose opinions about software scope,
modularity, and interfaces I generally agree with (often a faster shortcut
than reading the code/docs very closely)

* ソフトウェアのスコープ、モジュラー性、インターフェイスに関する意見について私自身が大枠同意できる作者またはメインテナーであること。(コードやドキュメントをきちんと読み込むより大抵はこちらの方が早い)

* inspecting which modules depend on the library I'm evaluating - this is baked
into the package page for modules published to npm

* 評価中のライブラリがほかのどのモジュールから依存されているかを調べる。npmで公開されたパッケージのページで確認できる。

Other metrics like number of stars on github, project activity, or a slick
landing page, are not as reliable.

GitHubのスターの数や、プロジェクトの活動、オシャレなランディングページの有無はそれほど信頼できるデータではない。

## module philosophy

People used to think that exporting a bunch of handy utility-style things would
be the main way that programmers would consume code because that is the primary
way of exporting and importing code on most other platforms and indeed still
persists even on npm.

プログラマにコードを利用してもらうには、たくさんの便利なユーティリティのようなパッケージを公開することが主要な方法だと考えています。そうすることが他のプラットフォームでも、もちろんnpmでもコードを公開したり、利用したりする方法だからです。

However, this
[kitchen-sink mentality](https://github.com/substack/node-mkdirp/issues/17)
toward including a bunch of thematically-related but separable functionality
into a single package appears to be an artifact for the difficulty of
publishing and discovery in a pre-github, pre-npm era.

しかし、このようなテーマ的には関連していても、分割できる機能を1つのパッケージに[詰め込もうとする精神](https://github.com/substack/node-mkdirp/issues/17)はGitHub以前、npm以前のパッケージの公開や発見が難しかった時代の遺物です。

There are two other big problems with modules that try to export a bunch of
functionality all in one place under the auspices of convenience: demarcation
turf wars and finding which modules do what.

きっと便利になるであろうと思って、すべての機能を寄せ集めているモジュールには他に2つの大きな問題があります。  
縄張り争いとどのモジュールが何をするのかを見つけ出すことです。

Packages that are grab-bags of features
[waste a ton of time policing boundaries](https://github.com/jashkenas/underscore/search?q=%22special-case%22&ref=cmdform&type=Issues)
about which new features belong and don't belong.
There is no clear natural boundary of the problem domain in this kind of package
about what the scope is, it's all
[somebody's smug opinion](http://david.heinemeierhansson.com/2012/rails-is-omakase.html).

機能の寄せ集めパッケージは多くの無駄な時間を新しい機能がそのパッケージの属すべきか否かの[境界線の警備](https://github.com/jashkenas/underscore/search?q=%22special-case%22&ref=cmdform&type=Issues)に費やします。  
このようなパッケージにおけるスコープには明確で当然な問題領域の境界線はなく、いつだって[誰かの独りよがりな意見](http://david.heinemeierhansson.com/2012/rails-is-omakase.html)でしかありません。

Node, npm, and browserify are not that. They are avowedly ala-carte,
participatory, and would rather celebrate disagreement and the dizzying
proliferation of new ideas and approaches than try to clamp down in the name of
conformity, standards, or "best practices".

Node、npm、そしてbrowserifyはそうではありません。心地いい、標準、または『ベストプラクティス』の名の元で取り締まるのではなく、アラカルト的で、参加型であり、意見の不一致をたたえ、そして目眩がするほどの新しいアイデアやアプローチをたたえます。

Nobody who needs to do gaussian blur ever thinks "hmm I guess I'll start checking
generic mathematics, statistics, image processing, and utility libraries to see
which one has gaussian blur in it. Was it stats2 or image-pack-utils or
maths-extra or maybe underscore has that one?"
No. None of this. Stop it. They `npm search gaussian` and they immediately see
[ndarray-gaussian-filter](https://npmjs.org/package/ndarray-gaussian-filter) and
it does exactly what they want and then they continue on with their actual
problem instead of getting lost in the weeds of somebody's neglected grand
utility fiefdom.

グラシアン・ブラーを必要としている誰もが、「あー、まずは一般数学か、統計か、画像処理かユーティリティライブラリの中からグラシアン・ブラーがあるか確認しよう。あれ、stats2だったか。画像便利ユーティティだったか、math-extraにあったか、もしかしてunderscoreに入っていたっけ?」というようなことを考える必要はないはずです。  
もう、そんなことは嫌ですよね?  
`npm search gaussian`とすれば、[ndarray-gaussian-filter](https://npmjs.org/package/ndarray-gaussian-filter)をすぐ見つけ出すことができ、このパッケージは丁度やりたいことを実装しています。誰かが放置した贅沢なユーティリティ国の草むらで迷うのでは無く、実際の問題の解決へ向かうことができるわけです。

# organizing modules

## avoiding ../../../../../../..

Not everything in an application properly belongs on the public npm and the
overhead of setting up a private npm or git repo is still rather large in many
cases. Here are some approaches for avoiding the `../../../../../../../`
relative paths problem.

アプリケーション内の全てのモジュールをnpmで公開できるわけではありませんし、プライベートなnpmやgitレポジトリを用意するのは多くの場合はまだ面倒です。以下に`../../../../../../../`というような相対パスの問題を回避するアプローチを紹介します。

### node_modules

People sometimes object to putting application-specific modules into
node_modules because it is not obvious how to check in your internal modules
without also checking in third-party modules from npm.

第三者製のモジュールと内製モジュールを分離して管理することができないため、アプリケーション専用モジュールを`node_modules`ディレクトリに格納することに反対する人もいます。

The answer is quite simple! If you have a `.gitignore` file that ignores
`node_modules`:

それに関する答えは簡単です。`.gitignore`ファイルで`node_modules`を無視する設定をしている場合には、

```
node_modules
```

You can just add an exception with `!` for each of your internal application
modules:

内製したアプリケーション専用モジュールに対して、`!`を追加するだけで、例外処理を行えます。

```
node_modules/*
!node_modules/foo
!node_modules/bar
```

Please note that you can't *unignore* a subdirectory,
if the parent is already ignored. So instead of ignoring `node_modules`,
you have to ignore every directory *inside* `node_modules` with the 
`node_modules/*` trick, and then you can add your exceptions.

注意しなければならないことは、親ディレクトリが無視されている場合に、サブ・ディレクトリを *無視しない* 設定にすることはできません。  
`node_modules`ディレクトリの *中の* ディレクトリすべてを`node_modules/*`というように無視する必要があります。こうすることで、例外処理を行えます。

Now anywhere in your application you will be able to `require('foo')` or
`require('bar')` without having a very large and fragile relative path.

こうすることでアプリケーション内にて、巨大でかつ脆弱な相対パスを指定することなく、`require('foo')`や、`require('bar')`というようにモジュールを呼び出せます。

If you have a lot of modules and want to keep them more separate from the
third-party modules installed by npm, you can just put them all under a
directory in `node_modules` such as `node_modules/app`:

たくさんのモジュールがあって、npmでインストールした第三者製のモジュールとの区別をより明確にしたい場合は、すべての内製モジュールを`node_modules`内のある1つのディレクトリ、例えば、`node_modules/app`のような場所に格納することもできます。

```
node_modules/app/foo
node_modules/app/bar
```

Now you will be able to `require('app/foo')` or `require('app/bar')` from
anywhere in your application.

この場合には、アプリケーションからは`require('app/foo')`、`require('app/bar')`というように呼び出せます。

In your `.gitignore`, just add an exception for `node_modules/app`:

`.gitignore`ファイルには、`node_modules/app`に対してのみ例外を記述すればよいことになります。

```
node_modules/*
!node_modules/app
```

If your application had transforms configured in package.json, you'll need to
create a separate package.json with its own transform field in your
`node_modules/foo` or `node_modules/app/foo` component directory because
transforms don't apply across module boundaries. This will make your modules
more robust against configuration changes in your application and it will be
easier to independently reuse the packages outside of your application.

transformの設定をpackage.jsonで記述している場合、`node_modules/foo`や`node_modules/app/foo`にあるディレクトリ内にある別のpackage.json内に記述する必要があります。    
アプリケーション内での設定の変更に対して堅牢で、アプリケーション外でパッケージを個別で再利用することも簡単にできるため、transformsはモジュールを超えて適用できないからです。

### symlink

Another handy trick if you are working on an application where you can make
symlinks and don't need to support windows is to symlink a `lib/` or `app/`
folder into `node_modules`. From the project root, do:

もう1つ便利なトリックとして、シンボリック・リンクが使えて、Windowsをサポートする必要がない場合には、`lib/`やapp/`から、`node_modules`にシンボリック・リンクを作る方法もあります。  
プロジェクトのルートから、以下のように実行します。

```
ln -s ../lib node_modules/app
```

and now from anywhere in your project you'll be able to require files in `lib/`
by doing `require('app/foo.js')` to get `lib/foo.js`.

こうすることで、プロジェクト内のどこからでも`lib/`内にあるファイルに対して、`require('app/foo.js')`とすることができます。

### custom paths

You might see some places talk about using the `$NODE_PATH` environment variable
or `opts.paths` to add directories for node and browserify to look in to find
modules.

皆さんもこれまでに環境変数である`$NODE_PATH`や`opts.paths`を使って、nodeやbrowserifyのモジュールを格納するディレクトリを追加する例を見たことがあることでしょう。

Unlike most other platforms, using a shell-style array of path directories with
`$NODE_PATH` is not as favorable in node compared to making effective use of the
`node_modules` directory.

ほかのプラットフォームとは異なり、`$NODE_PATH`に対してパスディレクトリをシェルのスタイルの配列を使って追加することは、nodeにおいては`node_modules`を効果的に使うことに比べて、好まれてはいません。

This is because your application is more tightly coupled to a runtime
environment configuration so there are more moving parts and your application
will only work when your environment is setup correctly.

こうすることでアプリケーションがより不確定要素の多い、実行環境の設定対して強い依存を持つことになってしまい、環境を正確に用意できないとアプリケーションが動作しなくなってしまうからです。

node and browserify both support but discourage the use of `$NODE_PATH`.

nodeもbrowserifyも`$NODE_PATH`をサポートはしていますが、利用については推奨していません。

## non-javascript assets

There are many
[browserify transforms](https://github.com/substack/node-browserify/wiki/list-of-transforms)
you can use to do many things. Commonly, transforms are used to include
non-javascript assets into bundle files.

非常に多くのことができる[browserifyのトランスフォーム](https://github.com/substack/node-browserify/wiki/list-of-transforms)が公開されています。中にはJavaScriptではないアセットをバンドル化するトランスフォームも存在しています。

### brfs

One way of including any kind of asset that works in both node and the browser
is brfs.

Nodeでもブラウザでも動作する様々なアセットをバンドル化する方法としてbrfsが上げられます。

brfs uses static analysis to compile the results of `fs.readFile()` and
`fs.readFileSync()` calls down to source contents at compile time.

brfsは静的解析を行い`fs.readFile()`と`fs.readFileSync()`の実行結果をコンパイルすることができます。

For example, this `main.js`:

例えば、以下の`main.js`では、

``` js
var fs = require('fs');
var html = fs.readFileSync(__dirname + '/robot.html', 'utf8');
console.log(html);
```

applied through brfs would become something like:

brfsを適用すると以下のように処理されます。

``` js
var fs = require('fs');
var html = "<b>beep boop</b>";
console.log(html);
```

when run through brfs.

This is handy because you can reuse the exact same code in node and the browser,
which makes sharing modules and testing much simpler.

Nodeとブラウザ間でまったく同じコードを再利用できるので非常に便利ですし、モジュールの共有やテストをよりシンプルにできます。

`fs.readFile()` and `fs.readFileSync()` accept the same arguments as in node,
which makes including inline image assets as base64-encoded strings very easy:

Nodeでは、`fs.readFile()`と`fs.readFileSync()`は同じ引数を利用できるため、base64でエンコードされた文字列として画像のアセットをバンドル化することも簡単に行えます。

``` js
var fs = require('fs');
var imdata = fs.readFileSync(__dirname + '/image.png', 'base64');
var img = document.createElement('img');
img.setAttribute('src', 'data:image/png;base64,' + imdata);
document.body.appendChild(img);
```

If you have some css you want to inline into your bundle, you can do that too
with the assistance of a module such as
[insert-css](https://npmjs.org/package/insert-css):

[insert-css](https://npmjs.org/package/insert-css)のようなモジュールを利用すれば、CSSをバンドル化することもできます。

``` js
var fs = require('fs');
var insertStyle = require('insert-css');

var css = fs.readFileSync(__dirname + '/style.css', 'utf8');
insertStyle(css);
```

Inserting css this way works fine for small reusable modules that you distribute
with npm because they are fully-contained, but if you want a more wholistic
approach to asset management using browserify, check out
[atomify](https://www.npmjs.org/package/atomify) and
[parcelify](https://www.npmjs.org/package/parcelify).

npmで配布する小さな再利用可能なモジュールとしてCSSを上記のように挿入することもできますが、browserifyを利用してより総体的なアセット管理をする場合には、[atomify](https://www.npmjs.org/package/atomify)や[parcelify](https://www.npmjs.org/package/parcelify)を試してみるといいでしょう。

### hbsify

### jadeify

### reactify

## reusable components

Putting these ideas about code organization together, we can build a reusable UI
component that we can reuse across our application or in other applications.

ここまで紹介してきたコードの統合に関する様々なアイデアを活用して、開発中のアプリケーション内やほかのアプリケーションでも再利用可能なUIコンポーネントを生み出すことができます。

Here is a bare-bones example of an empty widget module:

ベア・ボーンな例として空のウィジェットモジュールを作成します。

``` js
module.exports = Widget;

function Widget (opts) {
    if (!(this instanceof Widget)) return new Widget(opts);
    this.element = document.createElement('div');
}

Widget.prototype.appendTo = function (target) {
    if (typeof target === 'string') target = document.querySelector(target);
    target.appendChild(this.element);
};
```

Handy javascript constructor tip: you can include a `this instanceof Widget`
check like above to let people consume your module with `new Widget` or
`Widget()`. It's nice because it hides an implementation detail from your API
and you still get the performance benefits and indentation wins of using
prototypes.

JavaScriptのコンストラクタに関するコツ: 上記のように`this instanceof Widget`とすることで、`new Widget`または`Widget()`というようにモジュールを利用することができます。実装に関する詳細をAPIから隠すことができる上、prototypeを利用したパフォーマンスの優位性を活用することもできます。

To use this widget, just use `require()` to load the widget file, instantiate
it, and then call `.appendTo()` with a css selector string or a dom element.

先ほどのウィジェットを使うには、`require()`を使ってウィジェット・ファイルを呼びだし、初期化し、CSSセレクタの文字列か、DOM要素で`.appendTo()`を実行すればいいだけです。

Like this:

以下がその例です。

``` js
var Widget = require('./widget.js');
var w = Widget();
w.appendTo('#container');
```

and now your widget will be appended to the DOM.

これでウィジェットはDOMに追加されます。

Creating HTML elements procedurally is fine for very simple content but gets
very verbose and unclear for anything bigger. Luckily there are many transforms
available to ease importing HTML into your javascript modules.

HTML要素を手続き的に作っていくことはシンプルなコンテンツの場合には問題ないでしょう。しかし、ほんの少し大きなシステムでも、冗長で不明瞭になってしまうプロセスです。運がいいことにHTMLをJavaScriptモジュール内にインポートするトランスフォームはたくさん公開されています。

Let's extend our widget example using [brfs](https://npmjs.org/package/brfs). We
can also use [domify](https://npmjs.org/package/domify) to turn the string that
`fs.readFileSync()` returns into an html dom element:

先ほどのウィジェットの例を[brfs](https://npmjs.org/package/brfs)を使って拡張してみましょう。それに加えて、[domify](https://npmjs.org/package/domify)を使って、`fs.readFileSync()`から戻ってくる文字列をHTMLのDOM要素として利用できるようにします。

``` js
var fs = require('fs');
var domify = require('domify');

var html = fs.readFileSync(__dirname + '/widget.html', 'utf8');

module.exports = Widget;

function Widget (opts) {
    if (!(this instanceof Widget)) return new Widget(opts);
    this.element = domify(html);
}

Widget.prototype.appendTo = function (target) {
    if (typeof target === 'string') target = document.querySelector(target);
    target.appendChild(this.element);
};
```

and now our widget will load a `widget.html`, so let's make one:

これでウィジェットは`widget.html`を呼び出すことになったので、早速作成します。

``` html
<div class="widget">
  <h1 class="name"></h1>
  <div class="msg"></div>
</div>
```

It's often useful to emit events. Here's how we can emit events using the
built-in `events` module and the [inherits](https://npmjs.org/package/inherits)
module:

大抵の場合はイベントを発火すると便利です。以下にNode標準モジュールである`events`と[inherits](https://npmjs.org/package/inherits)モジュールを使って実装する例を示します。

``` js
var fs = require('fs');
var domify = require('domify');
var inherits = require('inherits');
var EventEmitter = require('events').EventEmitter;

var html = fs.readFileSync(__dirname + '/widget.html', 'utf8');

inherits(Widget, EventEmitter);
module.exports = Widget;

function Widget (opts) {
    if (!(this instanceof Widget)) return new Widget(opts);
    this.element = domify(html);
}

Widget.prototype.appendTo = function (target) {
    if (typeof target === 'string') target = document.querySelector(target);
    target.appendChild(this.element);
    this.emit('append', target);
};
```

Now we can listen for `'append'` events on our widget instance:

こうするとウィジェット・インスタンスで`'append'`イベントの発火を監視することができます。

``` js
var Widget = require('./widget.js');
var w = Widget();
w.on('append', function (target) {
    console.log('appended to: ' + target.outerHTML);
});
w.appendTo('#container');
```

We can add more methods to our widget to set elements on the html:

ウィジェットのHTMLに対して要素を付与するメソッドを追加します。

``` js
var fs = require('fs');
var domify = require('domify');
var inherits = require('inherits');
var EventEmitter = require('events').EventEmitter;

var html = fs.readFileSync(__dirname + '/widget.html', 'utf8');

inherits(Widget, EventEmitter);
module.exports = Widget;

function Widget (opts) {
    if (!(this instanceof Widget)) return new Widget(opts);
    this.element = domify(html);
}

Widget.prototype.appendTo = function (target) {
    if (typeof target === 'string') target = document.querySelector(target);
    target.appendChild(this.element);
};

Widget.prototype.setName = function (name) {
    this.element.querySelector('.name').textContent = name;
}

Widget.prototype.setMessage = function (msg) {
    this.element.querySelector('.msg').textContent = msg;
}
```

If setting element attributes and content gets too verbose, check out
[hyperglue](https://npmjs.org/package/hyperglue).

要素の属性やコンテンツが冗長になってきたら、[hyperglue](https://npmjs.org/package/hyperglue)を試してみてください。

Now finally, we can toss our `widget.js` and `widget.html` into
`node_modules/app-widget`. Since our widget uses the
[brfs](https://npmjs.org/package/brfs) transform, we can create a `package.json`
with:

最後に`widget.js`と`widget.html`を`node_modules/app-widget`に格納します。このウィジェットは[brfs](https://npmjs.org/package/brfs)トランスフォームを利用するので、`package.json`は以下のようになります。

``` json
{
  "name": "app-widget",
  "version": "1.0.0",
  "private": true,
  "main": "widget.js",
  "browserify": {
    "transform": [ "brfs" ]
  },
  "dependencies": {
    "brfs": "^1.1.1",
    "inherits": "^2.0.1"
  }
}
```

And now whenever we `require('app-widget')` from anywhere in our application,
brfs will be applied to our `widget.js` automatically!
Our widget can even maintain its own dependencies. This way we can update
dependencies in one widgets without worrying about breaking changes cascading
over into other widgets.

これでアプリケーション内のどこで`require('app-widget')`が実行されようと、brfsは自動で`widget.js`に適用されます!  
このウィジェットはもちろん自身の依存を管理できます。こうすることで、ある1つのウィジェット内の依存をアップデートしても、ほかのウィジェットに影響を与える心配をしなくてもすむようになります。

Make sure to add an exclusion in your `.gitignore` for
`node_modules/app-widget`:

`node_modules/app-widget`を`.gitignore`を使って除外するのも忘れないでください。

```
node_modules/*
!node_modules/app-widget
```

You can read more about [shared rendering in node and the
browser](http://substack.net/shared_rendering_in_node_and_the_browser) if you
want to learn about sharing rendering logic between node and the browser using
browserify and some streaming html libraries.

レンダリング・ロジックをbrowserifyといくつかのHTMLストリーミングライブラリを使って、Nodeとブラウザ間で共有する方法について詳しく知りたい場合は、[shared rendering in node and the browser](http://substack.net/shared_rendering_in_node_and_the_browser)の記事を参考にしてください。

# testing in node and the browser

Testing modular code is very easy! One of the biggest benefits of modularity is
that your interfaces become much easier to instantiate in isolation and so it's
easy to make automated tests.

モジュール化したコードをテストをするのは簡単です!モジュール化の最大の効用はインターフェースを初期化する際に独立した形で行うことができるため、自動テストを作成するのが容易になることです。

Unfortunately, few testing libraries play nicely out of the box with modules and
tend to roll their own idiosyncratic interfaces with implicit globals and obtuse
flow control that get in the way of a clean design with good separation.

残念なことに、モジュール化したコードをテストするのに簡単なテストライブラリは多くはありません。それに加えて、多くは暗黙のグローバルを使った独特なインターフェースを用意している上、分離された整然としたデザインの邪魔になるようなフロー制御を行ってしまっています。

People also make a huge fuss about "mocking" but it's usually not necessary if
you design your modules with testing in mind. Keeping IO separate from your
algorithms, carefully restricting the scope of your module, and accepting
callback parameters for different interfaces can all make your code much easier
to test.

多くの人が"モック"に対して大きな関心を寄せていますが、モジュール化を念頭にコードをデザインする場合には必要ない場合が多い。IOとアルゴリズムと分化し、モジュールのスコープを注意深く制限し、そして、異なるインターフェース用のコールバックパラメータを受け入れることで容易にテストするできるようになります。

For example, if you have a library that does both IO and speaks a protocol,
[consider separating the IO layer from the
protocol](https://www.youtube.com/watch?v=g5ewQEuXjsQ#t=12m30)
using an interface like [streams](https://github.com/substack/stream-handbook).

IOとプロトコルへのアクセスのあるライブラリがある場合、[Streams](https://github.com/substack/stream-handbook)インターフェースなどを利用して、[IOレイヤとプロトコルを分離することを検討してください](https://www.youtube.com/watch?v=g5ewQEuXjsQ#t=12m30)。

Your code will be easier to test and reusable in different contexts that you
didn't initially envision. This is a recurring theme of testing: if your code is
hard to test, it is probably not modular enough or contains the wrong balance of
abstractions. Testing should not be an afterthought, it should inform your
whole design and it will help you to write better interfaces.

コードはテストが容易になり、元々想定していたよりも多くのコンテキストで再利用できるようになります。テストに関する繰り返し発生するテーマとして、もしコードがテストしづらい場合は、そのコードのモジュール化は十分でないか、異なるバランスの抽象化が含まれている。というものがあります。  
テストを行うことは後付けではなく、全体のデザインについて情報を提供するものであり、より優れたインターフェースを書くための助けとなるべきです。

## testing libraries

### [tape](https://npmjs.org/package/tape)

Tape was specifically designed from the start to work well in both node and
browserify. Suppose we have an `index.js` with an async interface:

TapeはNodeとBrowserifyの両方で利用するように始めから作成されたライブラリです。以下の様な非同期インターフェースを持った`index.js`があるとします。

``` js
module.exports = function (x, cb) {
    setTimeout(function () {
        cb(x * 100);
    }, 1000);
};
```

Here's how we can test this module using [tape](https://npmjs.org/package/tape). 
Let's put this file in `test/beep.js`:

このモジュールを[tape](https://npmjs.org/package/tape)を使ってテストしてみましょう。以下を`test/beep.js`に記述します。

``` js
var test = require('tape');
var hundreder = require('../');

test('beep', function (t) {
    t.plan(1);
    
    hundreder(5, function (n) {
        t.equal(n, 500, '5*100 === 500');
    });
});
```

Because the test file lives in `test/`, we can require the `index.js` in the
parent directory by doing `require('../')`. `index.js` is the default place that
node and browserify look for a module if there is no package.json in that
directory with a `main` field.

`test/`ディレクトリ内にテストファイルが設置されているため、`require('../')`の記述で、親ディレクトリにある`index.js`を呼び出します。NodeとBrowserifyは`package.json`内に`main`フィールドで設定がない限り、`index.js`というファイル名をデフォルト値として扱います。

We can `require()` tape like any other library after it has been installed with
`npm install tape`.

`npm install tape`でインストールを行えば、ほかのライブラリと同様に、tapeを`require()`することができます。

The string `'beep'` is an optional name for the test.
The 3rd argument to `t.equal()` is a completely optional description.

例にある`'beep'`という文字列はテストの名称です。これは必須ではありません。`t.equal()`の第3引数も同様に必須ではありません。

The `t.plan(1)` says that we expect 1 assertion. If there are not enough
assertions or too many, the test will fail. An assertion is a comparison
like `t.equal()`. tape has assertion primitives for:

`t.plan(1)`は1つのアサーションがある事を示しています。アサーションは指定より多くても少なくてもテストは失敗します。アサーションとは`t.equal()`のような比較を指します。Tapeには以下のような基本アサーションがあります。

* t.equal(a, b) - compare a and b strictly with `===`
* t.deepEqual(a, b) - compare a and b recursively
* t.ok(x) - fail if `x` is not truthy

* t.equal(a, b) - aとbを`===`使って厳密に比較します
* t.deepEqual(a, b) - aとbを再帰的に比較します
* t.ok(x) - `x`が'truthy'でない場合に失敗します

and more! You can always add an additional description argument.

他にもまだアサーションはあります。また全てのアサーションに対して説明文を追加する引数は利用できます。

Running our module is very simple! To run the module in node, just run
`node test/beep.js`:

モジュールを実行するのはとても簡単です。`node test/beep.js`として、Node内でモジュールを実行できます。

```
$ node test/beep.js
TAP version 13
# beep
ok 1 5*100 === 500

1..1
# tests 1
# pass  1

# ok
```

The output is printed to stdout and the exit code is 0.

To run our code in the browser, just do:

テスト結果はstdoutとして表示され、0を終了ステータスとします。 

ブラウザでこのコードを実行するには、以下のようにするだけです。

```
$ browserify test/beep.js > bundle.js
```

then plop `bundle.js` into a `<script>` tag:

そして`bundle.js`を`<script>`タグで呼び出します。

```
<script src="bundle.js"></script>
```

and load that html in a browser. The output will be in the debug console which
you can open with F12, ctrl-shift-j, or ctrl-shift-k depending on the browser.

そしてHTMLをブラウザで呼び出します。テスト結果はデバッグ・コンソールに表示されます。ブラウザによって異なりますが、F12か、ctrl-shift-j、またはctrl-shift-kなどのショートカットキーで呼び出します。

This is a bit cumbersome to run our tests in a browser, but you can install the
`testling` command to help. First do:

ブラウザでテストを実行するのはやや面倒ですが、`testling`を利用すると楽になります。まずはインストールします。

```
npm install -g testling
```

And now just do `browserify test/beep.js | testling`:

それから、`browserify test/beep.js | testling`とするだけです。

```
$ browserify test/beep.js | testling

TAP version 13
# beep
ok 1 5*100 === 500

1..1
# tests 1
# pass  1

# ok
```

`testling` will launch a real browser headlessly on your system to run the tests.

`testling`は本物のブラウザをヘッドレスで立ち上げ、テストを実行します。

Now suppose we want to add another file, `test/boop.js`:

`test/boop.js`という別のファイルを追加したとします。

``` js
var test = require('tape');
var hundreder = require('../');

test('fraction', function (t) {
    t.plan(1);

    hundreder(1/20, function (n) {
        t.equal(n, 5, '1/20th of 100');
    });
});

test('negative', function (t) {
    t.plan(1);

    hundreder(-3, function (n) {
        t.equal(n, -300, 'negative number');
    });
});
```

Here our test has 2 `test()` blocks. The second test block won't start to
execute until the first is completely finished, even though it is asynchronous.
You can even nest test blocks by using `t.test()`.

このテストには2つの`test()`ブロックがあります。非同期であっても2つ目のテストブロックは1つ目のブロックのテストが完全に終了するまで実行されません。  
また`t.test()`を使ってテストブロックを入れ子にすることもできます。

We can run `test/boop.js` with node directly as with `test/beep.js`, but if we
want to run both tests, there is a minimal command-runner we can use that comes
with tape. To get the `tape` command do:

先ほどの`test/beep.js`と同じく`test/boop.js`をNodeから直接実行することもできますが、両方のテストを実行する場合、tapeに付属するミニマムなコマンドランナーを使うこともできます。  
`tape`を利用するには以下のようにしてインストールします。

```
npm install -g tape
```

and now you can run:

すると、以下のように利用できます。

```
$ tape test/*.js
TAP version 13
# beep
ok 1 5*100 === 500
# fraction
ok 2 1/20th of 100
# negative
ok 3 negative number

1..3
# tests 3
# pass  3

# ok
```

and you can just pass `test/*.js` to browserify to run your tests in the
browser:

ブラウザでテストを実行するには`test/*.js`をBrowserifyに渡すだけです。

```
$ browserify test/* | testling

TAP version 13
# beep
ok 1 5*100 === 500
# fraction
ok 2 1/20th of 100
# negative
ok 3 negative number

1..3
# tests 3
# pass  3

# ok
```

Putting together all these steps, we can configure `package.json` with a test
script:

`package.json`にこれらのステップをテストスクリプトとして設定することもできます。

``` json
{
  "name": "hundreder",
  "version": "1.0.0",
  "main": "index.js",
  "devDependencies": {
    "tape": "^2.13.1",
    "testling": "^1.6.1"
  },
  "scripts": {
    "test": "tape test/*.js",
    "test-browser": "browserify test/*.js | testlingify"
  }
}
```

Now you can do `npm test` to run the tests in node and `npm run test-browser` to
run the tests in the browser. You don't need to worry about installing commands
with `-g` when you use `npm run`: npm automatically sets up the `$PATH` for all
packages installed locally to the project.

この設定を行うと、`npm test`でNodeでテストを実行でき、`npm run test-browser`でブラウザでテストを実行できます。  
`npm run`を利用する場合には、npmは自動的にプロジェクト内にインストールされたパッケージに対して、`$PATH`を設定するため、コマンドを`-g`でインストールする必要はありません。

If you have some tests that only run in node and some tests that only run in the
browser, you could have subdirectories in `test/` such as `test/server` and
`test/browser` with the tests that run both places just in `test/`. Then you
could just add the relevant directory to the globs:

いくつかのテストがNodeだけで実行されたり、ブラウザで実行されたりするような場合には、`test/`ディレクトリ内に`test/server`や`test/browser`というサブディレクトリを作ることもできます。`test/`内には両方の環境で実行されるものを置きます。  
そして、Globsに適切なディレクトリを設定すれば問題ありません。

``` json
{
  "name": "hundreder",
  "version": "1.0.0",
  "main": "index.js",
  "devDependencies": {
    "tape": "^2.13.1",
    "testling": "^1.6.1"
  },
  "scripts": {
    "test": "tape test/*.js test/server/*.js",
    "test-browser": "browserify test/*.js test/browser/*.js | testlingify"
  }
}
```

and now server-specific and browser-specific tests will be run in addition to
the common tests.

こうするとサーバ専用とブラウザ専用のテストを共通のテストに加えて実行することができます。

If you want something even slicker, check out
[prova](https://www.npmjs.org/package/prova) once you have gotten the basic
concepts.

基本コンセプトを理解した上で、もっと華麗な解決をしたい場合には[prova](https://www.npmjs.org/package/prova)をチェックしてみてください。

### assert

The core assert module is a fine way to write simple tests too, although it can
sometimes be tricky to ensure that the correct number of callbacks have fired.

コアのアサートモジュールは単純なテストを書くのに十分な方法ですが、コールバックが正しい回数実行されたかなどのようなテストを行うのには面倒な場合もあります。

You can solve that problem with tools like
[macgyver](https://www.npmjs.org/package/macgyver) but it is appropriately DIY.

[macgyver](https://www.npmjs.org/package/macgyver)を使うことでその問題を解決できますが、DIYな解決ではあります。

### mocha


## code coverage

## testling-ci

# bundling

This section covers bundling in more detail.

Bundling is the step where starting from the entry files, all the source files
in the dependency graph are walked and packed into a single output file.

## saving bytes

One of the first things you'll want to tweak is how the files that npm installs
are placed on disk to avoid duplicates.

When you do a clean install in a directory, npm will ordinarily factor out
similar versions into the topmost directory where 2 modules share a dependency.
However, as you install more packages, new packages will not be factored out
automatically. You can however use the `npm dedupe` command to factor out
packages for an already-installed set of packages in `node_modules/`. You could
also remove `node_modules/` and install from scratch again if problems with
duplicates persist.

browserify will not include the same exact file twice, but compatible versions
may differ slightly. browserify is also not version-aware, it will include the
versions of packages exactly as they are laid out in `node_modules/` according
to the `require()` algorithm that node uses.

You can use the `browserify --list` and `browserify --deps` commands to further
inspect which files are being included to scan for duplicates.

## standalone

You can generate UMD bundles with `--standalone` that will work in node, the
browser with globals, and AMD environments.

Just add `--standalone NAME` to your bundle command:

```
$ browserify foo.js --standalone xyz > bundle.js
```

This command will export the contents of `foo.js` under the external module name
`xyz`. If a module system is detected in the host environment, it will be used.
Otherwise a window global named `xyz` will be exported.

You can use dot-syntax to specify a namespace hierarchy:

```
$ browserify foo.js --standalone foo.bar.baz > bundle.js
```

If there is already a `foo` or a `foo.bar` in the host environment in window
global mode, browserify will attach its exports onto those objects. The AMD and
`module.exports` modules will behave the same.

Note however that standalone only works with a single entry or directly-required
file.

## external bundles

## ignoring and excluding

In browserify parlance, "ignore" means: replace the definition of a module with
an empty object. "exclude" means: remove a module completely from a dependency graph.

Another way to achieve many of the same goals as ignore and exclude is the
"browser" field in package.json, which is covered elsewhere in this document.

### ignoring

Ignoring is an optimistic strategy designed to stub in an empty definition for
node-specific modules that are only used in some codepaths. For example, if a
module requires a library that only works in node but for a specific chunk of
the code:

``` js
var fs = require('fs');
var path = require('path');
var mkdirp = require('mkdirp');

exports.convert = convert;
function convert (src) {
    return src.replace(/beep/g, 'boop');
}

exports.write = function (src, dst, cb) {
    fs.readFile(src, function (err, src) {
        if (err) return cb(err);
        mkdirp(path.dirname(dst), function (err) {
            if (err) return cb(err);
            var out = convert(src);
            fs.writeFile(dst, out, cb);
        });
    });
};
```

browserify already "ignores" the `'fs'` module by returning an empty object, but
the `.write()` function here won't work in the browser without an extra step like
a static analysis transform or a runtime storage fs abstraction.

However, if we really want the `convert()` function but don't want to see
`mkdirp` in the final bundle, we can ignore mkdirp with `b.ignore('mkdirp')` or
`browserify --ignore mkdirp`. The code will still work in the browser if we
don't call `write()` because `require('mkdirp')` won't throw an exception, just
return an empty object.

Generally speaking it's not a good idea for modules that are primarily
algorithmic (parsers, formatters) to do IO themselves but these tricks can let
you use those modules in the browser anyway.

To ignore `foo` on the command-line do:

```
browserify --ignore foo
```

To ignore `foo` from the api with some bundle instance `b` do:

``` js
b.ignore('foo')
```

### excluding

Another related thing we might want is to completely remove a module from the
output so that `require('modulename')` will fail at runtime. This is useful if
we want to split things up into multiple bundles that will defer in a cascade to
previously-defined `require()` definitions.

For example, if we have a vendored standalone bundle for jquery that we don't want to appear in
the primary bundle:

```
$ npm install jquery
$ browserify -r jquery --standalone jquery > jquery-bundle.js
```

then we want to just `require('jquery')` in a `main.js`:

``` js
var $ = require('jquery');
$(window).click(function () { document.body.bgColor = 'red' });
```

defering to the jquery dist bundle so that we can write:

``` html
<script src="jquery-bundle.js"></script>
<script src="bundle.js"></script>
```

and not have the jquery definition show up in `bundle.js`, then while compiling
the `main.js`, you can `--exclude jquery`:

```
browserify main.js --exclude jquery > bundle.js
```

To exclude `foo` on the command-line do:

```
browserify --exclude foo
```

To exclude `foo` from the api with some bundle instance `b` do:

``` js
b.exclude('foo')
```

## browserify cdn

# shimming

Unfortunately, some packages are not written with node-style commonjs exports.
For modules that export their functionality with globals or AMD, there are
packages that can help automatically convert these troublesome packages into
something that browserify can understand.

## browserify-shim

One way to automatically convert non-commonjs packages is with
[browserify-shim](https://npmjs.org/package/browserify-shim).

[browserify-shim](https://npmjs.org/package/browserify-shim) is loaded as a
transform and also reads a `"browserify-shim"` field from `package.json`.

Suppose we need to use a troublesome third-party library we've placed in
`./vendor/foo.js` that exports its functionality as a window global called
`FOO`. We can set up our `package.json` with:

``` json
{
  "browserify": {
    "transform": "browserify-shim"
  },
  "browserify-shim": {
    "./vendor/foo.js": "FOO"
  }
}
```

and now when we `require('./vendor/foo.js')`, we get the `FOO` variable that
`./vendor/foo.js` tried to put into the global scope, but that attempt was
shimmed away into an isolated context to prevent global pollution.

We could even use the [browser field](#browser-field) to make `require('foo')`
work instead of always needing to use a relative path to load `./vendor/foo.js`:

``` json
{
  "browser": {
    "foo": "./vendor/foo.js"
  },
  "browserify": {
    "transform": "browserify-shim"
  },
  "browserify-shim": {
    "foo": "FOO"
  }
}
```

Now `require('foo')` will return the `FOO` export that `./vendor/foo.js` tried
to place on the global scope.

# partitioning

Most of the time, the default method of bundling where one or more entry files
map to a single bundled output file is perfectly adequate, particularly
considering that bundling minimizes latency down to a single http request to
fetch all the javascript assets.

However, sometimes this initial penalty is too high for parts of a website that
are rarely or never used by most visitors such as an admin panel.
This partitioning can be accomplished with the technique covered in the
[ignoring and excluding](#ignoring-and-excluding) section, but factoring out
shared dependencies manually can be tedious for a large and fluid dependency
graph.

Luckily, there are plugins that can automatically factor browserify output into
separate bundle payloads.

## factor-bundle

[factor-bundle](https://www.npmjs.org/package/factor-bundle) splits browserify
output into multiple bundle targets based on entry-point. For each entry-point,
an entry-specific output file is built. Files that are needed by two or more of
the entry files get factored out into a common bundle.

For example, suppose we have 2 pages: /x and /y. Each page has an entry point,
`x.js` for /x and `y.js` for /y.

We then generate page-specific bundles `bundle/x.js` and `bundle/y.js` with
`bundle/common.js` containing the dependencies shared by both `x.js` and `y.js`:

```
browserify x.js y.js -p [ factor-bundle -o bundle/x.js -o bundle/y.js ] \
  -o bundle/common.js
```

Now we can simply put 2 script tags on each page. On /x we would put:

``` html
<script src="/bundle/common.js"></script>
<script src="/bundle/x.js"></script>
```

and on page /y we would put:

``` html
<script src="/bundle/common.js"></script>
<script src="/bundle/y.js"></script>
```

You could also load the bundles asynchronously with ajax or by inserting a
script tag into the page dynamically but factor-bundle only concerns itself with
generating the bundles, not with loading them.

## partition-bundle

[partition-bundle](https://www.npmjs.org/package/partition-bundle) handles
splitting output into multiple bundles like factor-bundle, but includes a
built-in loader using a special `loadjs()` function.

partition-bundle takes a json file that maps source files to bundle files:

```
{
  "entry.js": ["./a"],
  "common.js": ["./b"],
  "common/extra.js": ["./e", "./d"]
}
```

Then partition-bundle is loaded as a plugin and the mapping file, output
directory, and destination url path (required for dynamic loading) are passed
in:

```
browserify -p [ partition-bundle --map mapping.json \
  --output output/directory --url directory ]
```

Now you can add:

``` html
<script src="entry.js"></script>
```

to your page to load the entry file. From inside the entry file, you can
dynamically load other bundles with a `loadjs()` function:

``` js
a.addEventListener('click', function() {
  loadjs(['./e', './d'], function(e, d) {
    console.log(e, d);
  });
});
```

# compiler pipeline

Since version 5, browserify exposes its compiler pipeline as a
[labeled-stream-splicer](https://www.npmjs.org/package/labeled-stream-splicer).

This means that transformations can be added or removed directly into the
internal pipeline. This pipeline provides a clean interface for advanced
customizations such as watching files or factoring bundles from multiple entry
points.

For example, we could replace the built-in integer-based labeling mechanism with
hashed IDs by first injecting a pass-through transform after the "deps" have
been calculated to hash source files. Then we can use the hashes we captured to
create our own custom labeler, replacing the built-in "label" transform:

``` js
var browserify = require('browserify');
var through = require('through2');
var shasum = require('shasum');

var b = browserify('./main.js');

var hashes = {};
var hasher = through.obj(function (row, enc, next) {
    hashes[row.id] = shasum(row.source);
    this.push(row);
    next();
});
b.pipeline.get('deps').push(hasher);

var labeler = through.obj(function (row, enc, next) {
    row.id = hashes[row.id];

    Object.keys(row.deps).forEach(function (key) {
	row.deps[key] = hashes[row.deps[key]];
    });

    this.push(row);
    next();
});
b.pipeline.get('label').splice(0, 1, labeler);

b.bundle().pipe(process.stdout);
```

Now instead of getting integers for the IDs in the output format, we get file
hashes:

```
$ node bundle.js
(function e(t,n,r){function s(o,u){if(!n[o]){if(!t[o]){var a=typeof require=="function"&&require;if(!u&&a)return a(o,!0);if(i)return i(o,!0);var f=new Error("Cannot find module '"+o+"'");throw f.code="MODULE_NOT_FOUND",f}var l=n[o]={exports:{}};t[o][0].call(l.exports,function(e){var n=t[o][1][e];return s(n?n:e)},l,l.exports,e,t,n,r)}return n[o].exports}var i=typeof require=="function"&&require;for(var o=0;o<r.length;o++)s(r[o]);return s})({"5f0a0e3a143f2356582f58a70f385f4bde44f04b":[function(require,module,exports){
var foo = require('./foo.js');
var bar = require('./bar.js');

console.log(foo(3) + bar(4));

},{"./bar.js":"cba5983117ae1d6699d85fc4d54eb589d758f12b","./foo.js":"736100869ec2e44f7cfcf0dc6554b055e117c53c"}],"cba5983117ae1d6699d85fc4d54eb589d758f12b":[function(require,module,exports){
module.exports = function (n) { return n * 100 };

},{}],"736100869ec2e44f7cfcf0dc6554b055e117c53c":[function(require,module,exports){
module.exports = function (n) { return n + 1 };

},{}]},{},["5f0a0e3a143f2356582f58a70f385f4bde44f04b"]);
```

Note that the built-in labeler does other things like checking for the external,
excluded configurations so replacing it will be difficult if you depend on those
features. This example just serves as an example for the kinds of things you can
do by hacking into the compiler pipeline.

## build your own browserify

## labeled phases

Each phase in the browserify pipeline has a label that you can hook onto. Fetch
a label with `.get(name)` to return a
[labeled-stream-splicer](https://npmjs.org/package/labeled-stream-splicer)
handle at the appropriate label. Once you have a handle, you can `.push()`,
`.pop()`, `.shift()`, `.unshift()`, and `.splice()` your own transform streams
into the pipeline or remove existing transform streams.

### recorder

The recorder is used to capture the inputs sent to the `deps` phase so that they
can be replayed on subsequent calls to `.bundle()`. Unlike in previous releases,
v5 can generate bundle output multiple times. This is very handy for tools like
watchify that re-bundle when a file has changed.

### deps

The `deps` phase expects entry and `require()` files or objects as input and
calls [module-deps](https://npmjs.org/package/module-deps) to generate a stream
of json output for all of the files in the dependency graph.

module-deps is invoked with some customizations here such as:

* setting up the browserify transform key for package.json
* filtering out external, excluded, and ignored files
* setting the default extensions for `.js` and `.json` plus options configured
in the `opts.extensions` parameter in the browserify constructor
* configuring a global [insert-module-globals](#insert-module-globals)
transform to detect and implement `process`, `Buffer`, `global`, `__dirname`,
and `__filename`
* setting up the list of node builtins which are shimmed by browserify

### json

This transform adds `module.exports=` in front of files with a `.json`
extension.

### unbom

This transform removes byte order markers, which are sometimes used by windows
text editors to indicate the endianness of files. These markers are ignored by
node, so browserify ignores them for compatibility.

### syntax

This transform checks for syntax errors using the
[syntax-error](https://npmjs.org/package/syntax-error) package to give
informative syntax errors with line and column numbers.

### sort

This phase uses [deps-sort](https://www.npmjs.org/package/deps-sort) to sort
the rows written to it in order to make the bundles deterministic.

### dedupe

The transform at this phase uses dedupe information provided by
[deps-sort](https://www.npmjs.org/package/deps-sort) in the `sort` phase to
remove files that have duplicate contents.

### label

This phase converts file-based IDs which might expose system path information
and inflate the bundle size into integer-based IDs.

The `label` phase will also normalize path names based on the `opts.basedir` or
`process.cwd()` to avoid exposing system path information.

### emit-deps

This phase emits a `'dep'` event for each row after the `label` phase.

### debug

If `opts.debug` was given to the `browserify()` constructor, this phase will
transform input to add `sourceRoot` and `sourceFile` properties which are used
by [browser-pack](https://npmjs.org/package/browser-pack) in the `pack` phase.

### pack

This phase converts rows with `'id'` and `'source'` parameters as input (among
others) and generates the concatenated javascript bundle as output
using [browser-pack](https://npmjs.org/package/browser-pack).

### wrap

This is an empty phase at the end where you can easily tack on custom post
transformations without interfering with existing mechanics.

## browser-unpack

[browser-unpack](https://npmjs.org/package/browser-unpack) converts a compiled
bundle file back into a format very similar to the output of
[module-deps](https://npmjs.org/package/module-deps).

This is very handy if you need to inspect or transform a bundle that has already
been compiled.

For example:

``` js
$ browserify src/main.js | browser-unpack
[
{"id":1,"source":"module.exports = function (n) { return n * 100 };","deps":{}}
,
{"id":2,"source":"module.exports = function (n) { return n + 1 };","deps":{}}
,
{"id":3,"source":"var foo = require('./foo.js');\nvar bar = require('./bar.js');\n\nconsole.log(foo(3) + bar(4));","deps":{"./bar.js":1,"./foo.js":2},"entry":true}
]
```

This decomposition is needed by tools such as
[factor-bundle](https://www.npmjs.org/package/factor-bundle)
and [bundle-collapser](https://www.npmjs.org/package/bundle-collapser).

# plugins

When loaded, plugins have access to the browserify instance itself.

## using plugins

Plugins should be used sparingly and only in cases where a transform or global
transform is not powerful enough to perform the desired functionality.

You can load a plugin with `-p` on the command-line:

```
$ browserify main.js -p foo > bundle.js
```

would load a plugin called `foo`. `foo` is resolved with `require()`, so to load
a local file as a plugin, preface the path with a `./` and to load a plugin from
`node_modules/foo`, just do `-p foo`.

You can pass options to plugins with square brackets around the entire plugin
expression, including the plugin name as the first argument:

```
$ browserify one.js two.js \
  -p [ factor-bundle -o bundle/one.js -o bundle/two.js ] \
  > common.js
```

This command-line syntax is parsed by the
[subarg](https://npmjs.org/package/subarg) package.

To see a list of browserify plugins, browse npm for packages with the keyword
"browserify-plugin": http://npmjs.org/browse/keyword/browserify-plugin

## authoring plugins

To author a plugin, write a package that exports a single function that will
receive a bundle instance and options object as arguments:

``` js
// example plugin

module.exports = function (b, opts) {
  // ...
}
```

Plugins operate on the bundle instance `b` directly by listening for events or
splicing transforms into the pipeline. Plugins should not overwrite bundle
methods unless they have a very good reason.
