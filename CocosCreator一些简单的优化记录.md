### cocos creator 一些简单的优化记录

目标: 微信小游戏
引擎版本: 2.0.8
语言: js
时间: 2019年2月19日15:50:29

大纲:
	- 缩小包体
	- 优化 drawcall
	- 增加游戏编译后包体的安全性(防止反编译)
	- 优化游戏的计算量

微信小游戏对包体的大小控制的很严格只有`4M`后面又提出了分包加载的机制,但是能自己优化一些还是更好的一些做法.

<!-- more -->

# 缩小包体

## 压缩纹理
这里提供的第一个方法是压缩纹理,这个是比较的直接有效果.

1.压缩纹理使用 [TinyPng](https://tinypng.com/), [pngquant](https://pngquant.org/), 等等压缩纹理的工具.
> 这里有一个使用脚本进行压缩的工具,最近在看了一下 `gulp`,发现有个png压缩的插件,于是就写了一个批量用脚本压缩的工具吧,很简陋用法也很简单.
> 如果你想看懂的具体的细节的话,需要去阅读以下[gulp](https://gulpjs.com/)的基本知识点吧. 其实 gulp 还是相对比较的简单易用的.
> 这些依赖的包你就自己安装,如果不会的话,你可以去阅读`nodejs`的`npm` (对包管理的一个工具) 相关知识点吧.

[插件地址](https://www.npmjs.com/package/gulp-tinypng-nokey)

```js
// 配置projectPath => {你的工程根路径/build}
var projectPath = "";

var gulp = require('gulp');
var tinypng_nokey = require('gulp-tinypng-nokey');

gulp.task('default', function (cb) {
    gulp.src([projectPath + "/res/raw-assets/**/*.{png,jpg,jpeg}"])
        .pipe(tinypng_nokey())
        .pipe(gulp.dest(projectPath + "/res/dest/"))
        .on("end", cb);
});
```

* `注意`： 
    * 如果执行`gulp`命令时提示"不是内部或外部命令，也不是可运行的程序或批处理文件"，则需要执行`npm i -g gulp`命令重新安装，完成后再执行`gulp`命令。
    * 成功安装过一次之后，下一个项目在操作时不用再次安装。

> 进入命令行
```
gulp
```

这个操作执行完会在 `{projectPath/res/dest}` 当然你也可以直接输出到本目录,它会直接进行替换的.(或者更改成你想要的目录)
大概纹理会减小 30% ~ 40%.

{% asset_img sp6.jpg build compress png %}

2.这里你可以使用[TexturePacker](https://www.codeandweb.com/texturepacker)这些第三方的辅助工具进行纹理压缩,你也可以使用`CocosCreator`内置的`auto atlas`(自动图集)的工具进行合图.

* 做第二步的目的是:
	* 可以减少总体的透明像素,也会有一定的体积减少
	* 能够`和批`优化游戏的性能

3.如果做原生的游戏的话你可以使用 `*.webp` 这种后缀的图片是 png 的物理体积的 1/10 倍. 你可直接在百度搜索 png 转 webp 应该有相关的脚本文件, 依赖之间的工作经验的话, 你可以写 python 去做这件事情, 当然也可以是其他的语言.

* `注意`
	* 这种做法只是物理体积的减小,如果是你又资源是需要在远端下载的本地的需求那这将是一个不错的选择,但是他并不能优化的运行时所占用的内存.意思就是说你 png 占用的内存在 webp 这种格式 他同样占用这么多.
	* 由于微信不允许这种纹理后缀名字,所以这种做法不适用于微信小游戏.
	* png 如果压缩过图片是不能进行 png 到 webp 的转换的 会出错, 解决方案是让你们的美术人员将这张图片导入 ps 以不压缩的方式从新导出.
	* 这里面贴出代码, 感兴趣的你也可以自己去实现一下.

## 移除不需要的模块
在打包前，找到项目设置-->模块设置，将项目中没有用到的模块移除（取消勾选状态），如下图所示。
{% asset_img module.png x %}

* `注意`
	* 这个也许不能优化游戏的运行速度但是能有效的减少你的包体大小.
	* 尤其是你游戏中有好友排行榜,这个会含有两个 Creator 的引擎库文件是极其占用包体资源的.

# 优化 drawcall

> 游戏中 drawcall 是游戏性能比较重要的一个指标, drawcall 越低,证明 cpu 给 gpu 传递次数越少, 每次 cpu 通知 gpu 是需要性能开销的如果能一次传递过去,就能优化游戏的运行效率的.

## 自动合图
* 尽量让你相同的精灵帧是挨在一起的.

类似下面这样:
```
A A A B 这样就会只有2个`drawcall`
A B A A 这样的设计时很糟糕的. => 3 个 drawcall

- Node
	- A
- Node
	- A
// 这种排列不影响 和批 1 个 drawcall

- Node
	- A
	- B
- Node
	- A
	- B
// 这种的排列布局 不能和批 4 个 drawcall

相同的预制件不能 和批
```

* `注意`
	* 这个只针对于cocos creator 的渲染机制, 不同的引擎他的和批策略可能不相同.

## 将碎图进行合图

这里面有将多个碎图合成一个大图, 这样也可以实现 和批 处理,
* `这里有两种实现合图`:
	* [TexturePacker](https://www.codeandweb.com/texturepacker)
	* auto atlas 这个是`Cocos Creator`内置的合图工具.

下面说一下这两种工具在开发的过程中的优缺点.

1.TexturePacker 的 drawcall的优化在浏览器中可以直接看到, 后者需要打包后才能看到.
2.在开发过程中如果纹理更换的频繁,前者需要不停的打包,开发效率低下,后者则不用管(因为是在打包后才能看的见合图的图集).

## fnt的图片 和 游戏资源进行合图

[官方描述](https://docs.cocos.com/creator/manual/zh/advanced-topics/ui-auto-batch.html)

{% asset_img sp8.jpg x %}
* 将需要自动合图的图片放在同一个目录下，并在此目录下创建一个`自动图集配置（AutoAtlas）`
* 可以勾选不包含未引用的资源 要管理好自己的资源目录避免一些没用的资源也打包到一个图集里面.

* `注意`
	* 系统字体会打断和批 做倒计时尽量避免使用系统字体,效率没有 fnt 的搞笑, 因为一个系统字体是包含中文和英文的, 在没有必要的情况下, 尽量就不要使用.
	* 做和批测试的时候尽量使用 `Google Chrome 浏览器`,其他浏览器在和批效果上好像没有效果.但是小游戏其实算是属于一个原生的游戏,所以这些都会微信小游戏还是生效的.

## 动态合图

[官方描述](https://docs.cocos.com/creator/manual/zh/advanced-topics/dynamic-atlas.html)

这个特性是 `Cocos Creator` 2.x 更换渲染层提供的机制.更大的提高游戏性能一种机制.

动态合图功能使用更简单，只需要在代码中开启动态合图功能即可，也可以更改相关属性。
```js
cc.dynamicAtlasManager.enabled = true;      // 开启动态合图
//cc.dynamicAtlasManager.maxAtlasCount = 5;   // 最大合图数量
//cc.dynamicAtlasManager.textureSize = 1024;  // 创建的图集的宽高
//cc.dynamicAtlasManager.maxFrameSize = 512;  // 可以添加进图集的图片的最大尺寸
```

* `注意`
	* 动态合图时，目前发现除了Chrome浏览器上`DrawCall`次数正常以外，其他浏览器的`DrawCall`次数并非预期结果。

# 增加游戏编译后包体的安全性(防止反编译)

由于js动态运行时语言,所以你开发的项目可能很容易被别人给反编译,窃取了你的劳动成果.

这里只是有效的让代码进行混合,并不能保证一定不能被反编译.

这里还是用 `gulp` 提供的插件 `gulp-javascript-obfuscator`
[插件的介绍地址](https://www.npmjs.com/package/gulp-javascript-obfuscator)
[插件github地址](https://github.com/javascript-obfuscator/javascript-obfuscator)

里面的option(设置)很多

```
compact: true,
controlFlowFlattening: false,
controlFlowFlatteningThreshold: 0.75,
deadCodeInjection: false,
deadCodeInjectionThreshold: 0.4,
debugProtection: false,
debugProtectionInterval: false,
disableConsoleOutput: false,
domainLock: [],
identifierNamesGenerator: 'hexadecimal',
identifiersPrefix: '',
inputFileName: '',
log: false,
renameGlobals: false,
reservedNames: [],
reservedStrings: [],
rotateStringArray: true,
seed: 0,
selfDefending: false,
sourceMap: false,
sourceMapBaseUrl: '',
sourceMapFileName: '',
sourceMapMode: 'separate',
stringArray: true,
stringArrayEncoding: false,
stringArrayThreshold: 0.75,
target: 'browser',
transformObjectKeys: false,
unicodeEscapeSequence: false
```

```js
var projectPath = "";

var javascriptObfuscator = require("gulp-javascript-obfuscator")

gulp.task("js", function (cb) {
    gulp.src([projectPath + "/src/project.js"])
        .pipe(javascriptObfuscator({
            // compact: true, //类型：boolean默认：true
            mangle: true, //短标识符的名称，如a，b，c
            stringArray: true,
            target: "browser",
        }))
        .pipe(gulp.dest(projectPath + "/js-dst")
        .on("end", cb));
});
```

* `注意`
	* 可能会使包体增大一点点,这个你可以在几去做权衡.

# 优化游戏的计算量

* 减少对粒子的使用,或者是减少总共的粒子数.
* 游戏的数据配置,尽量使用数据简洁,紧凑的数据格式(json),读取速度快.
* 尽量减少混合,混合是逐像素进行计算的,计算量较大.
* 减少 `富文本` 和 `mask` 占用 drawcall 的数量比较多.
* 尽量少使用 系统字体(ttf) 不能和批, 效率没有 游戏项目里面的 fnt 效率高.
* 图片不符合尺寸尽量然美术进行调整,不然虽然效果一致的,但是运行时占用的内存是不一样的.游戏是(物理缩放), ps(本质的缩放).