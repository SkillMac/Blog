今天主要记录一下 TweenLite 这个补间动画库,他们的官方地址
[TweenLite官方网址](https://greensock.com)

库版本是:2.0.2

这里面有几个文件可以使用 TweenLite TimelineLite TweenlineMax TweenMax

我这里就简单说一下这几个文件都做了什么事情,说先说明一下我这里主要研究跟 `CocosCreator` 相关的功能,它里面还有一些关于h5的东西,感兴趣的可以了解一下.
<!-- more -->

##讲解每个文件的功能

###TweenLite
这个是 Tween 动画的基础库, 所有的其他更丰富的功能都是在这个基础上封装的.

里面主要使用 
```js
	TweenLite.to(target:Object, duration:Number, vars:Object)
```
第一个是操作的对象,第二个是执行的时间,第三个是要进行补间的属性.

首先你们要导入这个文件(TweenLite.js)
你下载的文件里面应该是有个压缩后的文件 (TweenLite.min.js) 这个更省空间

例如如下的例子
```js
import TweenLite from "TweenLite.min"

function main() {
	TweenLite.to(this.node,2,{x:100,y:"+=100",rotation:45,opacity:"+=255",onComplete:()=>{
		console.log('执行结束');
	}});
}

main();
```

上面这个例子是 让一个node 同时执行 位置, 旋转 和透明度的 变换,
这种写法要比 CocosCreator 的那套 Action 要简洁

其他的 API 我就捡着几个重要的说一下吧,其他的你们可以看一下他们的[官文文档](https://greensock.com/docs/)讲解的很清楚

主要说第三个参数
```js
{
	x:100,// 跑到当前坐标系下 x=100 的位置
	y:"+=100", // 自身值 增加100
	...,

	onStartParams:['123'],
	onStart:(p1)=>{console.log(p1)},// 输出 123  在这个动画开始执行的时候调用
	onCompleteParams:['123'],
	onComplete:(p1)=>{console.log(p1)},// 输出入123 在这个动画结束的时候调用
	onReverseComplete:()=>{}, // 调用翻转函数的说 这个动画结束后 会调用
	delay:2,// 开始延时
	ease: Elastic.easeOut, // 要使用这东西要导入 easeing 文件下的那个 easePack.js 文件
}
```

调用TweenLite.to 函数的时候是会去返回一个对象的, 在这个补间动画库中 使用的是链式(chain)调用, 这个对象使用几个 api 的

>pause 暂停当前的动画
>play 播放当前动画
>resume 恢复当前动画
>reverse 在任意时间都可以调用这个 翻转函数 翻转当前动画
>restart 从新开始
>progress 它的取值区间是[0,1] 指定当前动画处开始播放
>seek 跳转到指定位置 和 progress 函数有点相似 这个指的是时间的位置点

例子如下
```js
import Ease from "EasePack.min";
import TweenLite from "TweenLite.min"

let tl = TweenLite.to(this.node,2,{
	x:100,
	ease: Elastic.easeOut
});
tl.play();
tl.pause();
tl.resume();
tl.restart();
tl.progress(0.5);
tl.seek(0.5);
tl.progress(0.5).pause();
```
因为他是链式调用所以 最后一个的写法是不会报错的.
你们要是想要使用 ease 这个属性的话 是需要将 EasePack.min.js 导入到工程中 不需要在代码中导入, 至于为什么可以自己去看他们 uncompressed 文件下对应的代码文件.

后面那三个文件都是在这个基础上扩展的所以需要理解这个文件的用法.

> TweenLite 还有其他的几个静态函数 TweenLite.to TweenLite.from TweenLite.fromTo 用法比较的简单看文档也就可知道了.

###TimelineLite

看这个名字凭字面上的意思是 时间线 也可以理解成 时间轴,看官方文档解释是他可以形成一个队列,就是说你可以组成一个动画列表一次播放.

例子如下
```js
import TimelineLite from "TimelineLite.min"

let tl = new TimelineLite();// 创建一个新实例对象

tl.to(this.node,2,{x:"+=100"})
.to(this.node,2,{y:"+=100"});
```

这个就是创建了一个动画队列,让一个物体向x轴正方向移动100像素, 然后在向y轴正方向移动100像素.

里面的参数配置和 TweenLite 里面的配置是一样的.

他提供了 TweenLite对象的嵌套 同时也支持 Timeline 的嵌套 为什么这么说看下面的例子

```js
import TimelineLite from "TimelineLite.min";
import TweenLite from "TweenLite.min";

let tl = new TimelineLite();// 创建一个新实例对象

tl.to(this.node,2,{x:"+=100"})
.to(this.node,2,{y:"+=100"});

let tl1 = new TimelineLite();
tl1.add(TweenLite.to(this.node,2,{x:"+=100"}));
tl1.add(TweenLite.to(this.node,2,{y:"+=100"}));

let tl2 = new TimelineLite();
tl2.to(this.node,2,{x:"+=100"});

tl1.append(tl2);
```

这里面 to 和 add 的功能效果都是一样的 可以看到 add 里面是个 TweenLite 对象 所以 TimelineLite是维护 TweenLite每个对象进而形成 一个队列

>TimelineLite 还有其他的用 入 addLabel 等函数,看他们解释和代码例子很容易懂

###TimelineMax

是对 TimelineLite 进行的扩展 主要扩展 使用 加入 repeat repeatDelay yoyo  currentLabel(), addCallback(), removeCallback(), tweenTo(), tweenFromTo(), getLabelAfter(), getLabelBefore(), getActive().

如果上面额功能不能满足你们项目的需求可以再去用 TimelineMax 这个更强大额 扩展包.

###TweenMax
TweenMax 是对 TweenLite 的扩展, 也加入 repeat(), repeatDelay(), yoyo() 等等, 而且看他们未压缩的 代码 好像这个文件是可以不要依赖 其他三的,因为他把其他三个都压缩到这个文件里面了,但是相应的文件的大小也增加了,同时也集成了一些扩展插件. 你们的项目如果对 代码的大小没有太高的要求的话,你们可以去直接导入这个TweenMax.min.js 这个文件.


上面讲了这么多也只是个入们吧,这个库里面还封装了一些很多有趣的东西,你们可以去多了解了解.

这里使用这个库去替代 CocosCreator 的 Action 主要是 api 使用灵活, 写法简单,运行速度也很快. 也有一些 Action 所不具有的动能 入随时翻转,控制动画流程,暂停动画等等,随机点开始播放动画,时间轴交叉等.

好了,今天就先到这里.
写作不易,且行且珍惜.

我是小魏. 时间:2018年11月21日17:16:36.