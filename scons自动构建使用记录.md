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
[传送门](https://scons.org/doc/production/HTML/scons-user/ch10.html)
```python
>>>> 如果你想使用者调用一些除了scons提供的指令,你可以这样做

ARGUMENTS 这个东西将会帮到你.
但是有个缺陷是不能存储两个值,只能 key-value 的形式.
ARGUMENTS.get() 获取 debug 这个指令 没有用后面的默认值

env = Environment()
debug = ARGUMENTS.get('debug', 0)
if int(debug):
    env.Append(CCFLAGS = '-g')
env.Program('prog.c')

ej:
scons -Q debug=0

>>>> 如果你想 一对多 你可以使用这个东西
ARGLIST

cppdefines = []
for key, value in ARGLIST:
    if key == 'define':
        cppdefines.append(value)
env = Environment(CPPDEFINES = cppdefines)
env.Object('prog.c')

ej: scons -Q define=FOO define=BAR

>>>> 从文件中读取用户构建配置
>>>> 会读取 custom.py 文件的配置表
vars = Variables('custom.py')
vars.Add('RELEASE', 'Set to 1 to build for release', 0)
env = Environment(variables = vars,
                  CPPDEFINES={'RELEASE_BUILD' : '${RELEASE}'})
env.Program(['foo.c', 'bar.c'])
Help(vars.GenerateHelpText(env))

>>>> custom.py
>>>> RELEASE=1

>>>> 可以结合 两种做法 读取用户 在命令行输入或则是配置文件
>>>> 其中 本地配置文件会首先 覆盖 命令行 的 配置
vars = Variables('custom.py',ARGUMENTS)

>>>> 定义构造变量的函数
>>>> Bool 可以指定的值有 true => t on all 1 true    False => f 0 no false

vars = Variables('custom.py')
vars.Add(BoolVariable('RELEASE', 'Set to build for release', 0))
env = Environment(variables = vars,
                  CPPDEFINES={'RELEASE_BUILD' : '${RELEASE}'})
env.Program('foo.c')

ej:
scons -Q RELEASE=yes foo.o

>>>> Enum 有个 allowed_values 的字段 定义枚举变量值
>>>> map 字段备用映射名 map={'navy':'blue'},
>>>> ignorecase=1 忽略 指定指令的大小写  如 COLOR=NaVy

vars = Variables('custom.py')
vars.Add(EnumVariable('COLOR', 'Set background color', 'red',
                    allowed_values=('red', 'green', 'blue')))
env = Environment(variables = vars,
                  CPPDEFINES={'COLOR' : '"${COLOR}"'})
env.Program('foo.c')

ej: scons -Q COLOR=red

>>>> ListVariable 允许一个字段指定多个值
>>>> 允许使用 all 或则是 none

vars = Variables('custom.py')
vars.Add(ListVariable('COLORS', 'List of colors', 0,
                    ['red', 'green', 'blue']))
env = Environment(variables = vars,
                  CPPDEFINES={'COLORS' : '"${COLORS}"'})
env.Program('foo.c')

ej: scons -Q COLORS=red,blue foo.o => cc -o foo.o -c -DCOLORS="red blue" foo.c
ej: scons -Q COLORS=all foo.o => cc -o foo.o -c -DCOLORS="red green blue" foo.c
ej: scons -Q COLORS=none foo.o => cc -o foo.o -c -DCOLORS="" foo.c

>>>> PathVariable 路径变量
>>>> 提供输入的路径是文件 PathVariable.PathIsFile
>>>> 是目录 PathVariable.PathIsDir
>>>> 目录不存在,并创建 PathVariable.PathIsDirCreate
>>>> 不关心上面的三点 PathVariable.PathAccept

ej:
vars = Variables('custom.py')
vars.Add(PathVariable('CONFIG',
                    'Path to configuration file',
                    '/etc/my_config',
                    PathVariable.PathIsFile))
env = Environment(variables = vars,
                  CPPDEFINES={'CONFIG_FILE' : '"$CONFIG"'})
env.Program('foo.c')

>>>> PackageVariable
>>>> 启用或禁止 路径名字 没有太懂
vars = Variables('custom.py')
vars.Add(PackageVariable('PACKAGE',
                       'Location package',
                       '/opt/location'))
env = Environment(variables = vars,
                  CPPDEFINES={'PACKAGE' : '"$PACKAGE"'})
env.Program('foo.c')

ej:
% scons -Q foo.o
cc -o foo.o -c -DPACKAGE="/opt/location" foo.c
% scons -Q PACKAGE=/usr/local/location foo.o
cc -o foo.o -c -DPACKAGE="/usr/local/location" foo.c
% scons -Q PACKAGE=yes foo.o
cc -o foo.o -c -DPACKAGE="True" foo.c
% scons -Q PACKAGE=no foo.o
cc -o foo.o -c -DPACKAGE="False" foo.c

>>>> AddVariables 一次性添加多个变量功能

vars = Variables()
vars.AddVariables(
    ('RELEASE', 'Set to 1 to build for release', 0),
    ('CONFIG', 'Configuration file', '/etc/my_config'),
    BoolVariable('warnings', 'compilation with -Wall and similiar', 1),
    EnumVariable('debug', 'debug output and symbols', 'no',
               allowed_values=('yes', 'no', 'full'),
               map={}, ignorecase=0),  # case sensitive
    ListVariable('shared',
               'libraries to build as shared libraries',
               'all',
               names = list_of_libs),
    PackageVariable('x11',
                  'use X11 installed here (yes = search some places)',
                  'yes'),
    PathVariable('qtdir', 'where the root of Qt is installed', qtdir),
)

>>>> UnknownVariables 在用户使用了未定义的 指令时 你可以警告调用者 拼写错误 并退出程序

vars = Variables(None)
vars.Add('RELEASE', 'Set to 1 to build for release', 0)
env = Environment(variables = vars,
                  CPPDEFINES={'RELEASE_BUILD' : '${RELEASE}'})
unknown = vars.UnknownVariables()
if unknown:
    print("Unknown variables: %s"%unknown.keys())
    Exit(1)
env.Program('foo.c')

>>>> 你可以在用户使用某个命令的时候 提醒用户一些操作
if 'bar' in COMMAND_LINE_TARGETS:
    print("Don't forget to copy `bar' to the archive!")
Default(Program('foo.c'))
Program('bar.c')

ej:
% scons -Q bar
Don't forget to copy `bar' to the archive!

>>>> Default() 用户没有指定构建是 可以使用 这个函数 默认 构建一个程序
env = Environment()
hello = env.Program('hello.c')
env.Program('goodbye.c')
Default(hello)

ej:
    % scons -Q
    cc -o hello.o -c hello.c
    cc -o hello hello.o
    % scons -Q
    scons: `hello' is up to date.
    % scons -Q goodbye
    cc -o goodbye.o -c goodbye.c
    cc -o goodbye goodbye.o
Default 跟过用法 [传送门](https://scons.org/doc/production/HTML/scons-user/ch10s03.html)

>>>> DEFAULT_TARGETS 主要是迎合上面 Default 函数, 看一下例子应该很容易理解
prog1 = Program('prog1.c')
Default(prog1)
print("DEFAULT_TARGETS is %s"%map(str, DEFAULT_TARGETS))

ej:
% scons
scons: Reading SConscript files ...
DEFAULT_TARGETS is ['prog1']
scons: done reading SConscript files.
scons: Building targets ...
cc -o prog1.o -c prog1.c
cc -o prog1 prog1.o
scons: done building targets.

>>>> BUILD_TARGETS
prog1 = Program('prog1.c')
Program('prog2.c')
Default(prog1)
print ("BUILD_TARGETS is %s"%map(str, BUILD_TARGETS))

ej:
% scons -Q
BUILD_TARGETS is ['prog1']
cc -o prog1.o -c prog1.c
cc -o prog1 prog1.o
% scons -Q prog2
BUILD_TARGETS is ['prog2']
cc -o prog2.o -c prog2.c
cc -o prog2 prog2.o
% scons -Q -c .
BUILD_TARGETS is ['.']
Removed prog1.o
Removed prog1
Removed prog2.o
Removed prog2

>>>> 产生帮助文档(help)
vars = Variables(None, ARGUMENTS)
vars.Add('RELEASE', 'Set to 1 to build for release', 0)
env = Environment(variables = vars)
Help(vars.GenerateHelpText(env))
```

### InstallBuilder 安装构建的可执行程序
安装暂时没有 显示功能上需求就不做详细叙述了
[传送门](https://scons.org/doc/production/HTML/scons-user/ch11.html)

### 与平台无关的文件操作
scons 封装了 与平台无关的文件操作
如: 拷贝(copy),移动(move),移除(delete),创建(touch),创建目录(mkdir),执行(execute)
[传送门](https://scons.org/doc/production/HTML/scons-user/ch12.html)WWW

### 控制构建目标的移除
```python
>>>> 有的时候你想要在每次构建一个东西的时候,不清除上一次的构建,而是以增量的方式构建
>>>> Precious
env = Environment(RANLIBCOM='')
lib = env.Library('foo', ['f1.c', 'f2.c', 'f3.c'])
env.Precious(lib)

>>>> 有的时候你可能使用 scons -c 但是这个命令会将所有的构建产物都移除掉,你可以使用这个命令,保证有些东西不被 -c 指令 给移除掉
>>>> NoClean
env = Environment(RANLIBCOM='')
lib = env.Library('foo', ['f1.c', 'f2.c', 'f3.c'])
env.NoClean(lib)

>>>> Clean 有的时候你可能 清楚自己在产生过程中生成的 自定义文件 你可以这样做
t = Command('foo.out', 'foo.in', 'build -o $TARGET $SOURCE')
Clean(t, 'foo.log')

ej:
% scons -Q
build -o foo.out foo.in
% scons -Q -c
Removed foo.out
Removed foo.log


```
### 分级构建
分级构建,就是你的源文件可能不是一个目录,需要多一个目录构建.
[传送门](https://scons.org/doc/production/HTML/scons-user/ch14.html)
```python
>>>> SConscript 根目录调用每个子目录的 构建脚本, 通常子目录的构建脚本叫 SConscript 这是官方的解释
SConscript(['drivers/display/SConscript',
            'drivers/mouse/SConscript',
            'parser/SConscript',
            'utilities/SConscript'])

>>>> 通常里面的路径都是相对于当前这个构建脚本 如果你先让子构建脚本 使用顶级目录你可以这样做 加上 '#' 这个符号
env = Environment()
env.Program('prog', ['main.c', '#lib/foo1.c', 'foo2.c'])

>>>> 当然你也可以直接使用绝对路径 不需要加任何符号
>>>> 
>>>> 你可以在多个构建脚本共享变量 你可以使用 Export Import 这两个方法

ej:
env = Environment()
Export(env)

or:
env = Environment()
debug = 10
Export(env, debug)

or:
Export('env debug')

or:
SConscript('src/SConscript', 'env')

or:
SConscript('src/SConscript', exports='env')

ej:
Import(env)

Import(env, debug)

Import('env debug')

Import('*')

>>>> 有事你需要 从两个此目录返回结果 构建出一个对象
env = Environment()
Export('env')
objs = []
for subdir in ['foo', 'bar']:
    o = SConscript('%s/SConscript' % subdir)
    objs.append(o)
env.Library('prog', objs)

other Sconstruct(其他构建脚本):
    Import('env')
    obj = env.Object('foo.c')
    Return('obj')

>>>> 
```
### 分离构建 目标 和 源文件
分离构建结果
[传送门](https://scons.org/doc/production/HTML/scons-user/ch15.html)
```python
方式1:
主要是利用这个字段 variant_dir
SConscript('src/SConscript', variant_dir='build')

禁掉在输出目录拷贝源文件
SConscript（'src / SConscript'，variant_dir ='build'，duplicate = 0）
方式2:
VariantDir('build', 'src', duplicate=0)
env = Environment()
env.Program('build/hello.c')

子脚本
VariantDir('build', 'src')
SConscript('build/SConscript')
```

### 变体构建
[传送门](https://scons.org/doc/production/HTML/scons-user/ch16.html)
同一个目录构建不同的目标产物(可执行程序)
```python
platform = ARGUMENTS.get('OS', Platform())

include = "#export/$PLATFORM/include"
lib = "#export/$PLATFORM/lib"
bin = "#export/$PLATFORM/bin"

env = Environment(PLATFORM = platform,
                  BINDIR = bin,
                  INCDIR = include,
                  LIBDIR = lib,
                  CPPPATH = [include],
                  LIBPATH = [lib],
                  LIBS = 'world')

Export('env')

env.SConscript('src/SConscript', variant_dir='build/$PLATFORM')
```


### 国际化
[传送门](https://scons.org/doc/production/HTML/scons-user/ch17.html)

### 编写自己的 builder(构造器)
[传送门](https://scons.org/doc/production/HTML/scons-user/ch18.html)
```python
>>>> 创建构造器
env = Environment()
bld = Builder(action = 'foobuild < $SOURCE > $TARGET')
env.Append(BUILDERS = {'Foo' : bld})
env.Foo('file.foo', 'file.input')
env.Program('hello.c')

or:
env = Environment()
bld = Builder(action = 'foobuild < $SOURCE > $TARGET')
env['BUILDERS']['Foo'] = bld
env.Foo('file.foo', 'file.input')
env.Program('hello.c')

>>>> 指定构建的文件后缀 和 输出文件的后缀
bld = Builder(action = 'foobuild < $SOURCE > $TARGET',
              suffix = '.foo',
              src_suffix = '.input')
env = Environment(BUILDERS = {'Foo' : bld})
env.Foo('file1')
env.Foo('file2')

>>>> 构建是调用 python 命令 
target:
A list of Node objects representing the target or targets to be built by this builder function. The file names of these target(s) may be extracted using the Python str function.
(是一个目标节点列表,可通过 str 的方法 解析里面的内容)
source:
A list of Node objects representing the sources to be used by this builder function to build the targets. The file names of these source(s) may be extracted using the Python str function.
(是一个源节点列表,可通过 str 的方法 解析里面的内容)
env:
The construction environment used for building the target(s). The builder function may use any of the environment's construction variables in any way to affect how it builds the targets.
(这个构造变量被用在 目标节点上, 这个构造器可能使用任意的构造环境中的构造变量以任意的方式去影响如何构建目标节点)
(这个解释可能有点绕,其实还是挺容易懂得,不是吗?)

>>>> 自定义构建函数 上面是三个参数的解释, 很详细.
def build_function(target, source, env):
    # Code to build "target" from "source"
    return None
bld = Builder(action = build_function,
              suffix = '.foo',
              src_suffix = '.input')
env = Environment(BUILDERS = {'Foo' : bld})
env.Foo('file')

>>>> Generator 产生一条记录信息
def generate_actions(source, target, env, for_signature):
    return 'foobuild < %s > %s' % (source[0], target[0])
bld = Builder(generator = generate_actions,
              suffix = '.foo',
              src_suffix = '.input')
env = Environment(BUILDERS = {'Foo' : bld})
env.Foo('file')

>>>> Builders That Modify the Target or Source Lists Using an Emitter
(修改目标节点或者是 源节点 通过发射器)
bld = Builder(action = 'my_command $SOURCES > $TARGET',
              suffix = '.foo',
              src_suffix = '.input',
              emitter = '$MY_EMITTER')
def modify1(target, source, env):
    return target, source + ['modify1.in']
def modify2(target, source, env):
    return target, source + ['modify2.in']
env1 = Environment(BUILDERS = {'Foo' : bld},
                   MY_EMITTER = modify1)
env2 = Environment(BUILDERS = {'Foo' : bld},
                   MY_EMITTER = modify2)
env1.Foo('file1')
env2.Foo('file2')

>>>> 放置自定义工具或者是 构造器
>>>> scons 会自动加载 site_scons/site_init.py 比任意一个 构建脚本都早
>>>> site_scons/site_tools 在顶级的构建脚本是最后执行的 它覆盖其他的构建脚本定义
>>>> 
site_scons/site_init.py

def TOOL_ADD_HEADER(env):
   """A Tool to add a header from $HEADER to the source file"""
   add_header = Builder(action=['echo "$HEADER" > $TARGET',
                                'cat $SOURCE >> $TARGET'])
   env.Append(BUILDERS = {'AddHeader' : add_header})
   env['HEADER'] = '' # set default value

SConstruct
# Use TOOL_ADD_HEADER from site_scons/site_init.py
env=Environment(tools=['default', TOOL_ADD_HEADER], HEADER="=====")
env.AddHeader('tgt', 'src')

site_scons/my_utils.py

from SCons.Script import *   # for Execute and Mkdir
def build_id():
   """Return a build ID (stub version)"""
   return "100"
def MakeWorkDir(workdir):
   """Create the specified dir immediately"""
   Execute(Mkdir(workdir))

import my_utils
print("build_id=" + my_utils.build_id())
my_utils.MakeWorkDir('/tmp/work')
```
### 缓存构建的文件

### 验证 python scons 版本

### 别名目标

### 多平台配置（Autoconf功能）

### 从代码存储库构建

### 编写扫描仪

### Not Writing a Builder: the Command Builder

### Pseudo-Builders: the AddMethod function(伪造构建起, 添加方法)

