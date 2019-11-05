## 基本用法

> 由于Git是管理修改而不是管理文件

- **工作区**：是电脑中能够看到的目录
- **版本库**（**Repository**）：指的是在工作区里面隐藏目录 `.git`，这个目录不属于 工作区，是 `Git` 的版本库。Git的版本库里存了很多东西，其中最重要的就是称为`stage`的暂存区，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。如图 

![](https://img-blog.csdn.net/20170808201722782?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGVpc2ly/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



- **暂存区**（**stage**)：数据暂时存放的区域，可在工作区和版本库之间数据的友好交流。

![](https://www.runoob.com/wp-content/uploads/2019/05/1557394755-3986-basic-usage.svg-.png)

上面的四条命令在`工作目录`、`暂存目录`和`仓库之间`复制文件。

- `git add files` 把当前文件放入暂存区域。
- `git commit` 给暂存区域**生成快照**并提交。
- `git reset -- files` 用来撤销最后一次`git add files`，你也可以用`git reset` 撤销所有暂存区域文件。**恢复快照**。
- `git checkout -- files` 把文件从**暂存区域复制到工作目录**，用来丢弃本地修改。

也可以跳过暂存区域直接从仓库取出文件或者直接提交代码。

![](https://www.runoob.com/wp-content/uploads/2019/05/1557394755-5406-basic-usage-2.svg-.png)

- `git commit -a `相当于运行 `git add` 把**所有当前目录下的文件加入暂存区域**再运行。`git commit`.
- `git commit files` 进行一次包含最后一次提交加上工作目录中文件快照的提交。并且文件被添加到暂存区域。
- `git checkout HEAD -- files` 回滚到复制最后一次提交。

![img](https://www.runoob.com/wp-content/uploads/2019/05/1557394755-9088-conventions.svg-.png)

绿色的 5位字符表示提交的ID，分别指向 父节点。分支用橘色显示，分别指向特点的提交。当前分支由附在其上的 `Head` 标识。这张图片里显示最后5次提交，`ed489`是最新提交。 *master*分支指向此次提交，另一个`maint`分支指向祖父提交节点。

## 命令详解

### Diff

有许多种方法查看两次提交之间的变动。下面是一些示例。

![](https://www.runoob.com/wp-content/uploads/2019/05/1557394756-8361-diff.svg-.png)

- **git diff** :  工作区与缓冲区的差异
- **git diff head** : 工作区与版本库的差异
- **git diff --cached** ：暂存区与版本库的差异

### Commit

提交时，git用**暂存区域**的文件创建一个新的提交，并把此时的节点设为**父节点**。然后把当前分支指向新的提交节点。下图中，当前分支是*master*。 在运行命令之前，*master*指向`ed489`，提交后，*master*指向新的节点`f0cec`并以`ed489`作为父节点。

![](https://www.runoob.com/wp-content/uploads/2019/05/1557394758-7356-commit-master.svg-.png)

即便**当前分支**是某次提交的祖父节点，git会同样操作。下图中，在*master*分支的祖父节点*maint*分支进行一次提交，生成了`1800b`。 这样，*maint*分支就不再是*master*分支的祖父节点(**分成两个链表**)。此时合并(或者衍合) 是必须的。

![](https://www.runoob.com/wp-content/uploads/2019/05/1557394758-4847-commit-maint.svg-.png)

如果想更改一次提交，使用 `git commit --amend`。git会使用与当前提交相同的父节点进行一次新提交，旧的提交会被取消。

![](https://www.runoob.com/wp-content/uploads/2019/05/1557394758-1429-commit-amend.svg-.png)

### checkout

`checkout`命令用于从历史提交（或者暂存区域）中拷贝文件到工作目录，也可用于**切换分支**。

当给定某个文件名（或者打开-p选项，或者文件名和-p选项同时打开）时，git会从指定的提交中拷贝文件到暂存区域和工作目录。比如，`git checkout HEAD~ foo.c`会将提交节点*HEAD~*(即当前提交节点的父节点)中的`foo.c`复制到工作目录并且加到暂存区域中。（**如果命令中没有指定提交节点，则会从暂存区域中拷贝内容**。）注意当前分支不会发生变化。

![](https://www.runoob.com/wp-content/uploads/2019/05/1557394762-8267-checkout-files.svg-.png)

当不指定文件名，而是给出一个（本地）分支时，那么*HEAD*标识会移动到那个分支（也就是说，我们"**切换"到那个分支了**），**然后暂存区域和工作目录中的内容会和*HEAD*对应的提交节点一致**。新提交节点（下图中的a47c3）中的所有文件都会被复制（到暂存区域和工作目录中）；只存在于老的提交节点（ed489）中的文件会被删除；不属于上述两者的文件会被忽略，不受影响。

![](https://www.runoob.com/wp-content/uploads/2019/05/1557394763-1721-checkout-branch.svg-.png)

如果**既没有指定文件名**，**也没有指定分支名**，而是一个标签、远程分支、SHA-1值或者是像*master~3*类似的东西，就得到一个匿名分支，称作*detached HEAD*（**被分离的*HEAD*标识**）。这样可以很方便地在历史版本之间互相切换。比如说你想要编译1.6.6.1版本的git，你可以运行`git checkout v1.6.6.1`（这是一个标签，而非分支名），编译，安装，然后切换回另一个分支，比如说`git checkout master`。然而，当提交操作涉及到"分离的HEAD"时，其行为会略有不同，详情见在下面。

![](https://www.runoob.com/wp-content/uploads/2019/05/1557394761-6610-checkout-detached.svg-.png)

### HEAD标识处于分离状态时的提交操作

当*HEAD*处于分离状态（不依附于任一分支）时，提交操作可以正常进行，但是不会更新任何已命名的分支。(你可以认为这是在更新一个匿名分支。)

![](http://marklodato.github.io/visual-git-guide/commit-detached.svg)

一旦此后你切换到别的分支，比如说*master*，那么这个提交节点（可能）再也不会被引用到，然后就会被丢弃掉了。注意这个命令之后就不会有东西引用*2eecb*。

![](http://marklodato.github.io/visual-git-guide/checkout-after-detached.svg)

但是，如果你想保存这个状态，可以用命令`git checkout -b name`来创建一个新的分支。

![](http://marklodato.github.io/visual-git-guide/checkout-b-detached.svg)

### Reset

> 版本回退

reset命令把**当前分支指向另一个位置**，并且有选择的**变动工作目录和索引**。也用来在从历史仓库中复制文件到索引，而不动工作目录。

如果不给选项，那么当前分支指向到那个提交。如果用`--hard`选项，**那么工作目录也更新**，如果用`--soft`选项，那么都不变。

![](http://marklodato.github.io/visual-git-guide/reset-commit.svg)

如果没有给出提交点的版本号，那么默认用*HEAD*。这样，分支指向不变，但是索引会回滚到最后一次提交，如果用`--hard`选项，工作目录也同样。

![](http://marklodato.github.io/visual-git-guide/reset.svg)

如果给了文件名(或者 `-p`选项), 那么工作效果和带文件名的[checkout](http://marklodato.github.io/visual-git-guide/index-zh-cn.html#checkout)差不多，除了索引被更新。

![](http://marklodato.github.io/visual-git-guide/reset-files.svg)

### Merge

`merge` 命令把不同分支合并起来。合并前，索引必须和当前提交相同。如果另一个分支是当前提交的祖父节点，那么合并命令将什么也不做。 **另一种情况是如果当前提交是另一个分支的祖父节点**，就导致*fast-forward*合并。指向只是简单的移动，并生成一个新的提交。

![](http://marklodato.github.io/visual-git-guide/merge-ff.svg)

否则就是一次真正的合并。默认把当前提交(`ed489` 如下所示)和另一个提交(`33104`)以及他们的共同祖父节点(`b325c`)进行一次[三方合并](http://en.wikipedia.org/wiki/Three-way_merge)。结果是**先保存当前目录和索引**，然后和父节点*33104*一起做一次新提交。

![](http://marklodato.github.io/visual-git-guide/merge.svg)

**三方合并**  用额外的承诺来将两个分支联系在一起。当你试图合并两个（**两个分支改变了同一个文件的同一部分**）出现。发生这种情况时，`Git` 将无法确定要使用哪个版本。

> `git merge` 命令用于将两个或两个以上的开发历史加入（合并）一起

**示例**

合并分支`fixes`和`enhancements`在当前分支的顶部，使它们合并：

```shell
git merge fixes enhancements
```



### git pull

`git pull ` 命令从另一个存储库或本地分支获取并整合`git pull`命令的作用是：取回远程主机某个分支的更新，再与本地的指定分支合并。 只是更新了 版本库 。工作目录没有变。

**使用语法** 

```shell
git pull [options] [<repository> [<refspec>…]]
```

**描述**

将远程存储库中的更改合并到当前分支中。在默认模式下，`git pull`是`git fetch`后跟`git merge FETCH_HEAD`的缩写。

**示例**

比如，要取回`origin`主机的`next`分支，与本地的`master`分支合并，需要写成下面这样 -

```shell
git pull origin next:master
```

如果远程分支（`next`)要与当前分支合并，则冒号后面的部分可以省略。

```shell
git pull origin next
```

在某些场合，`Git` 会自动在本地分支与远程分支之间，建立一种追踪关系（`tracking`)。

比如，在`git clone`的时候，所有本地分支默认与远程主机的同名分支，建立追踪关系，也就是说，本地的`master`分支自动”追踪”`origin/master`分支。

如果  `当前分支` 与 `远程分支`存在追踪关系，`git pull` 就可以省略远程分支名。

```shell
git push orign
```

上面命令表示，本地的当前分支自动与对应的 `origin` 主机`追踪分支`进行合并

如果当前分支只有一个追踪分支，连远程主机名都可以省略。

```shell
git pull
```

上面命令表示，当前分支自动与唯一一个追踪分支进行合并。



### git push

> 多人协同工作，可通过修改操作将代码文件最后一个确定版本提交，然后再推送变更。推送（push）操作将数据永久存储在 `git` 仓库。成功的推送操作后，其他开发人员可以看到新提交的变化。

```shell
git push origin master
```

将 本地 `commit` 过的 `push` 到远程服务器。 

### 创建分支

本地创建分支

`git checkout -b newbranch`

然后新增内容

`git add a.txt`

将分支推到远程，并在远程创建分支

`git push origin newbranch `

## 技术说明

文件内容并没有真正存储在索引(*.git/index*)或者提交对象中，而是以blob的形式分别存储在数据库中(*.git/objects*)，并用SHA-1值来校验。 索引文件用识别码列出相关的blob文件以及别的数据。对于提交来说，以树(*tree*)的形式存储，同样用对于的哈希值识别。树对应着工作目录中的文件夹，树中包含的 树或者blob对象对应着相应的子目录和文件。每次提交都存储下它的上一级树的识别码。

如果用detached HEAD提交，那么最后一次提交会被the reflog for HEAD引用。但是过一段时间就失效，最终被回收，与`git commit --amend`或者`git rebase`很像。