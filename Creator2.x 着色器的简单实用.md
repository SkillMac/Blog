Cocos Creator 版本: 2.0.8
时间: 2019年3月15日16:53:18

由于Cocos 的材质系统并不是太完善 所以官方并没有推出相关 着色器的文档,但是官方给出了一个Demo, 可以更具这个进行改造一下就可以写自己的着色器了.
(官方给出的着色器的例子)[https://github.com/cocos-creator/heartfelt]

# 给 Sprite 填充一些函数

```js
// 支持自定义Shader
const renderEngine = cc.renderer.renderEngine;
const SpriteMaterial = renderEngine.SpriteMaterial;
const GraySpriteMaterial = renderEngine.GraySpriteMaterial;
const STATE_CUSTOM = 101;

// 取自定义材质
cc.Sprite.prototype.getMaterial = function(name) {
    if (this._materials) {
        return this._materials[name];
    } else {
        return undefined;
    }
}

// 设置自定义材质
cc.Sprite.prototype.setMaterial = function(name, mat) {
    if (!this._materials) {
        this._materials = {}
    }
    this._materials[name] = mat;
}

// 激活某个材质
cc.Sprite.prototype.activateMaterial = function(name) {
    var mat = this.getMaterial(name);
    if (mat && mat !== this._currMaterial) {
        if (mat) {
            if (this.node) {
                mat.color = this.node.color;
            }
            if (this.spriteFrame) {
                mat.texture = this.spriteFrame.getTexture();
            }
            this.node._renderFlag |= cc.RenderFlow.FLAG_COLOR;
            this._currMaterial = mat;
            this._currMaterial.name = name;
            this._state = STATE_CUSTOM;
            this._activateMaterial();
        } else {
            console.error("activateMaterial - unknwon material: ", name);
        }
    }
}

// 取当前的材质


cc.Sprite.prototype._activateMaterial = function() {
    let spriteFrame = this._spriteFrame;

    // WebGL 暂时支持WebGL
    if (cc.game.renderType !== cc.game.RENDER_TYPE_CANVAS) {
        // Get material
        let material;
        if (this._state === cc.Sprite.State.GRAY) {
            if (!this._graySpriteMaterial) {
                this._graySpriteMaterial = new GraySpriteMaterial();
                this.node._renderFlag |= cc.RenderFlow.FLAG_COLOR;
            }
            material = this._graySpriteMaterial;
            this._currMaterial = null;
        }
        else if (this._state === STATE_CUSTOM) {
            if (!this._currMaterial) {
                console.error("_activateMaterial: _currMaterial undefined!")
                return;
            }
            material = this._currMaterial;
        }
        else {
            if (!this._spriteMaterial) {
                this._spriteMaterial = new SpriteMaterial();
                this.node._renderFlag |= cc.RenderFlow.FLAG_COLOR;
            }
            material = this._spriteMaterial;
            this._currMaterial = null;
        }
        // Set texture
        if (spriteFrame && spriteFrame.textureLoaded()) {
            let texture = spriteFrame.getTexture();
            if (material.texture !== texture) {
                material.texture = texture;
                this._updateMaterial(material);
            }
            else if (material !== this._material) {
                this._updateMaterial(material);
            }
            if (this._renderData) {
                this._renderData.material = material;
            }
            this.markForUpdateRenderData(true);
            this.markForRender(true);
        }
        else {
            this.disableRender();
        }
    }
}
```

# 构建一个Shader Library

> 在自定义Shader的时候 要在游戏运行前, 传递自定义的 Shader 给引擎进行编译

```js
/*
 Shader Library
*/
var ShaderLib = {
    _shaders: {},

    // 增加一个新的Shader
    addShader: function(shader) {
        if (this._shaders[shader.name]) {
            console.error("addShader - shader already exist: ", shader.name);
            return;
        }
        cc.renderer._forward._programLib.define(shader.name, shader.vert, shader.frag, shader.defines);
        this._shaders[shader.name] = shader;
    },

    // 取Shader的定义
    getShader: function(name) {
        return this._shaders[name];
    }
}
```

# 构建自定义材质(Material)

> 这个和 Unity 差不多 也是 多通道(pass)渲染 但是这个例子里面只有 单通道, 没有用到多通道,
> 我这里举一个多通道的例子吧 比如渲染 描边, 在第一个通道渲染 比原图大5像素的图片, 第二个通道 渲染原图, 进而实现描边.
> 至于这个 多通道 我后面看一下 Creator Engine 源码吧 ,, 有时间写一下,,

```js
/**
 * Custom Material
 */
const renderEngine = cc.renderer.renderEngine;
const renderer = renderEngine.renderer;
const gfx = renderEngine.gfx;
const Material = renderEngine.Material;

var CustomMaterial = (function (Material$$1) {
	function CustomMaterial(shaderName, params, defines, passList) {
		Material$$1.call(this, false);
		var pass;
		if(!passList) {
			// 配置一个简单的的通道 你仔细看后发现后面其实 是个 数组,, 也就是说可以配置多个通道.
			// 这里的 shaderName 就是 前面 传递给引擎的 自定义 Shader 的名字
			pass = new renderer.Pass(shaderName);
			pass.setDepth(false, false);
			pass.setCullMode(gfx.CULL_NONE);
			pass.setBlend(
				gfx.BLEND_FUNC_ADD,
				gfx.BLEND_SRC_ALPHA, gfx.BLEND_ONE_MINUS_SRC_ALPHA,
				gfx.BLEND_FUNC_ADD,
				gfx.BLEND_SRC_ALPHA, gfx.BLEND_ONE_MINUS_SRC_ALPHA
			);
		}
		
		var techParams = [
			{ name: 'texture', type: renderer.PARAM_TEXTURE_2D },
			{ name: 'color', type: renderer.PARAM_COLOR4 }
		];
		if (params) {
			techParams = techParams.concat(params);
		}
		var mainTech = new renderer.Technique(
			['transparent'],
			techParams,
			passList ? passList : [pass],
		);
		this.name = shaderName;
		this._color = { r: 1, g: 1, b: 1, a: 1 };
		this._effect = new renderer.Effect(
			[
				mainTech
			],
			{},
			defines,
		);

		this._mainTech = mainTech;
		this._texture = null;
	}

	if (Material$$1) CustomMaterial.__proto__ = Material$$1;
	CustomMaterial.prototype = Object.create(Material$$1 && Material$$1.prototype);
	CustomMaterial.prototype.constructor = CustomMaterial;

	var prototypeAccessors = { effect: { configurable: true }, texture: { configurable: true }, color: { configurable: true } };

	prototypeAccessors.effect.get = function () {
		return this._effect;
	};

	prototypeAccessors.texture.get = function () {
		return this._texture;
	};

	prototypeAccessors.texture.set = function (val) {
		if (this._texture !== val) {
			this._texture = val;
			this._effect.setProperty('texture', val.getImpl());
			this._texIds['texture'] = val.getId();
		}
	};

	prototypeAccessors.color.get = function () {
		return this._color;
	};

	prototypeAccessors.color.set = function (val) {
		var color = this._color;
		color.r = val.r / 255;
		color.g = val.g / 255;
		color.b = val.b / 255;
		color.a = val.a / 255;
		this._effect.setProperty('color', color);
	};

	CustomMaterial.prototype.clone = function clone() {
		var copy = new CustomMaterial();
		copy.texture = this.texture;
		copy.color = this.color;
		copy.updateHash();
		return copy;
	};

	// 设置自定义参数的值
	CustomMaterial.prototype.setParamValue = function (name, value) {
		this._effect.setProperty(name, value);
	};

	// 设置定义值
	CustomMaterial.prototype.setDefine = function (name, value) {
		this._effect.define(name, value);
	};

	Object.defineProperties(CustomMaterial.prototype, prototypeAccessors);

	return CustomMaterial;
}(Material));
```
# 开始测试刚才写的代码

```js
// 先 Creator 的 写法太长 进行简单的封装
const renderer = cc.renderer.renderEngine.renderer;
const PARAMTYPE = {
    i: renderer.PARAM_INT,
    i2: renderer.PARAM_INT2,
    i3: renderer.PARAM_INT3,
    i4: renderer.PARAM_INT4,

    f : renderer.PARAM_FLOAT,
    f2 : renderer.PARAM_FLOAT2,
    f3 : renderer.PARAM_FLOAT3,
    f4 : renderer.PARAM_FLOAT4,

    mat2: renderer.PARAM_MAT2,
    mat3: renderer.PARAM_MAT3,
    mat4: renderer.PARAM_MAT4,

    tex2d: renderer.PARAM_TEXTURE_2D,
    texCube: renderer.PARAM_TEXTURE_CUBE,
}
cc.Class({
	properties: {
	},

	onLoad() {
		// 初始化 给 Sprite 补充的函数
		require("SpriteHook").init();
		// 初始化 自定义 的 Shader
		// 这个 Flash 文件 后面同意给出
		// 因为我还写了几个简单的 Shader
		ShaderLib.addShader(require("Flash"));

		// 初始化 为了实现这效果需要的变量
		this._time = 0;
		this._sin = 0;
		this.sp = this.node.getComponent(cc.Sprite);

		this.flashShader();
	},

	/*
	给精灵设置 材质
	*/
	flashShader() {
	    let name = 'Flash';
	    let material = this.sp.getMaterial(name);
	    if(!material) {
	        var CustomMaterial = require("CustomMaterial");
	        material = new CustomMaterial(name,[
	            {name: 'run_time', type: PARAMTYPE.f}
	        ]);
	        this.sp.setMaterial(name, material);
	    }

	    this.sp.activateMaterial(name);
	},

	update(dt) {
		this._time += dt;
		this._sin = Math.sin(this._time);
	    if(this._sin > 0.99) {
	    	this._sin = 0;
	        this._time = 0;
	    }
	    const material = this.sp.getCurrMaterial('Flash');
	    if(!material) {
	        return;
	    }
	    material.setParamValue('run_time', this._sin);
	}
});

```

# 自定义 着色器的代码
## Flash (扫光)
```js
// 要初始化的变量
this._time = 0;
this._sin = 0;

// 要不停更新的变量(就是要放在update函数里面的变量)
this._time += dt;
this._sin = Math.sin(this._time);
if(this._sin > 0.99) {
	this._sin = 0;
    this._time = 0;
}
const material = this.sp.getCurrMaterial('Flash');
if(!material) {
    return;
}
material.setParamValue('run_time', this._sin);
```
```js
var shader = {
    name: "Flash",

    defines: [],

    vert: `uniform mat4 viewProj;
    attribute vec3 a_position;
    attribute vec2 a_uv0;

    varying vec2 v_uv0;

    void main() {
        vec4 pos = viewProj * vec4(a_position, 1.);
        gl_Position = pos;
        v_uv0 = a_uv0;
    }
    `,

    frag: `uniform sampler2D texture;
    /*uniform vec4 color;*/
    uniform float run_time;
    varying vec2 v_uv0;
    
    void main() {
        vec4 origin_color = texture2D(texture, v_uv0);

        float width = 0.05;
        float start = run_time * 1.2;
        float strength = 0.01;
        float offset = 0.2;
        vec4 out_color = origin_color;

        if(v_uv0.x < (start - offset * v_uv0.y) && v_uv0.x > (start - offset * v_uv0.y - width)) {
            vec3 blendColor = strength * vec3(255,255,255);
            out_color = vec4((blendColor * vec3(origin_color.rgb)),out_color.a);
        }
        gl_FragColor = out_color;
    }
    `
}
```

## Mosaic(方形马赛克)
```js
这个没有要变量的 就是根据 周围像素区中进行模糊
```
```js
var shader = {
    name: "Mosaic",
    
    defines: [],

    vert: `uniform mat4 viewProj;
        attribute vec3 a_position;
        attribute vec2 a_uv0;
        varying vec2 uv0;
        void main () {
            vec4 pos = viewProj * vec4(a_position, 1);
            gl_Position = pos;
            uv0 = a_uv0;
        }
        `,

    frag: `uniform sampler2D texture;
    uniform vec3 imageSize;
    uniform float mosaicSize;
    varying vec2 uv0;
     
    void main(void)
    {
        vec4 color;
        vec2 xy = vec2(uv0.x * imageSize.x, uv0.y * imageSize.y);
        vec2 xyMosaic = vec2(floor(xy.x / mosaicSize) * mosaicSize, floor(xy.y / mosaicSize) * mosaicSize);
        vec2 uvMosaic = vec2(xyMosaic.x / imageSize.x, xyMosaic.y / imageSize.y);
        color = texture2D( texture, uvMosaic);
        gl_FragColor = color; 
    }`,
}
```

## Move(UV动画)
```js
// 要初始化的变量
this._time = 0;

// 要更新的变量
this._time += dt;
this._moveTime = this._time;
this._moveTime %= 3.35;
const material = this.sp.getCurrMaterial('Move');
if(!material) {
    return;
}
material.setParamValue('run_time', this._moveTime);
```
```js
var shader = {
    name: "Move",
    
    defines: [],

    vert: `uniform mat4 viewProj;
        attribute vec3 a_position;
        attribute vec2 a_uv0;
        varying vec2 uv0;
        void main () {
            vec4 pos = viewProj * vec4(a_position, 1);
            gl_Position = pos;
            uv0 = a_uv0;
        }
        `,

    frag: `uniform sampler2D texture;
    uniform float run_time;
    varying vec2 uv0;

    void main()
    {
        float x = uv0.x + run_time * 0.3;
        vec2 uv_o = vec2(x - floor(x),uv0.y);
        gl_FragColor = texture2D(texture, uv_o);
    }`,
}
```

Vortex(风暴)
```js
// 要初始化的变量
this._angle = 0;
this._radius = 0.1;
this._time = 0;

// 要不停更新的变量(就是要放在update函数里面的变量)
this._time += dt;
this._angle += 0.03;
this._radius += 0.003;

if(this._angle >= 5) {
    this._angle = 0;
    this._radius = 0.1;
    
}
const material = this.sp.getCurrMaterial('Vortex');
if(!material) {
    return;
}
material.setParamValue('_Radius',this._radius);
material.setParamValue('_Angle',this._angle);
```
```js
var shader = {
    name: "Vortex",
    
    defines: [],

    vert: `uniform mat4 viewProj;
        attribute vec3 a_position;
        attribute vec2 a_uv0;
        varying vec2 uv0;
        void main () {
            vec4 pos = viewProj * vec4(a_position, 1);
            gl_Position = pos;
            uv0 = a_uv0;
        }
        `,

    frag: `
    uniform sampler2D texture;
    uniform float _Radius;
    uniform float _Angle;
    varying vec2 uv0;

    vec2 swirl(vec2 uv)
    {
        //先减去贴图中心点的纹理坐标,这样是方便旋转计算
        uv -= vec2(0.5, 0.5);

        //计算当前坐标与中心点的距离。
        float dist = length(uv);

        //计算出旋转的百分比
        float percent = (_Radius - dist) / _Radius;

        if (percent < 1.0 && percent >= 0.0)
        {
            //通过sin,cos来计算出旋转后的位置。
            float theta = percent * percent * _Angle * 8.0;
            float s = sin(theta);
            float c = cos(theta);
            //uv = vec2(dot(uv, vec2(c, -s)), dot(uv, vec2(s, c)));
            uv = vec2(uv.x*c - uv.y*s, uv.x*s + uv.y*c);
        }

        //再加上贴图中心点的纹理坐标，这样才正确。
        uv += vec2(0.5, 0.5);

        return uv;
    }

    void main() {
        vec2 uv_new = swirl(uv0);
        gl_FragColor = texture2D(texture, uv_new);
    }
    `,
}
```

## Water(水)
```
// 要初始化的变量
this._time = 0;

// 要更新的变量
this._time += dt;
const material = this.sp.getCurrMaterial('Water');
if(!material) {
    return;
}
material.setParamValue('run_time', this._time);
```
```js
var shader = {
    name: "Water",
    
    defines: [],

    vert: `uniform mat4 viewProj;
        attribute vec3 a_position;
        attribute vec2 a_uv0;
        varying vec2 uv0;
        void main () {
            vec4 pos = viewProj * vec4(a_position, 1);
            gl_Position = pos;
            uv0 = a_uv0;
        }
        `,

    frag: `uniform sampler2D texture;
    uniform vec3 imageSize;
    uniform float run_time;
    varying vec2 uv0;

    #define F cos(x-y)*cos(y),sin(x+y)*sin(y)

    vec2 s(vec2 p)
    {
        float d=run_time*0.2,x=8.*(p.x+d),y=8.*(p.y+d);
        return vec2(F);
    }
    void mainImage( out vec4 fragColor, in vec2 fragCoord )
    {
        vec2 rs = imageSize.xy;
        vec2 uv = fragCoord;
        vec2 q = uv+2./imageSize.x*(s(uv)-s(uv+rs));
        // q.y=1.-q.y;
        fragColor = texture2D(texture, q);
    }
    void main()
    {
        mainImage(gl_FragColor, uv0.xy);
    }`,
}
```

## Wave(波浪)
```js
// 要初始化的变量
this._time = 0;

// 要更新的变量
this._time += dt;
const material = this.sp.getCurrMaterial('Wave');
if(!material) {
    return;
}
material.setParamValue('run_time', this._time);
```
```js
var shader = {
    name: "Wave",

    defines: [],

    vert: `uniform mat4 viewProj;
    attribute vec3 a_position;
    attribute vec2 a_uv0;

    varying vec2 v_uv0;

    void main() {
        vec4 pos = viewProj * vec4(a_position, 1.);
        gl_Position = pos;
        v_uv0 = a_uv0;
    }
    `,

    frag: `uniform sampler2D texture;
    /*uniform vec4 color;*/
    uniform float run_time;
    uniform vec2 offsetXY;
    uniform float speed;
    
    varying vec2 v_uv0;
    
    void main() {
        
        vec4 out_color;

        vec2 xy = v_uv0;
        xy.x += (sin(xy.x * 30. + run_time * speed) / 30. * offsetXY.x);
        xy.y += (sin(xy.y * 30. + run_time * speed) / 30. * offsetXY.y);

        gl_FragColor = texture2D(texture, xy);
    }
    `
}
```

> 还有一个简单的置灰效果没有写，这个因为Creator Engine 自带，所以就没有写。可以看一下`Sprite` 这个组件的API官方应该介绍的有， 置灰其实 rgb×0.3 就差不多了。

# 将所有的 逻辑整合到一个脚本里面 ShaderComponent

{% asset_link ShaderComponent.txt %}
> 由于后缀名字为 js 会导致博客渲染出错, 所以改为 txt 你们自行改之就行.
> 这里面会去 require 那几个着色器 你们自己将上面自定义的着色器代码 创建出来.

# 这么做会带来的一些问题
1.精灵运行渐变的Action 就会没有效果 因为这里只是采样纹理的颜色并做出一定的变换并没有融合节点本身的颜色值，如果想做透明变换，你可以给着色器传递值
2.在手机上可能会因为精度问题导致，渲染有问题， 你可以采取高精度的类型 如： #ifdef GL_ES highp float xx; #enif  还 中等精度 mediump lowp

如果有其他问题或则是意见可以加我QQ讨论。
