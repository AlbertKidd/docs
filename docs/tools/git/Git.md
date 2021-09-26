
## 简介

### Git是啥

Git是一款免费并且开源的**分布式**版本控制系统，旨在快速高效地处理从小到大所有项目的版本管理。

### 为什么要用 Git

* 最流行，最大的开源社区Github也是基于Git
* 更加方便、快捷地协同工作

## Git 安装与配置

前往Git官方地址 https://git-scm.com/downloads 选择相应的平台进行下载并安装。

安装完成后的首次使用，需使用`git config`命令进行一些配置。

git的配置一般会存在3个位置：

* `<Git安装目录>/etc/gitconfig` 全局配置，对所有用户生效，命令参数：`--system`
* `<用户目录>/.gitconfig` 用户配置，只对当前用户生效，命令参数：`--global`
* `<项目目录>/.git/config` 项目配置，只对当前项目生效，命令参数：`--local`（默认）

同样的配置，最低层级的配置将会生效，即 项目配置 > 用户配置 > 全局配置。

常用的配置如下：

1. 配置用户标识（必需配置），所有Git操作均将使用此标识

```shell
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

2. 配置默认分支名称。Git默认的分支名为`master`，而GitHub以`master`可能使人联想奴隶制为由，将默认分支改为了`main`。Git支持更改默认分支名：

```shell
git config --global init.defaultBranch main
```

3. 配置proxy。由于某种原因，某些情况下使用Git与GitHub远程仓库无法进行通畅的交互时，Git支持配置proxy连接远程仓库：

```shell
# 配置HTTP代理
git config --global http.proxy http://127.0.0.1:1080

# 取消HTTP代理
git config --global --unset http.proxy

# 配置HTTPS代理
git config --global https.proxy http://127.0.0.1:1080

# 取消HTTPS代理
git config --global --unset https.proxy
```

配置完成后，可通过`git config -l`查看所有配置。

## 获取帮助

Git在命令行中内置了使用说明，有两种方式：

* `git <命令> -h` 在命令行中展示指定Git命令的简略使用说明
* `git help <命令>` 在浏览器中展示Git命令的详细使用说明

## Git 基础

[git常用命令](https://ndpsoftware.com/git-cheatsheet.html)

### 创建本地仓库

对于新项目需要使用Git进行版本控制时，可直接在项目目录下运行`git init`创建本地仓库。

对于已有Git仓库，需要在本地进行开发时，可使用`git clone <远程仓库地址>'克隆已有仓库。

### 修改与提交

在使用Git前，我们需要了解Git是如何管理文件的。
Git将文件分为了三个区：

![](img/areas.png)

* `working directory` 工作区，与操作系统的文件是一致的。
* `staging area` 暂存区，储存已add但还没有commit的修改。
* `repository` 仓库区，储存已commit的修改。

现在我们来看Git修改与提交流程

1. 修改或新增文件
2. 命令行运行`git add <文件名>`将修改/新增的文件添加到暂存区
3. 命令行运行`git commit`将暂存区的改动提交到本地仓库

此外，Git还提供了rename与remove操作

```shell
# 将指定文件从工作区与暂存区删除
git remove <文件>

# 将指定文件从暂存区删除
git remove --cache <文件>

# 将指定文件改名
git mv <文件> <新文件>

```

### 查看文件修改

在使用的过程中，可以随时通过`git status`查看当前分支的所有文件状态。

Git项目中文件状态的生命周期主要为以下4种：

![](img/lifecycle.png)

* `untracked` 未跟踪状态。从未添加至暂存区的文件，或直接删除暂存区的文件，会变为此状态
* `unmodified` 未修改状态。commit后的文件未修改时均为此状态。
* `modified` 已修改状态。commit/add后的文件修改后的状态。
* `staged` 暂存状态。add后的文件均为此状态。


Git中还可以通过`git diff`命令来对比文件差异：
```shell

# 查看工作区与暂存区文件的区别
git diff <文件>

# 查看暂存区与仓库区文件的区别
git diff --cached <文件>
```

### 撤销修改

`git reset`命令可进行撤销修改操作

```shell

# 将文件从暂存区移除
git reset HEAD <文件>
git restore --stage <文件>

# 将工作区的文件撤销修改
git restore <文件>
```

### gitigore

使用git在做相关操作时，若项目目录下存在`.gitignore`文件，则git命令会自动排除匹配`.gitignore`中定义的文件。`.gitignore`的规则类似antPath，实例如下：

```gitignore
# 排除所有.a文件
*.a

# 不排除lib.a, 即使上面已经定义排除所有的.a文件
!lib.a

# 只排除当前目录的TODO文件， 并不排除子目录中的TODO文件
/TODO

# 排除所有build目录
build/

# 排除doc/notes.txt
doc/*.txt

# 排除所有在doc目录中的.pdf文件，不管中间有多少层子目录
doc/**/*.pdf
```

## Git 分支

### 分支与合并

一般接到一个需求后，我们会新创建一个针对这个需求的分支进行开发。

分支常用命令：

```shell

# 创建新分支
git branch <分支名>

# 查看所有分支名
git branch -l 

# 切换分支
git checkout <分支名>

# 删除分支
git branch -d <分支名>
```

需求分支开发测试完成后，即可将此分支合并至主分支中，然后删除此分支。

合并分支常用命令：

```shell

# 将指定分支合并至当前分支
git merge <分支名>

```

若当前分支与合并分支有冲突，则需要解决冲突后方能提交。

### 远程分支
当项目存在远程仓库时，可添加远程分支进行拉取与推送。

```shell

# 添加远程仓库
git remote <远程仓库名> <远程仓库URL>

# 获取远程仓库数据，注意：此操作并不会与本地代码产生影响
git fetch <远程仓库名>

# 获取远程仓库数据并与当前分支合并
git pull <远程仓库名> <远程分支名>

# 向远程分支推送数据
git push <远程仓库名> <远程分支名>

```

## 使用 Github

Github是目前全球最大、使用人数最多的开源社区，顾名思义，其版本控制也是基于Git。Github目前提供了免费的公开/私有仓库以供开发者使用，我们也可以创建自己的Github仓库学习使用git。

