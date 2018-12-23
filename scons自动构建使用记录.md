[scons user guide](https://scons.org/doc/production/HTML/scons-user/)

### scons 普通简单的构建

```c++
// file name main.cpp
#include <iostream>
int main(int argc, char const *argv[])
{
    using namespace  std;
    cout<< "helloworld" << endl;

    return 0;
}
```

```python
#!/usr/bin/env python
Program('main.cpp')
```
### 多个文件构建

```python
#!/usr/bin/env python
# 构建多个文件
Program(['main.cpp','other.cpp'])

# 使用 匹配 的方式选择构建的的文件
# 可以使用 Glob()

Program(Glob('*.cpp'))

Glob() => 匹配的方式有个
- * 通配符
- ? 匹配任意一个字符
- [abc] 匹配任意一个符
- [!abc] 输了abc 之外的字符

Split函数 scons为了方便可以使用 分割函数一空格分割同样做到 多文件编译

Program("HelloWorld",Split("main.c other.c"))
Program("HelloWorld",Split("""
    main.c
    other.c"""))
允许使用 多行字符串

scons 也可以使用指定参数的形式调用

src_files = Split('main.c file1.c file2.c')
Program(target = 'program', source = src_files)

当然也可以反转参数
src_files = Split('main.c file1.c file2.c')
Program(source = src_files, target = 'program')
```

### Object 构建方法
```python
Object('main.c')

在 POSIX 系统上产生 *.o 文件
在 windows 系统上产生 *.obj 文件
```
### 清楚构建对中产生的对象
```bat
scons -c
```

### 让scons 简介输出
```bat
scons -Q
```

### 设置可执行文件的名字
```python
Program("HelloWorld","main.cpp")
在构建目录下产生 名字叫 HelloWorld.exe 的可执行文件
在 POSIX 系统上产生 HelloWorld 的可执行文件

如果不在 第一个参数 指定 可执行文件的名字 那么将会使用 第一个文件的名字 为可执行程序的名字
```

### 构建多个可执行程序
```python
Program('H1',['main.c'])
Program('H2',['main1.c'])

只需要调用多遍 Program 这个构造器就行了
```

### 多个可执行程序 使用共同的源文件
```python
common = ["common1.c", "common2.c"]
program1 = ["pro1.c"] + common
program2 = ["pro2.c"] + common

Program("pro1",program1)
Program("pro2",program2))
```

### 构建 Library(库)从源文件中
```python
Library('haha',['main.c'])

你可以混合构建最终的库 scons 可以识别出来那些需要编译 
这个只会编译 main.c
Library('haha',['main.c','common.o'])

构建静态库
StaticLibrary()
构建动态库
SharedLibrary()
```

### 链接库
```python
LIBS => 指定要链接的库的名字
LIBPATH => 要连接库名字的路径

Library('foo', ['f1.c', 'f2.c', 'f3.c'])
Program('prog.c', LIBS=['foo', 'bar'], LIBPATH='.')
这里可以简写 如果是一个 直接写库名字 不需要写成使用 列表 包一层

Library('foo', ['f1.c', 'f2.c', 'f3.c'])
Program('prog.c', LIBS='foo', LIBPATH='.')

多个库路径可以使用 如下格式
['/home/libs','/home/gg/libs']
'/home/libs:/home/gg/libs'(POSIX系统上的冒号：)
'C:\\libs;C:\\gg\\libs'(Windows系统上的分号;)
```

### 构造其方法返回节点列表
```python
这么做的目的是让我们写的 构建文件是平台的
不同平台 编译后的文件后缀不一样(a.obj(Windows), a.o(Lunix)) 所以 scons 帮我们处理了
返回节点列表 soncs 去处理这些差异
hello_list = Object('hello.c', CCFLAGS='-DHELLO')
goodbye_list = Object('goodbye.c', CCFLAGS='-DGOODBYE')

Program(hello_list + goodbye_list)

打印节点列表
print(hello_list[0]) // hello.obj(Windows)

判断文件是否存在
if not os.path.exists(program_name):
    print('不存在')
```

### 清晰区分 目录节点 和 文件节点
```python
hello_c = File('hello.c')
Program(hello_c)

目录节点
Dir()

不知道是 目录还是文件
Entry()
```

### 从一个节点或字符串获取路径
```python
env=Environment(VAR="value")
n=File("foo.c")
print(env.GetBuildPath([n, "sub/dir/$VAR"]))
```

### 重复构建 scons 可以识别出来 会出现 scons: `.' is up to date.
> 指出已经是最新的了

### 自定义 Decider('MD5')

> 自定义 重构规则 Decider('MD5') 默认是使用 md5 记录一个文件是够需要从新构建甚至 可以使用 'content' 代表同名词 md5 的缺陷是 如果修改了注释 scons 也认为 没有修改
> Decider('make') 只要文件的修改时间改变就从新构建 同义词 'timestamp-newer'(判定规则是 比上次构建的时间戳新 就从新构建 但是有的使用会碰到 文件还原 这样就会出现 之前构建的时候 比 现在的 新 下面的 记录规则 可以解决这个问题)
> 'timestamp-match' 匹配时间戳是否与构建的时候一样, 
> 'MD5-timestamp' 判定规则是  MD5 和 时间错都改变的时候 在重新构建 (如果在SCons构建文件的最后一秒内修改了源文件 这种情况可能不合适, 但是基本可能不存在)

### 编写自定义Decider函数
```python
Program('hello.c')
def decide_if_changed(dependency, target, prev_ni):
    if self.get_timestamp() != prev_ni.timestamp:
        dep = str(dependency)
        tgt = str(target)
        if specific_part_of_file_has_changed(dep, tgt):
            return True
    return False
Decider(decide_if_changed)

展开第三个参数
params :
    prev_ni :
        .csig The content signature, or MD5 checksum, of the contents of the dependency file the list time the target was built.
        .size The size in bytes of the dependency file the list time the target was built.
        .timestamp The modification time of the dependency file the list time the target was built.

当第一次构建的时候 这个参数 prev_ni 不存在

读这些东西的时候需要静心观看 不然就可能看不懂呦

env = Environment()

def config_file_decider(dependency, target, prev_ni):
    import os.path

    # We always have to init the .csig value...
    # 获取依赖文件的签名信息
    dep_csig = dependency.get_csig()
    # .csig may not exist, because no target was built yet...
    if 'csig' not in dir(prev_ni):
        return True
    # Target file may not exist yet
    if not os.path.exists(str(target.abspath)):
        return True
    if dep_csig != prev_ni.csig:
        # Some change on source file => update installed one
        return True
    return False

def update_file():
    f = open("test.txt","a")
    f.write("some line\n")
    f.close()

update_file()

# Activate our own decider function
env.Decider(config_file_decider)

env.Install("install","test.txt")


不同的环境使用不同 判定规则

env1 = Environment(CPPPATH = ['.'])
env2 = env1.Clone()
env2.Decider('timestamp-match')
env1.Program('prog-MD5', 'program1.c')
env2.Program('prog-timestamp', 'program2.c')
        
```

#### 隐式构造变量
``` c++
/* main.c */
#include <hello.h>
int main()
{
    printf("Hello, %s!\n", string);
}

/* hello.h */
#define string    "world"
```

```python
# Sconsturct
Program('hello.c', CPPPATH = '.')
也可以是多个 路径 也可以像 LIBPATH 那样
Program('hello.c', CPPPATH = ['include', '/home/project/inc'])
Window 分割符是 分号 (;) Linux 分隔符是 冒号(:)
Program('hello.c', CPPPATH = 'include:/home/project/inc')
```

### 缓存隐式依赖关系
```python
SetOption（'implicit_cache'，1）
```

或则是使用命令行 
> scons -Q --implicit-cache

> scons -Q --implicit-deps-changed 通知 scons 跟新隐式缓存 依赖的文件改变时

> scons -Q --implicit-deps-unchanged 强制及时依赖的文件改变时 仍然使用 缓存依赖

### 显示依赖关系

有的时候 scons 没有扫描(detected)到依赖关系的时候 scons 允许你 显示指定 依赖关系

```python
hello = Program('hello.c')
Depends(hello, 'other_file')

Depends 的第二个参数 也可以是节点 列表

hello = Program('hello.c')
goodbye = Program('goodbye.c')
Depends(hello, goodbye)
```

### 外部文件的依赖关系：ParseDepends 功能

```c++
#define FOO_HEADER <foo.h>
#include FOO_HEADER

int main() {
    return FOO;
}
```

上面这种情况 scons 可能无法 知道依赖关系

```python
obj = Object('hello.c', CCFLAGS='-MD -MF hello.d', CPPPATH='.')
SideEffect('hello.d', obj)
ParseDepends('hello.d')
 Program('hello', obj)
```

借助编译器 对宏的展开 并生成一个 *.d 的文件 第一次 scons 无法检测依赖 第二次 scons 可以根据生成的 *.d的文件确定依赖关系

### 忽略依赖项
```python
hello_obj=Object('hello.c')
hello = Program(hello_obj)
Ignore(hello_obj, 'hello.h')

上面的例子可能很难想象 它依赖的头文件 改变了 不从新编译
有一种情况 比如多个系统 使用同一个 头文件 每个系统的头文件 会有略微的差异
此时依赖的头文件改变了 但是不需要重新编译
hello = Program('hello.c', CPPPATH=['/usr/include'])
Ignore(hello, '/usr/include/stdio.h')

hello_obj=Object('hello.c')
hello = Program(hello_obj)
Ignore('.',[hello,hello_obj])
```

### AlwaysBuild
```python
hello = Program('hello.c')
AlwaysBuild(hello)
```
### 构造环境feature的使用
```python
创建新的 构造环境
env = Environment()

一个简单的例子 设置 c编译器
 env = Environment(CC = 'gcc', CCFLAGS = '-O2')
 env.Program('foo.c')

访问构造环境的各个变量  (它是个具有关联方法的对象)
env = Environment()
print("CC is: %s"%env['CC'])

如果你仅仅是访问他的构造变量,你可以这样做 调用构造环境的 Dictionary 方法
env = Environment(FOO = 'foo', BAR = 'bar')
dict = env.Dictionary()
for key in ['OBJSUFFIX', 'LIBSUFFIX', 'PROGSUFFIX']:
    print("key = %s, value = %s" % (key, dict[key]))

on Windows > OBJSUFFIX => .obj LIBSUFFIX => .lib PROGSUFFIX => .exe
on Linux > OBJSUFFIX => .o LIBSUFFIX => .a PROGSUFFIX => 

获取构造环境中的变量 方法2
使用 subst() 方法 使用 $ + 变量字段 的方式访问
使用这个方法的好处是 它可以将构造变量展开
env = Environment()
print("CC is: %s"%env.subst('$CC'))
# $CCCOM

处理 变量 展开带来的问题
如果访问一个不存在的变量是 scons 并不会报错 
如果想要报错你可以这样做
AllowSubstExceptions() 开始 使用 subst 函数异常的功能 这是个全局变量

除了定义默认的异常外 还可以定义 其他异常如 零除异常
AllowSubstExceptions(IndexError, NameError, ZeroDivisionError)
env = Environment()
print("value is: %s"%env.subst( '->${1 / 0}<-' ))

创建默认的 构造环境 可以设置 目标编译器
DefaultEnvironment(CC = '/usr/local/bin/gcc')

当然也可以这样做
env = DefaultEnvironment()
env['CC'] = '/usr/local/bin/gcc'

使用 DefaultEnvironment() 可以加快你初始化构造变量
在初始化时 scons 会在 系统的环境变量中搜索 可用的 编译器 连接器等 你可以指定 搜索的 工具 加快scons的初始化速度 你可以这样做

直接指定 编译工具 链接工具 指定 c编译器 为gcc
env = DefaultEnvironment(tools = ['gcc', 'gnulink'],
                         CC = '/usr/local/bin/gcc')

多构造环境 你可以这样做 不同的构造环境 使用不同的配置
opt = Environment(CCFLAGS = '-O2')
dbg = Environment(CCFLAGS = '-g')

opt.Program('foo', 'foo.c')
dbg.Program('foo', 'foo.c')
--------------------------------------
产生编译产物 *.obj(on Windows)
opt = Environment(CCFLAGS = '-O2')
dbg = Environment(CCFLAGS = '-g')

o = opt.Object('foo-opt', 'foo.c')
opt.Program(o)

d = dbg.Object('foo-dbg', 'foo.c')
dbg.Program(d)

克隆当前的环境变量
克隆一份一模一样的构造环境
env = Environment(CC = 'gcc')
opt = env.Clone(CCFLAGS = '-O2')
dbg = env.Clone(CCFLAGS = '-g')
env.Program('foo', 'foo.c')
o = opt.Object('foo-opt', 'foo.c')
opt.Program(o)
d = dbg.Object('foo-dbg', 'foo.c')
dbg.Program(d)

替换构造环境
env = Environment(CCFLAGS = '-DDEFINE1')
env.Replace(CCFLAGS = '-DDEFINE2')
env.Program('foo.c')

仅在尚未定义的情况下设置值
env.SetDefault(SPECIAL_FLAG = '-extra-option')

附加到构造变量末尾
env = Environment(CCFLAGS = ['-DMY_VALUE'])
env.Append(CCFLAGS = ['-DLAST'])
env.Program('foo.c')

附加唯一值
Some times it's useful to add a new value only if the existing construction variable doesn't already contain the value. This can be done using the AppendUnique method:
官方解释: 有时 它是有用的 在这个构造变量没有包含这个变量的时候添加一个新值 , 这样可以使使用这个 方法做
env.AppendUnique(CCFLAGS=['-g'])

In the above example, the -g would be added only if the $CCFLAGS variable does not already contain a -g value.
解释: 在上面这个例子, -g 标识将被添加 仅仅这个 $CCFLAGS 变量曾没有包含这个 -g 的标识

附加到值的开头
env = Environment(CCFLAGS = ['-DMY_VALUE'])
env.Prepend(CCFLAGS = ['-DFIRST'])
env.Program('foo.c')

附加唯一值到开头 和上面在结尾附加值 的 逻辑是一样的
env.PrependUnique（CCFLAGS = [ ' -  G']）
```

### 环境变量影响 scons 构建
it uses the dictionary stored in the $ENV construction variable as the external environment for executing commands.
解释: 它使用成字典的方式存储在 ENV 的构造变量中 作为外部可执行命令的环境

```python
添加 Path 环境变量到 ENV中执行
path = ['/usr/local/bin', '/bin', '/usr/bin']
env = Environment(ENV = {'PATH' : path})

env['ENV']['PATH'] = ['/usr/local/bin', '/bin', '/usr/bin']
env['ENV']['PATH'] = '/usr/local/bin:/bin:/usr/bin'

因为是 scons 的配置文件是以 python 的环境执行的 所以 你可以使用 os.environ['PATH'] 获取环境变量, 例子如下:
这样做,可以让自己的代码可移植,跨平台
import os
env = Environment(ENV = {'PATH' : os.environ['PATH']})

或者你可以将整个 环境变量 全部赋值给 ENV
import os
env = Environment(ENV = os.environ)

你可以在已有的环境变量中 拼接 scons 提供如下的 方法:
PrependENVPath()
AppendENVPath()
env =环境(ENV = os.environ)
env.PrependENVPath('PATH'，'/ usr / local / bin')
env.AppendENVPath('LIB'，'/ usr / local / lib')
```

### 使用外部工具
使用外部工具的时候 scons 默认的搜索路径:
scons 内置工具目录
Sconstruct 同级目录下/site_scons/site_tools/

```python
# Builtin tool or tool located within site_tools
env = Environment(tools = ['SomeTool'])
env.SomeTool(targets, sources)

# The search locations would include by default
SCons/Tool/SomeTool.py
SCons/Tool/SomeTool/__init__.py
./site_scons/site_tools/SomeTool.py
./site_scons/site_tools/SomeTool/__init__.py

也可以给 添加 除了 scons 默认搜索路径
# Tool located within the toolpath directory option
env = Environment(tools = ['SomeTool'], toolpath = ['/opt/SomeToolPath', '/opt/SomeToolPath2'])
env.SomeTool(targets, sources)

# The search locations in this example would include:
/opt/SomeToolPath/SomeTool.py
/opt/SomeToolPath/SomeTool/__init__.py
/opt/SomeToolPath2/SomeTool.py
/opt/SomeToolPath2/SomeTool/__init__.py
SCons/Tool/SomeTool.py
SCons/Tool/SomeTool/__init__.py
./site_scons/site_tools/SomeTool.py
./site_scons/site_tools/SomeTool/__init__.py

嵌套工具
# namespaced target
env = Environment(tools = ['SubDir1.SubDir2.SomeTool'], toolpath = ['/opt/SomeToolPath'])
env.SomeTool(targets, sources)

# With this example the search locations would include
/opt/SomeToolPath/SubDir1/SubDir2/SomeTool.py
/opt/SomeToolPath/SubDir1/SubDir2/SomeTool/__init__.py
SCons/Tool/SubDir1/SubDir2/SomeTool.py
SCons/Tool/SubDir1/SubDir2/SomeTool/__init__.py
./site_scons/site_tools/SubDir1/SubDir2/SomeTool.py
./site_scons/site_tools/SubDir1/SubDir2/SomeTool/__init__.py

使用 sys.path 作为扩展的工具路径
# namespaced target using sys.path within toolpath
import sys
searchpaths = []
for item in sys.path:
    if os.path.isdir(item): searchpaths.append(item)

env = Environment(tools = ['someinstalledpackage.SomeTool'], toolpath = searchpaths)
env.SomeTool(targets, sources)

你可以使用这个包 来避免使用 pip 安装包 拼接前面复杂的路径
# namespaced target using sys.path
import sys
env = Environment(tools = ['SomeTool'], toolpath = [PyPackageDir('tools_example.subdir1.subdir2')])
env.SomeTool(targets, sources)
```


---

这次你了解 构造变量的这几个方法
MergeFlags, ParseFlags, and ParseConfig methods of a construction environment
MergeFlags, ParseFlags ParseConfig

### 
```python
MergeFlags
尝试合并某个构造变量 例子如下:

env = Environment()
env.Append(CCFLAGS = '-option -O3 -O1')
flags = { 'CCFLAGS' : '-whatever -O3' }
env.MergeFlags(flags)
print env['CCFLAGS']

>>>>
scons -Q
['-option', '-O1', '-whatever', '-O3']
scons: '.' is up to date.

env = Environment()
env.Append(CPPPATH = ['/include', '/usr/local/include', '/usr/include'])
flags = { 'CPPPATH' : ['/usr/opt/include', '/usr/local/include'] }
env.MergeFlags(flags)
print env['CPPPATH']

>>>>
scons -Q
['/include', '/usr/local/include', '/usr/include', '/usr/opt/include']
scons: '.' is up to date.

如果我们这个方法传递的不是列表 scons 会自动调用 ParseFlags 将其解析成列表
env = Environment()
env.Append(CCFLAGS = '-option -O3 -O1')
env.Append(CPPPATH = ['/include', '/usr/local/include', '/usr/include'])
env.MergeFlags('-whatever -I/usr/opt/include -O3 -I/usr/local/include')
print env['CCFLAGS']
print env['CPPPATH']

>>>>
scons -Q
['-option', '-O1', '-whatever', '-O3']
['/include', '/usr/local/include', '/usr/include', '/usr/opt/include']
scons: `.' is up to date.

ParseFlags 
[ParseFlags](https://scons.org/doc/production/HTML/scons-user/ch08s02.html)
讲一个有规律的字符串 解析成一个字典列表
-I 指要包含的 源码路径
-L 库文件的路径
env = Environment()
d = env.ParseFlags("-I/opt/include -L/opt/lib -lfoo")
for k,v in sorted(d.items()):
    if v:
        print k, v
env.MergeFlags(d)
env.Program('f1.c')

支持递归输入 主要是看例子 和输出结果去理解这些东西, 这里官方也没有解释的太清楚 后面还得多去实践的真知
env = Environment()
d = env.ParseFlags(["-I/opt/include", ["-L/opt/lib", "-lfoo"]])
for k,v in sorted(d.items()):
    if v:
        print k, v
env.MergeFlags(d)
env.Program('f1.c')
>>>>
% scons -Q
CPPPATH ['/opt/include']
LIBPATH ['/opt/lib']
LIBS ['foo']
cc -o f1.o -c -I/opt/include f1.c
cc -o f1 f1.o -L/opt/lib -lfoo

ParseConfig
这个东西没有看明白 这里先记录一下链接
Finding Installed Library Information: the ParseConfig Function
[ParseConfig](https://scons.org/doc/production/HTML/scons-user/ch08s03.html)
```

### 在构建程序的时候 帮助信息
```python
Help("""
Type: 'scons program' to build the production program,
      'scons debug' to build the debug version.
""", append=True)

加上 append = True 在下面出现 `Use scons -H for help about command-line options.` 这样的字段

>>>>

scons -h
scons: Reading SConscript files ...
scons: done reading SConscript files.

Type: 'scons program' to build the production program,
      'scons debug' to build the debug version.

Use scons -H for help about command-line options.

你可以缩短的 命令输出 如下这种操作
env = Environment(CCCOMSTR = "Compiling $TARGET",
                  LINKCOMSTR = "Linking $TARGET")
env.Program('foo.c')

>>>>
% scons -Q
Compiling foo.o
Linking foo

比如下面的例子 如果用户输入的 详细字段不为 True 则自定义 编译命令 其实是简短编译命令
env = Environment()
if ARGUMENTS.get('VERBOSE') != "1':
    env['CCCOMSTR'] = "Compiling $TARGET"
    env['LINKCOMSTR'] = "Linking $TARGET"
env.Program('foo.c')

Progress 控制流程输出
[Progress](https://scons.org/doc/production/HTML/scons-user/ch09s03.html)

GetBuildFailures
[打印详细的错误信息](https://scons.org/doc/production/HTML/scons-user/ch09s04.html)
```

### 从命令行控制构建流程

### InstallBuilder 安装构建的可执行程序

### 与平台无关的文件操作

### 控制构建目标的移除

### 分层构建

### 分离构建 目标 和 源文件

### 变体构建

### 国际化

### 编写自己的 builder(构造器)

### 缓存构建的文件

### 验证 python scons 版本

### 别名目标

### 多平台配置（Autoconf功能）

### 从代码存储库构建

### 编写扫描仪

### Not Writing a Builder: the Command Builder

### Pseudo-Builders: the AddMethod function(伪造构建起, 添加方法)

