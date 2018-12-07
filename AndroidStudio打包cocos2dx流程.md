前几天闲着没事,去弄了一下AndroidStudio打包最新版 cocos2dx 系列原生平台的安装包.

引擎版本: 3.17
AndroidStudio版本: 3.2.1
SDK: 27
NDK: android-ndk-r16b
gradle: 4.1

首先提供这几个下载地址,有的可能需要翻墙,这个你们自己解决.

[gradle](http://services.gradle.org/distributions/)
android sdk android studio 自带有下载的管理器
[anroid ndk](https://developer.android.com/ndk/downloads/)
[android studio](https://developer.android.com/studio/)
[cocos2dx-系列](https://www.cocos.com/)

<!-- more -->

# 报错记录

#### Gradle sync failed: SSL peer shut down incorrectly

> 查这个路径下 gradle–>wrapper–>gradle-wrapper.properties

> 要下载 gradle 版本

> ej:distributionUrl=https\://services.gradle.org/distributions/gradle-4.1-all.zip

> [gradle下载地址](http://services.gradle.org/distributions/)

> 删除原先的资源

> 存放到下面路径下 C:\Users\Administrator\.gradle\wrapper\dists\gradle-4.1-all\bzyivzo6n839fup2jbap0tjew

> 记住一定要放到这个乱码的文件夹中

如果上面的步骤你你把android随机生成的文件夹(bzyivzo6n839fup2jbap0tjew) 删了 你可以执行下面的步骤

> D:\pro\cocos_pro\testCocosJs\frameworks\runtime-src\proj.android\gradlew.bat 执行这个脚本也可以.

执行你这里面的这个预处理脚本,它会重新生成一个类似于上面个文件夹,不要希望这个脚本能够下载好你想要的 gradle 版本.这回很慢,可以直接停掉这个批处理.
执行这一步的目的是为了生成 上面类似的文件夹(bzyivzo6n839fup2jbap0tjew).

当然如果你有更好的方案也可以.

---

#### No cached version of com.android.tools.build:gradle:3.0.0 available for offline mode.

> File/setting/build.../gradle/Offline work 禁掉

---

`如果 gradle 同步的很慢可以使用 镜像工程`

#### 替换proj.adnroid/build.gradle 中的地址

替换为镜像库

>maven { url 'https://maven.aliyun.com/repository/google' }

>maven { url 'https://maven.aliyun.com/repository/jcenter' }

>maven { url 'http://maven.aliyun.com/nexus/content/groups/public' }

我这个下面有截图
{% asset_img 1.png build.gradle 配置}

---

#### 官方提供的打包配置 地址

> [官方打包配置](http://docs.cocos2d-x.org/cocos2d-x/en/installation/Android-Studio.html)

---

#### No toolchains found in the NDK toolchains folder for ABI with prefix: mips64el-linux-android

> 在 ndk 的根目录里面有个 toolchains 的文件夹 里面没有 这个文件 mips64el-linux-android 原因是我用的是 r18 的 但是那里面没有这个东西,我去使用r16 里面就有这个东西.

---

#### ndk r9 下载路径

> 提供一下r9的下载路径 因为之前打包 3.15 是用的是 r9 所以这里记录一下地址 防止后面找不到了, 上面ndk 归档 只是到了 r10.

NDK r9d：

[r9-x86-window](http://dl.google.com/android/ndk/android-ndk-r9d-windows-x86.zip)
[r9-x86_64-window](http://dl.google.com/android/ndk/android-ndk-r9d-windows-x86_64.zip)
[r9-x86-mac]http://dl.google.com/android/ndk/android-ndk-r9d-darwin-x86.tar.bz2（Mac环境）
[r9-x86-linux](http://dl.google.com/android/ndk/android-ndk-r9d-linux-x86.tar.bz2)
[r9-x86_64-linux](http://dl.google.com/android/ndk/android-ndk-r9d-linux-x86_64.tar.bz2)
[r9d-cxx-stl-libs-with-debug-info](http://dl.google.com/android/ndk/android-ndk-r9d-cxx-stl-libs-with-debug-info.zip)

---

#### 配置NDK 环境变量
> ANDROID_NDK_ROOT

---

#### 编译externalNativeBuildDebug出错

> 可能出现路径太长导致 文件 查找失败(win10)
> 下面路径有个大神写了个补丁.

> [修复ndk打包长路径查找失败]https://discuss.cocos2d-x.org/t/the-solution-of-ndk-compile-system-longpaths-issue-on-windows-platform/42705

这个打包记录是纯净的打包没有介入任何 sdk(如,微信的sdk facebook 第三方广告平台(admob) 等);

我的每篇文章都不希望写的太长,不然看着都烦人. 这里之所以记录时间是为了 提醒一下 阅读者这里也许不是最新的,如果已过很长时间,那就给你提供,经验的借鉴吧.

时间: 2018年12月7日22:28:49