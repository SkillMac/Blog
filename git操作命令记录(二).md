时间: 2019年1月23日18:51:03

后面的格式是,在每篇文章开头都会加上时间,好比较你现在的时间和这篇博客书写的时间,你好判定你是借鉴还是可以直接用.

今天主要是介绍 git 的其他命令.
* git 仓库打标签(tag)
* git 扩展工具 lfs(large file system)

<!-- more -->

## git tag

### 给分支打标签
    git tag <tagName>
    这个命令是给本地仓库最近的一次提交打个标签.

ej:
```
git tag 1.0.0
```

### 获取本地所有的标签
    git tag

ej:
```
git tag
> 1.0.0
```

### 删除本地标签
    git tag -d <tagName>

ej:
```
git tag -d 1.0.0
```

### 给标签添加描述信息
    git tag -a <tagName> -m "<decription>"

ej:
```
git tag -a 1.0.1 -m "online version1.0.1"
```

### 给标签添加描述信息
    git tag -a <tagName> -m "<decription>"

ej:
```
git tag -a 1.0.1 -m "online version1.0.1"
```

### 给指定的提交打上标签
    git tag <tagName> <commitId>

>这里查询相关的 提交Id(commitId) 可以使用 git log 去看, 但是显示的 commitId 太长了不太好看
>可以使用下面的命令查看 git log --abbrev-commit 其实给git的 commitId 七位就可以锁定一次提交记录.

ej:
```
git tag 1.0.2 7b23fc5
```

### 显示标签的详细情况
    git show <tagName>

ej:
```
git show 1.0.2
```

#### 上面这些都是本地的操作,你肯定还需要将你这些,tag提交上去.

### 推送指定标签到远端
    git push origin <tagName>

ej:
```
git push origin 1.0.1
```

### 推送本地所有的标签
    git push origin --tags

ej:
```
git push origin --tags
```

### 删除远端标签
    git push origin :refs/tags/<tagName>
    git push origin --delete tag <tagName>

ej:
```
git push origin :refs/tags/1.0.1
git push origin --delete tag 1.0.1
```

也许有些人会疑惑,怎么去查看远程的 tag,这个你就不用纠结了,因为每次`git pull`的时候,都会将别人推送上去的 tag 给拉取到本地再次使用 git tag 查看就行了.

>删除远端 tag 的工作流(workflow) 先删除本地的tag 然后在删除远端的 tag.

## git lfs
[git-lfs参考地址](https://git-lfs.github.com/)
{% asset_img graphic.gif git-lfs示意图 %}
这个扩展工具是解决 主仓库日渐庞大的时候 拉取缓慢的问题.

每次提交都会使仓库的总体积变的原来越大.尤其是一些大的项目这一点会体现的越来越明显,导致别人在拉取的时候会变得很缓慢.

git lfs 的解决方案是 将一些大的文件 储存在另外的仓库中,保证你的主仓库的体积不会很大.正如,git-lfs 中说到的 更快的拉取速度.但是它并没有解决仓库体积的问题.

### git-lfs 的工作流(workflow)
1.先去安装 这个扩展工具.
2.然后在你的仓库目录下,打开 git 执行 `git lfs install`
3.添加你要追踪的大尺寸文件,它这里支持模糊匹配 `git lfs track "*.extName"`
4.执行 `git lfs install`的时候会产生一个.gitattributes的文件 确保他被提交被追踪 `git add .gitattributes`
5. 使用 `git lfs track` 查看现有的文件追踪模式
6. 提交后执行`git lfs ls-files` 可以显示当前跟踪的文件列表
7. 其他的文件添加跟正常的git 工作流是保持一致.
8. 拉取代码直接就是 git pull , 注意不需要使用 git lfs pull了.