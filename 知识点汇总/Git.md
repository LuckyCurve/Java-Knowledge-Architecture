# Git push与pull的默认行为



> 摘自博客：http://blog.angular.in/git-pushmo-ren-fen-zhi/
>
> 感觉写的非常好，没忍住摘了下来，怕哪天网站塌了就没有这篇技术文章了

一直以来对`git push`与`git pull`命令的默认行为感觉混乱，今天抽空总结下。

## git push

通常对于一个本地的新建分支，例如`git checkout -b develop`, 在develop分支commit了代码之后，如果直接执行`git push`命令，develop分支将不会被push到远程仓库（但此时`git push`操作有可能会推送一些代码到远程仓库，这取决于我们本地git config配置中的`push.default`默认行为，下文将会逐一详解）。

因此我们至少需要显式指定将要推送的分支名，例如`git push origin develop`，才能将本地新分支推送到远程仓库。

当我们通过显式指定分支名进行初次push操作后，本地有了新的commit，此时执行`git push`命令会有什么效果呢？

如果你未曾改动过git config中的`push.default`属性，根据我们使用的git不同版本（Git 2.0之前或之后），`git push`通常会有两种截然不同的行为:

1. develop分支中本地新增的commit被push到远程仓库
2. push失败，并收到git如下的警告

```
fatal: The current branch new has no upstream branch.  
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin develop
```

为什么git版本不同会有两种不同的push行为？

因为在[git的全局配置中，有一个push.default](http://git-scm.com/docs/git-config)属性，其决定了`git push`操作的默认行为。在Git 2.0之前，这个属性的默认被设为'matching'，2.0之后则被更改为了'simple'。

我们可以通过`git version`确定当前的git版本（如果小于2.0，更新是个更好的选择），通过`git config --global push.default 'option'`改变push.default的默认行为（或者也可直接编辑~/.gitconfig文件）。

**push.default** 有以下几个可选值：
**nothing, current, upstream, simple, matching**

其用途分别为：

- **nothing** - push操作无效，除非显式指定远程分支，例如`git push origin develop`（我觉得。。。可以给那些不愿学git的同事配上此项）。
- **current** - push当前分支到远程同名分支，如果远程同名分支不存在则自动创建同名分支。
- **upstream** - push当前分支到它的upstream分支上（这一项其实用于经常从本地分支push/pull到同一远程仓库的情景，这种模式叫做central workflow）。
- **simple** - simple和upstream是相似的，只有一点不同，simple必须保证本地分支和它的远程 upstream分支同名，否则会拒绝push操作。
- **matching** - push所有本地和远程两端都存在的同名分支。

因此如果我们使用了git2.0之前的版本，push.default = matching，git push后则会推送当前分支代码到远程分支，而2.0之后，push.default = simple，如果没有指定当前分支的upstream分支，就会收到上文的fatal提示。

## upstream & downstream

说到这里，需要解释一下[git中的upstream到底是什么](http://stackoverflow.com/questions/2739376/definition-of-downstream-and-upstream)：

> git中存在upstream和downstream，简言之，当我们把仓库A中某分支x的代码push到仓库B分支y，此时仓库B的这个分支y就叫做A中x分支的upstream，而x则被称作y的downstream，这是一个相对关系，每一个本地分支都相对地可以有一个远程的upstream分支（注意这个upstream分支可以不同名，但通常我们都会使用同名分支作为upstream）。

初次提交本地分支，例如`git push origin develop`操作，并不会定义当前本地分支的upstream分支，我们可以通过`git push --set-upstream origin develop`，关联本地develop分支的upstream分支，另一个更为简洁的方式是初次push时，加入-u参数，例如`git push -u origin develop`，这个操作在push的同时会指定当前分支的upstream。

注意push.default = current可以在远程同名分支不存在的情况下自动创建同名分支，有些时候这也是个极其方便的模式，比如初次push你可以直接输入 git push 而不必显示指定远程分支。

## git pull

弄清楚`git push`的默认行为后，再来看看`git pull`。

当我们未指定当前分支的upstream时，通常`git pull`操作会得到如下的提示：

```
There is no tracking information for the current branch.  
Please specify which branch you want to merge with.  
See git-pull(1) for details

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

    git branch --set-upstream-to=origin/<branch> new1
```

`git pull`的默认行为和`git push`完全不同。当我们执行`git pull`的时候，实际上是做了`git fetch + git merge`操作，fetch操作将会更新本地仓库的remote tracking，也就是refs/remotes中的代码，并不会对refs/heads中本地当前的代码造成影响。

当我们进行pull的第二个行为merge时，对git来说，如果我们没有设定当前分支的upstream，它并不知道我们要合并哪个分支到当前分支，所以我们需要通过下面的代码指定当前分支的upstream：

```
git branch --set-upstream-to=origin/<branch> develop  
// 或者git push --set-upstream origin develop 
```

实际上，如果我们没有指定upstream，git在merge时会访问git config中当前分支(develop)merge的默认配置，我们可以通过配置下面的内容指定某个分支的默认merge操作

```
[branch "develop"]
    remote = origin
    merge = refs/heads/develop // [1]为什么不是refs/remotes/develop?
```

或者通过command-line直接设置：

```
git config branch.develop.merge refs/heads/develop  
```

这样当我们在develop分支git pull时，如果没有指定upstream分支，git将根据我们的config文件去`merge origin/develop`；如果指定了upstream分支，则会忽略config中的merge默认配置。

以上就是git push和git pull操作的全部默认行为，如有错误，欢迎斧正🙈

------

[1] 为什么merge = refs/heads/develop 而不是refs/remotes/develop?
因为这里merge指代的是我们想要merge的远程分支，是remote上的refs/heads/develop，文中即是origin上的refs/heads/develop，这和我们在本地直接执行`git merge`是不同的(本地执行`git merge origin/develop`则是直接merge refs/remotes/develop)。





# Git配置



三种变量配置存储行为：

- 系统配置配置：`/etc/gitconfig`，查看参数`--system`
- 当前用户配置：`~/.gitconfig`，查看参数`--global`
- 当前仓库配置：`.git/config`，查看参数`--local`

默认下一级的配置变量会覆盖上一级的，可以使用`git config`+对应存储行为参数+`-l`来查看，如果没有指定存储行为那么直接获取当前能获取到的变量信息，可以加上`--show-origin`参数查看当前变量配置来源文件



就行存储行为变量配置时候也非常简单，带上`--global`参数之后，即可直接将变量信息写入到对应的配置文件当中去，如果没有指定只会写入当前临时变量区，因此以后就不用再去添加了 ，变量设置格式为：

```
git config --global user.name "LuckyCurve"
git config --global user.email "luckycurvec@gmail.com"
```

> 查找对应的参考手册很容易：`git <command> --help`，本地HTML文件，是英文的并且比较全，如果只是想获取使用参数：`git <command> -h`





# Git基础



我们使用`git clone`命令克隆仓库的时候，执行逻辑为：从远程仓库拉取`.git`文件夹，然后从中读取最新版本的文件的拷贝，拷贝到当前工作区当中来

如果需要单独指定文件名字：`git clone <url> <dir-name>`



文件的两种状态：

- 已追踪：在上一次的快照当中有它们的记录，可细分为
  - 未修改
  - 已修改
  - 已放入暂存区
- 未追踪：在上一次的快照当中没有它们的记录，往往是新创建的



直接使用`git status`来查看文件状态，会输出的比较繁琐，可以使用`git status -s`命令来简短输出内容，输出的几种格式：

- `M`：修改过的文件，绿色的：已经加入缓冲区，红色的：未加入缓冲区
- `??`：未追踪的文件
- `A`：新添加到的暂存区的文件，一般是开始追踪的文件



`.gitignore`忽略文件，使用标准的glob模式匹配（Shell使用的，简化了的正则表达式）

很好的一个Demo：

```
# 忽略所有的 .a 文件
*.a
# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a
# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO
# 忽略任何目录下名为 build 的文件夹
build/
# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt
# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```

> 文件 .gitignore 的格式规范如下：
> • 所有空行或者以 # 开头的行都会被 Git 忽略。
> • 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
> • 匹配模式可以以（/）开头防止递归。
> • 匹配模式可以以（/）结尾指定目录。
> • 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（!）取反。
> 所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。 星号（*）匹配零个或多个任意字符；[abc] 匹配
> 任何一个列在方括号中的字符 （这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）； 问号（?）只
> 匹配一个任意字符；如果在方括号中使用短划线分隔两个字符， 表示所有在这两个字符范围内的都可以匹配
> （比如 [0-9] 表示匹配所有 0 到 9 的数字）。 使用两个星号（**）表示匹配任意中间目录，比如 a/**/z 可以
> 匹配 a/z 、 a/b/z 或 a/b/c/z 等。



通常使用`git diff`命令查看两点：

1、哪些更新尚未暂存——`git diff`

2、哪些更新已经暂存并等待下次提交——`git diff --staged`或者`git diff -cached`



可以直接使用git commit命令，命令输出过程当中会调用git status帮助你查看哪些文件进行了修改

git提供了`git commit -a`的方式跳过暂存区，直接将所有以追踪的文件进行提交，相当于帮你自动执行add已经track的文件了



删除文件：

- git rm：从工作区和暂存区当中删除，如果暂存区当中存在修改未提交，此时需要-f参数才能rm成功
- rm：操作系统指令，删除工作区内容，进行add后将删除提交到缓冲区就等价于git rm了
- git rm --cached：仅在暂存区当中添加文件删除信息，磁盘上仍保留

> 如果将删除信息写入到暂存区了并且提交了（上述三条指令都有这个作用），那么Git仓库的最新版本当中文件消失



git历史

git log查看，可以使用`git log -p`查看每次提交的修改，以及使用`git log -p -2`查看最近两次提交的修改，或者使用`git log --stat`列出审计信息，如增加修改文件数量等等。

![image-20211228172348228](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211228172348228.png)





一个非常有用的命令：`git commit --amend` amend：修正

该命令会将当前暂存区内容直接合并到Git仓库的最新版本当中，并且可以重新提交commit信息，因此如果commit info写错了可以使用该命令来完成，版本的SHA-1的值也会变

实际操作就是用一个新的提交覆盖了旧的提交，因为上面的SHA-1的值完全改变了



取消暂存的文件，在原来的git版本当中我们使用`git status`命令的时候，git建议我们使用`git reset HEAD <file>`去将文件从暂存区当中删除，在新版本的git当中建议使用`git restore --staged <file>`，可能因为git reset是一个非常危险的命令，可能更改工作区当中的内容



想要撤销对工作区当中文件的修改，命令行当中也提示了：`git restore <file>`，会回到最近的一个版本当中去



在git当中任何已经提交的东西几乎总是可以恢复的，包括被删除的分支中的提交和使用amend覆盖的提交，如果没有提交很有可能找不回来了



`git fetch <remote> [<branch>]`命令相当于拉取远程仓库到本地仓库，但他不会合并或者修改你的工作区，因此`git clone <remote> [<dir>]`相当于是先fetch下来到本地然后再合并到当前工作区的组合命令了，`git pull <remote>`命令也是使用fetch远程仓库然后尝试合并到当前分支，如果有冲突就得我们手动去fetch然后处理了。



可以使用`git remote show <origin>`查看远程分支状态

```bash
$ git remote show github
* remote github
  Fetch URL: git@github.com:LuckyCurve/Java-Knowledge-Architecture.git
  Push  URL: git@github.com:LuckyCurve/Java-Knowledge-Architecture.git
  HEAD branch: master
  Remote branch:
    master tracked
  Local ref configured for 'git push':
    master pushes to master (up to date)
```



git推荐给一些重要的版本打上tag，可以使用`git tag`查看打上的所有标签

> 默认就是查看所有的，如果使用参数`git tag -l "user*"`这时候-l就不是list，而是强调使用后面的



标签：轻量标签和附注标签

轻量标签：某个特定提交的引用

附注标签：存储在Git仓库当中的一个完整对象，有自己的SHA-1，可以被校验的

通常建议创建附注标签：`git tag -a <tagname>`，通常是v1.2的形式，然后输入附加信息

使用`git show <tagname>`查看具体信息

轻量标签：不带参数a即可，没有附加信息

附注标签较之于轻量标签会多这个信息：

![image-20211229195744406](https://gitee.com/LuckyCurve/img/raw/master//img/image-20211229195744406.png)



如果想给历史版本打标签也非常简单：`git tag -a <tagname> SHA-1`

SHA-1的值建议使用`git log --pretty=oneline`来查看



默认操作下标签是不共享的，如果想要将本地标签push到服务器上，得显式的加上：

- `git push <branch> <tagname>`

如果需要上传所有标签：

- `git push <branch> --tags`

这样可以把标签上传到服务器，别人从服务器上拉取的时候也可以看到标签信息



如果删除本地标签，-d参数就好了，但是远程标签不会删除，即使我们也按照上面的push标签信息过去了也不会，要使用：`git push origin --delete <tagname>`来完成


git别名，可以使用一个别名代替组合命令：

```
git config --global alias.last 'log -1'
```

因此`git last`等价于`git log -1`





# Git分支



使用分支最主要的一点就是把工作从主线当中分离开来，以免影响主线的开发



Git在add到暂存区的时候就会计算文件的校验和信息，在commit的时候会产生一个完整的提交对象，提交对象可能有一个或者多个父对象指向上一次提交。

假设我们提交了三个文件到仓库，这次提交会产生五个对象，三个blob文件（文件快照），一个树信息（记录文件快照的层次结构，通过BLOB的SHA-1的值来构造的）和一个提交对象

提交对象包含上述的所有信息，此外还包括作者信息以及提交信息等等

![image-20220101131251901](https://gitee.com/LuckyCurve/img/raw/master//img/image-20220101131251901.png)

**Git的分支本质上仅仅是指向提交对象的可变指针**，只是在每次提交操作的时候会向后移动罢了



最直观的感受：创建一个testing分支：`git branch testing`

![image-20220101131506033](https://gitee.com/LuckyCurve/img/raw/master//img/image-20220101131506033.png)

Git区分当前是在哪一分支的，也非常简单：

![image-20220101131815805](https://gitee.com/LuckyCurve/img/raw/master//img/image-20220101131815805.png)

HEAD指向当前分支即可，相当于是当前分支的别名，因此在我们commit的时候，自然而然会让我们当前分支向后移动了

```
$ git log --oneline
db1066b (HEAD -> master, tag: v1.2, testing) complete Java v2
b606a4e (tag: v1.1) add Java.txt
401965b first commit
1870402 init version
```

切换分支也非常简单：`git checkout <branch>`，然后修改一下文件并进行提交，此时Git仓库状态为：

![image-20220101132244020](https://gitee.com/LuckyCurve/img/raw/master//img/image-20220101132244020.png)



如果我们此时再切换回master分支，**做了两件事儿**：

- 使HEAD重新指向master
- 将工作目录恢复成master所指向的快照内容

> 有一个非常奇怪的事儿，如果我们修改了内容，没有提交到git仓库，就checkout分支了，无论当前这个文件在工作目录还是在暂存区，都会直接带到切换后的工作目录和暂存区当中去
>
> :warning:：**不用太过在意修改内容**，如果修改了一个文件，在别的分支是没有的，是不允许切换过去的
>
> 
>
> ```
> error: Your local changes to the following files would be overwritten by checkout:
>      master.txt
> Please commit your changes or stash them before you switch branches.
> Aborting
> ```
>
> **官方给出的解答**：在你这么做之前，要留意你的工作目录和暂存区里那些还没有被提交的修改， 它可能会和你即将检出的分支产生冲突从而阻止 Git 切换到该分支。 最好的方法是，在你切换分支之前，保持好一个干净的状态。 

可以使用如下命令查看多分支：

```
$ git log --oneline --all[查看所有分支] --graph[以图形化的方式]
* 45ccf01 (HEAD -> master) master change
| * 64bf5f0 (testing) make a change in branch testing
|/
* db1066b (tag: v1.2) complete Java v2
* b606a4e (tag: v1.1) add Java.txt
* 401965b first commit
* 1870402 init version

```

![image-20220101133154429](https://gitee.com/LuckyCurve/img/raw/master//img/image-20220101133154429.png)



切换到原分支之后Git会重置工作区内容，让工作区内容和最后一次提交时候一模一样



分支合并：当前分支`git merge <branch>`合并其他分支

![image-20220101231939690](https://gitee.com/LuckyCurve/img/raw/master//img/image-20220101231939690.png)

一般切Master然后merge其他，简单的三方合并：C2，C4，C5

大部分情况不会出现冲突，出现冲突的情况：对同一文件的同一部分进行了不同的修改

文件会被标注成unmerged状态，并且git会手动修改文件内容成：

```
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```

`=======`用于文件内容分割，上半部分为HEAD的，下半部分为iss53的

修改完成之后add然后commit即可



git branch参数：

--merged 与 --no-merged 这两个有用的选项可以过滤这个列表中已经合并或尚未合并到当前分支的分支