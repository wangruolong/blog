---
title: Git
author: 大大白
date: 2018-01-03 12:00:00
categories:
- Git
tags: 
- Git
---

Git在使用过程中的一些积累和沉淀。目前包含了Sourcetree的安装使用、Gerrit基本用法、Submodule的基本用法等。
<!-- more -->

# Sourcetree
安装和使用Sourcetree。
# Gerrit
Gerrit的基本用法。
# Submodule
## 什么是Submodule
有种情况我们经常会遇到：某个工作中的项目需要包含并使用另一个项目。 也许是第三方库，或者你独立开发的，用于多个父项目的库。 现在问题来了：你想要把它们当做两个独立的项目，同时又想在一个项目中使用另一个。
Git 通过子模块来解决这个问题。 子模块允许你将一个 Git 仓库作为另一个 Git 仓库的子目录。 它能让你将另一个仓库克隆到自己的项目中，同时还保持提交的独立。

## 如何使用submodule
### 在主模块加入子模块
先进入到你想要存放子模块的目录，然后把子模块的git仓库加进来。
```
$ cd kindergarten-gis-web\src\main\js\src\components
$ git submodule add git@git.sdp.nd:component-h5/preschool-gis.git
$ git submodule add git@git.sdp.nd:component-h5/preschool-gis.git
Cloning into 'D:/workspace/preschool/gerrit/kindergarten-gis-web/src/main/js/src/components/preschool-gis'...
remote: Counting objects: 449, done.
remote: Compressing objects: 100% (335/335), done.
remote: Total 449 (delta 81), reused 445 (delta 79)
Receiving objects: 100% (449/449), 3.41 MiB | 881.00 KiB/s, done.
Resolving deltas: 100% (81/81), done.
warning: LF will be replaced by CRLF in .gitmodules.
The file will have its original line endings in your working directory.
```
 现在`git status`会发现，多了一个`.gitmodules`文件，和一个新的文件夹`preschool-gis`
```
$ git status
On branch feature/submodule
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   .gitmodules
        new file:   src/main/js/src/components/preschool-gis
```
`.gitmodules`文件保存了项目 URL 与已经拉取的本地目录之间的映射，用`cat`命令可以查看到该文件的具体内容。如果有多个子模块，该文件中就会有多条记录。要重点注意的是，该文件也像`.gitignore`文件一样受到（通过）版本控制。它会和该项目的其他部分一同被拉取推送。这就是克隆该项目的人知道去哪获得子模块的原因。
```
$ cat .gitmodules
[submodule "src/main/js/src/components/preschool-gis"]
        path = src/main/js/src/components/preschool-gis
        url = git@git.sdp.nd:component-h5/preschool-gis.git
        branch = stable
```
`path = src/main/js/src/components/preschoolGis`，表示是submodule的目录，同时它被作为key来唯一标识，因为一个项目可能会存在多个submodule。
`url = git@git.sdp.nd:component-h5/preschool-gis.git`，是远程仓库的地址。
`branch = stable`，是跟踪的分支，可以设置跟踪分支也可以不设置跟踪分支。这里没有设置所以没有显示。
> 特别需要注意的是：默认情况下是没有跟踪分支的。它既不是跟踪master也不是跟踪任何分支，他是出于游离状态的HEAD。那么这种游离状态的分支他要怎么跟新代码呢？你只需要`git merge origin/master`把master的代码合并到自己的游离分支就好了。当前游离的commitId和master的最新的commitId一致，则表示拉取成功！

>虽然`preschool-gis`是工作目录中的一个子目录，但 Git 还是会将它视作一个子模块。当你不在那个目录中时，Git 并不会跟踪它的内容， 而是将它看作该仓库中的一个特殊提交。

> 现在我们已经让`kindergarten-gis-web`和`preschool-gis`关联在一起了。那么别的小伙伴怎么用呢？

### 基本的检出和拉取
- 检出。根据检出的时机不同分两种情况。
    * 第一种：在主项目未添加submodule的时候已经检出，这时候要获取submodule的代码需要如下操作。
        ```
        git submodule init
        git submodule update
        ```
    * 第二种：直接重新检出项目，可以用递归检出的方式，把子模块也一起检出。
        ```
        git clone --recursive git@git.sdp.nd:app-web/kindergarten-gis-web.git
        ```
- 拉取。在第三方拉取代码
    * 第一种：进入到submodule拉取。
        ```
        cd src/main/js/src/components/preschoolGis
        git fetch
        git merge origin/master
        ```
    * 第二种：直接在主项目的根目录执行。
        ```
        git submodule update --remote 
        ```
        + 进一步的，我们可以设置remote跟踪的分支
        ```
        git config -f .gitmodules submodule.src/main/js/src/components/preschoolGis.branch stable
        ```
        + 然后接下来执行的每一次update都是跟踪这个分支。没有-f只对自己有效，加了-f则对所有人都有效。
        ```
        git submodule update --remote
        ```

### 拉取和推送

#### 在主模块拉取
`git submodule update --remote`默认会更新master分支的代码，但是它本身又不处于master分支。它会更新master最新的代码。然后让自己处于游离状态。
```
$ git submodule update --remote
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 3 (delta 2), reused 0 (delta 0)
Unpacking objects: 100% (3/3), done.
From git.sdp.nd:component-h5/preschool-gis
   4741b80..ed40b3f  stable     -> origin/stable
Submodule path 'preschool-gis': checked out '60611eafaa81b43bb8b27decd9cb219cb1a2b19d'
```
如果我们希望它更新我们指定分支的代码可以这样操作。有两种方法可以修改在全局范围内的子项目跟踪的远程分支。
- 第一种：直接修改`.gitmodule`文件，在最后一行加入`branch = stable`。例如：
```
$ cat .gitmodules
[submodule "src/main/js/src/components/preschool-gis"]
        path = src/main/js/src/components/preschool-gis
        url = git@git.sdp.nd:component-h5/preschool-gis.git
        branch = stable
```
要特别注意换行和tab
- 第二种：本质上也是和第一种一样，只不过它是用命令行的方式实现的操作。
```
$ git config -f .gitmodules submodule.src/main/js/src/components/preschool-gis.branch stable
```
这里要特别注意`src/main/js/src/components/preschool-gis`这个我们在git add submodule的时候的别名，如果没有设置别名，git会自动把你的文件目录路径作为别名，也就是key保存下来，他在全局是唯一的。
```
$ cat .gitmodules
[submodule "src/main/js/src/components/preschool-gis"]
        path = src/main/js/src/components/preschool-gis
        url = git@git.sdp.nd:component-h5/preschool-gis.git
        branch = stable
```
在查看一下`.gitmodules`文件发现branch已经变了。
这时候再`git submodule update --remote`就会发现子模块的代码已经被关联到stable分支了，并且拉取了stable分支的最新代码。

#### 在子模块拉取
在项目中使用子模块的最简模型，就是只使用子项目并不时地获取更新，而并不在你的检出中进行任何更改。
如果想要在子模块中查看新工作，可以进入到目录中运行`git fetch`与`git merge`，合并上游分支来更新本地代码。
```
git fetch 
git merge origin/master 
```
或者也可以
```
git checkout stable
git pull --rebase
```
这个和我们正常使用git是一样的。

#### 在主模块推送
这个和平常使用的git一样。子模块如果有更改，主模块也要commit后push。

#### 在子模块推送
如果我们在主项目中提交并推送但并不推送子模块上的改动，其他尝试检出我们修改的人会遇到麻烦，因为他们无法得到依赖的子模块改动。 那些改动只存在于我们本地的拷贝中。

为了确保这不会发生，你可以让 Git 在推送到主项目前检查所有子模块是否已推送。 git push 命令接受可以设置为 “check” 或 “on-demand” 的 --recurse-submodules 参数。 如果任何提交的子模块改动没有推送那么 “check” 选项会直接使 push 操作失败。

首先，让我们进入子模块目录然后检出一个分支。
```
$ git checkout stable
Switched to branch 'stable'
```
然后在里面修改代码之后
```
$ git add . 
$ git commit -a
$ git push
$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 464 bytes | 464.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0)
To git.sdp.nd:component-h5/preschool-gis.git
   8b3aca3..5a81d0d  stable -> stable

```
然后查看一下子目录都是clean的
```
$ git status
On branch stable
Your branch is up to date with 'origin/stable'.

nothing to commit, working tree clean
```
这样我们就在子模块里面把代码提交上去了。
然后我们再回到主目录查看一下状态。
```
$ git status
On branch feature/submodule
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)

        modified:   preschool-gis (new commits, modified content)

no changes added to commit (use "git add" and/or "git commit -a")
```
会发现还有一个变动，它把子目录的任何变动都视为`preschool-gis`的变动，你只要把这个文件commit上去就可以了。
``
总结来说就是，需要在子目录push一次，然后再回到主目录push一次。
这样操作变得很繁琐，而且有时候在子目录提交了会忘记在主目录再提交一次，会造成没必要的误会。
我们可以设置push的`--recurse-submodules`参数，它有两个值，一个是check另一个是on-demand。check是在你push的时候检查子目录是不是还有没提交的，如果还有没提交的你要手动到子目录提交。而on-demand则是直接进入子目录自动帮你提交。让我们来试一下这两种操作。

当我们在子目录修改完后commit，然后主目录的`preschool-gis`会显示修改，我们这时候把主目录的修改也commit，那么就剩最后一步push了。这时候加上参数，他会提示我们：子目录的文件还没有push，你可以用on-demand这个参数，或者手动cd进去push之后再回来主目录push

`
$ git push --recurse-submodules=check
The following submodule paths contain changes that can
not be found on any remote:
  src/main/js/src/components/preschool-gis

Please try

        git push --recurse-submodules=on-demand

or cd to the path and use

        git push
 
to push them to a remote.
``

然后我们尝试用on-demand，但是居然报错了。
$ git push --recurse-submodules=on-demand
fatal: src refspec 'refs/heads/feature/submodule' must name a ref
这是因为我们用了gerrit。



## 问题
1. git submodule的时候报错
```
A git directory for 'src/main/js/src/components/preschool-gis' is found locally with remote(s):
  origin        git@git.sdp.nd:component-h5/preschool-gis.git
If you want to reuse this local git directory instead of cloning again from
  git@git.sdp.nd:component-h5/preschool-gis.git
use the '--force' option. If the local git directory is not the correct repo
or you are unsure what this means choose another name with the '--name' option.
```
意思是这个名字已经被占用需要换一个，或者直接删除.git/modules下面的所有文件。
2. 克隆含有子模块的项目
方法一：进入到子模块的目录后
```
git submodule init
git submodule update
```
方法二：直接重新检出项目
```
git clone --recursive git@git.sdp.nd:app-web/kindergarten-gis-web.git
```