
#### cocos2dx js java oc 和 js 的相互调用

引擎版本: 3.17
语言: js
Xcode: 10.1
时间:2018年12月14日16:23:47

> 看这篇文章需要 会一些 android java oc ios 的一些知识的基础 不然看着可能有点费劲.

## js调用java

js 调用 java java中声名的方法需要是静态方法(static method)

##### Android 平台下

```
if(cc.sys.os == cc.sys.OS_ANDROID){
    jsb.reflection.callStaticMethod('类名加全路径','方法名字','方法的签名','传递参数');
}

ej: 
    jsb.reflection.callStaticMethod('org/cocos2dx/javascript/AppActivity','getStrVdong','(Ljava/lang/String;IZ)Ljava/lang/String;','haha',123,true);
    
其他方法签名的写法
ej:
    (Ljava/lang/String;)V //参数是字符串 没有返回值
    (I)V // void(int);
    ()V // void();
    (IZ)V // void(int,boolean);
    (IZ)Z // boolean(int,boolean);
    ()Ljava/lang/String; //String();
```

##### 支持传递的数据类型
+ Z 布尔
+ I int
+ F float
+ 字符串 Ljava/lang/String;

> 注意这里有个字符串是有个分号的

## java调用js

java调用js 实际是 将字符串转成 function 然后调用

```
/* 导入需要的包 */
import org.cocos2dx.lib.Cocos2dxJavascriptJavaBridge;
import org.cocos2dx.lib.Cocos2dxHelper;
/*
这里需要注意的是调用 js 代码需要运行在 GL 线程中

这里实现的逻辑是 将回调的逻辑移交到 js 代码层中, 这样就可以在 js 中 写逻辑 java 在合适的时机去执行这个调用 比如在做支付的时候 需要在之后响应后在 做游戏逻辑的处理.
*/
public static void callFuncVdong(final String code) {
    Cocos2dxHelper.runOnGLThread(new Runnable(){
        @Override
        public void run() {
            Cocos2dxJavascriptJavaBridge.evalString(code);
        }
    });
}
```

```
/*
写一下 js 调用的例子
*/
ej:
    jsb.reflection.callStaticMethod(
    "org/cocos2dx/javascript/AppActivity",
    "callFuncVdong",
    "(Ljava/lang/String;)V",
    "g_funcList.callFunc()"); /*呼叫一个全局函数*/
```

#### 问题记录
##### js调用 java 没反应

> 有很大的几率是 js调用中 方法签名没有写正确 你需要检查你的写法 具体的写法上面有写的.

---

## js 调用 oc

js 调用 oc 其实和 Java的写法是大同小异的

- 在 ios/AppController.h 中声名 静态函数

```
+(NSString *) showVdong:(NSString *)str title:(NSString *)tit;
```

- 在 ios/AppController.mm 文件中实现
```
@implementation

// 在这个之间实现函数

+(NSString *) showVdong:(NSString *)str title:(NSString *)tit {
    return @"haha";
}

@end
```

- js 调用用例
```
if(cc.sys.os == cc.sys.OS_IOS || cc.sys.os == cc.sys.OS_OSX) {
    var ret = jsb.reflection.callStaticMethod("AppController","showVdong:title","你是谁???","天呢");
    cc.log('****************',ret);
}
```

## oc 调用 js

- 在 ios/AppController.mm 文件中实现
```
#include "cocos/scripting/js-bindings/manual/ScriptingCore.h"

+(NSString *) showVdong:(NSString *)str title:(NSString *)tit {
    
    ScriptingCore::getInstance()->evalString("test");
    
    return @"haha";
}
```

- js 中的测试用例
```
var test = function() {
    cc.log("OC call JS success !!!");
}
```

参考地址(这个是CocosCreator原生游戏调用) > https://www.cnblogs.com/billyrun/articles/8529503.html