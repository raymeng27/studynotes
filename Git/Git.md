# Git

# 一、基础

> 版本控制是一种记录一个或如干个文件内容变化，以便将来查阅特定版本修订情况的系统。采用版本控制（CVS）可以将选定的文件回溯到之前的状态，甚至将整个项目都回退到过去某个时间点的状态。

## 1.Git简史

同生活中的许多伟大事物一样，Git 诞生于一个极富纷争大举创新的年代。

Linux 内核开源项目有着为数众多的参与者。 绝大多数的 Linux 内核维护工作都花在了提交补丁和保存归档的繁琐事务上（1991－2002年间）。 到 2002 年，整个项目组开始启用一个专有的分布式版本控制系统 BitKeeper 来管理和维护代码。

到了 2005 年，开发 BitKeeper 的商业公司同 Linux 内核开源社区的合作关系结束，他们收回了 Linux 内核社区免费使用 BitKeeper 的权力。 这就迫使 Linux 开源社区（特别是 Linux 的缔造者 Linus Torvalds）基于使用 BitKeeper 时的经验教训，开发出自己的版本系统。 他们对新的系统制订了若干目标。

- 速度
- 简单的设计
- 对非线性开发模式的强力支持（允许成千上万个并行开发的分支）
- 完全分布式
- 有能力高效管理类似 Linux 内核一样的超大规模项目（速度和数据量）

自诞生于 2005 年以来，Git 日臻成熟完善，在高度易用的同时，仍然保留着初期设定的目标。 它的速度飞快，极其适合管理大项目，有着令人难以置信的非线性分支管理系统。

## 2.Git详细内容

### 1.Git和其它文件对比

从概念上讲，其它大部分系统以文件变更列表的方式存储信息，这类系统将它们存储的信息看作是一组基本文件和每个文件随时间逐步积累的差异：

[![Rz03y4.png](https://z3.ax1x.com/2021/07/10/Rz03y4.png)](https://imgtu.com/i/Rz03y4)

Git不按照以上方式对待和保存数据。Git更像是把数据看作是对小型文件系统的一系列快照。在 Git 中，每当你提交更新或保存项目状态时，它基本上就会对当时的全部文件创建一个快照并保存这个快照的索引。 为了效率，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。

[![RzBKHA.png](https://z3.ax1x.com/2021/07/10/RzBKHA.png)](https://imgtu.com/i/RzBKHA)

在 Git 中的绝大多数操作都只需要访问本地文件和资源。除非你想进行远程仓库的推送。

### 2.Git保证完整性

Git 中所有的数据在存储前都计算校验和，然后以校验和来引用。 这意味着不可能在 Git 不知情时更改任何文件内容或目录内容。 这个功能建构在 Git 底层，是构成 Git 哲学不可或缺的部分。 若你在传送过程中丢失信息或损坏文件，Git 就能发现。

Git 用以计算校验和的机制叫做 SHA-1 散列（hash，哈希）。 这是一个由 40 个十六进制字符（0-9 和 a-f）组成的字符串，基于 Git 中文件的内容或目录结构计算出来。 SHA-1 哈希看起来是这样：

```
24b9da6552252987aa493b52f8696cd6d3b00373
```

Git 中使用这种哈希值的情况很多，你将经常看到这种哈希值。 实际上，Git 数据库中保存的信息都是以文件内容的哈希值来索引，而不是文件名。

你执行的 Git 操作，几乎只往 Git 数据库中 **添加** 数据。 你很难使用 Git 从数据库中删除数据，也就是说 Git 几乎不会执行任何可能导致文件不可恢复的操作。

### 3.三种状态

Git 有三种状态，你的文件可能处于其中之一： **已提交（committed）**、**已修改（modified）** 和 **已暂存（staged）**。这会让我们的 Git 项目拥有三个阶段：工作区、暂存区以及 Git 目录。

工作区是对项目的某个版本独立提取出来的内容。 这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。

暂存区是一个文件，保存了下次将要提交的文件列表信息，一般在 Git 仓库目录中。 按照 Git 的术语叫做“索引”，不过一般说法还是叫“暂存区”。

Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。 这是 Git 中最重要的部分，从其它计算机克隆仓库时，复制的就是这里的数据。

基本的 Git 工作流程如下：

1. 在工作区中修改文件。
2. 将你想要下次提交的更改选择性地暂存，这样只会将更改的部分添加到暂存区。
3. 提交更新，找到暂存区的文件，将快照永久性存储到 Git 目录。

如果 Git 目录中保存着特定版本的文件，就属于 **已提交** 状态。 如果文件已修改并放入暂存区，就属于 **已暂存** 状态。 如果自上次检出后，作了修改但还没有放到暂存区域，就是 **已修改** 状态。

### 4.Git安装

* 在Linux上安装

  基于RPM的发行版（如CentOS）：

  \# dnf install git-all

  基于Debian发行版（如Ubuntu）：

  \# apt install git -all

* 在macOS上安装

  在 Mac 上安装 Git 有多种方式。 最简单的方法是安装 Xcode Command Line Tools。

  如果你想安装更新的版本，可以使用二进制安装程序。 官方维护的 macOS Git 安装程序可以在 Git 官方网站下载，网址为 https://git-scm.com/download/mac。

  你也可以将它作为 GitHub for macOS 的一部分来安装。 它们的图形化 Git 工具有一个安装命令行工具的选项。 你可以从 GitHub for macOS 网站下载该工具，网址为 [https://mac.github.com](https://mac.github.com/)。

* 在Windows上安装

  官方版本可以在 Git 官方网站下载。 打开 https://git-scm.com/download/win，下载会自动开始。

### 5.Git初始配置

每台计算机上只需要配置一次，程序升级时会保留配置信息。 你可以在任何时候再次通过运行命令来修改它们。

Git自带一个git config的工具来帮助设置控制Git外观和行为的配置变量。这些变量存储在三个不同的位置：

* /ect/config文件：包含系统上每一个用户及他们仓库的通用配置。如果执行git config时带上--system选项，那么它就会读写该文件中的配置变量。
* ~/.gitconfig或~/.confif/git/config文件：只针对当前用户。可以传递--global选项让Git读写此文件，会对系统上的所有仓库生效。
* 当前仓库的Git目录中的config文件（即.git/config）：针对该仓库。可以传递--local选项让Git强制读写此文件，默认情况下用的就是它。

你可以通过以下命令查看所有的配置以及它们所在的文件：

```shell
# git config --list --show-origin
```

配置用户信息：

安装完 Git 之后，要做的第一件事就是设置你的用户名和邮件地址。 这一点很重要，因为每一个 Git 提交都会使用这些信息，它们会写入到你的每一次提交中，不可更改：

```shell
# git config --global user.name "Ray"
# git config --global user.email "raymeng27@163.com"
```

检查配置：

```shell
# git config --list
```

你可能会看到重复的变量名，因为 Git 会从不同的文件中读取同一个配置。可以通过git config <key>来检查git的某一项配置：

```shell
# git config user.name
```

获取帮助：git help <verb>，例如

```shell
# git help config
```

# 二、Git基础操作

## 1.获取仓库

通常有两种获取Git项目仓库的方式：

1. 将尚未进行版本控制的本地目录转换为Git仓库。
2. 从其它服务器克隆一个已存在的Git仓库。

### 1.在已存在目录中初始化仓库

如果你有一个尚未进行版本控制的目录，想要用Git来控制它，那么首先需要进入该项目目录中。然后执行：

```shell
$ git init
```

该命令将创建一个名为.git的子目录，这个子目录含有初始化的git仓库中所有的必须文件。这些文件是仓库的骨干。但是，在这个时候，仅仅是做了一个初始化的操作，项目里的文件还没有被追踪。

如果在一个已存在文件的文件夹中进行版本控制，你应该开始追踪这些文件并进行初始提交。可以通过`git add`命令来指定所需文件来进行追踪，然后执行`git commit`。

```shell
$ git add Test.java
$ git commit -m "add Test.java file"
```

### 2.克隆现有的仓库

如果想获得一份已经存在了的Git仓库的拷贝，可以使用`git clone`命令。Git克隆的是该仓库服务上的几乎所有数据，而不是仅仅复制完成工作所需的文件。当执行`git clone`命令的时候，默认配置下远程Git仓库中的每一个文件的每一个版本都将被拉取下来。事实上，如果服务器的磁盘坏掉了，通常可以使用任何一个克隆下来的用户端来重现服务器上的仓库。

克隆仓库的命令是`git clone <url>`。比如了龙Git用户raymeng27的goga仓库，可以使用命令：

```shell
$ git clone https://github.com/raymeng27/goga.git
```

Git支持多种数据传输协议。上面的例子是使用的https://协议，也可以使用git://协议或者SSH传输协议。

## 2.记录每次更新到仓库

### 1.操作流程

工作目录下的每一个文件都不外乎这两种状态：**已跟踪** 或 **未跟踪**。 已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后， 它们的状态可能是未修改，已修改或已放入暂存区。简而言之，已跟踪的文件就是 Git 已经知道的文件。

工作目录中除已跟踪文件外的其它所有文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有被放入暂存区。 初次克隆某个仓库的时候，工作目录中的所有文件都属于已跟踪文件，并处于未修改状态，因为 Git 刚刚检出了它们， 而你尚未编辑过它们。

编辑过某些文件之后，由于自上次提交后你对它们做了修改，Git 将它们标记为已修改文件。 在工作时，你可以选择性地将这些修改过的文件放入暂存区，然后提交所有已暂存的修改，如此反复。

[![WSVZRO.png](https://z3.ax1x.com/2021/07/10/WSVZRO.png)](https://imgtu.com/i/WSVZRO)

* 检查当前文件状态

  ```shell
  $ git status
  ```

* 追踪新文件

  使用命令`git add`开始跟踪一个文件。所以，要跟踪README.md文件，运行：

  ```shell
  $ git add README.md
  ```

### 2.状态简览

`git status`命令的输出十分详细，但其用于有些繁琐。Git有一个选项可以缩短状态命令的输出，这样可以以简洁的方式查看更改。可以使用 `git status -s`或者 `git status --short` 例如：

``` shell
$ git status -s
?? Git/
```

新添加的未跟踪文件前面有??标记，新添加到暂存区中的文件前面有A标记，修改过的文件前面有M标记。输出中有两栏，左栏指明了暂存区的状态，右栏指明了工作区的状态。

### 3.忽略文件

一般我们总会有些问价无需纳入Git的管理，也不希望他们总出现在未跟踪文件列表。通常首饰写自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。在这种情况下，可以创建名为.gitignore的文件，列出要忽略的文件的模式。如：

```shell
$ cat .gitignore
*.[oa]
*~
```

文件.gitignore的个数规范如下：

* 所有空行或者以#开头的行都会被Git忽略
* 可以使用标准的glob模式匹配，它会递归地应用在整个工作区中。
* 匹配模式可以以（/）开头防止递归。
* 匹配模式可以以（/）结尾指定目录。
* 要忽略指定模式以外的文件或目录，可以在模式钱加上叹号（!）取反。

如下例子：

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

### 4.查看以暂存和未暂存的修改

如果 `git status` 命令的输出对于你来说过于简略，而你想知道具体修改了什么地方，可以用 `git diff` 命令。

若要查看已暂存的将要添加到下次提交里的内容，可以用 `git diff --staged` 命令。 这条命令将比对已暂存文件与最后一次提交的文件差异：

然后用 `git diff --cached` 查看已经暂存起来的变化（ `--staged` 和 `--cached` 是同义词）

### 5.提交更新

现在的暂存区已经准备就绪，可以提交。务必确认还有什么已修改或新建的文件还没有 `git add` 过， 否则提交的时候不会记录这些尚未暂存的变化。 这些已修改但未暂存的文件只会保留在本地磁盘。 所以，每次准备提交前，先用 `git status` 看下，你所需要的文件是不是都已暂存起来了， 然后再运行提交命令 `git commit`：

```shell
$ git commit
```

这样会启动你选择的文本编辑器来输入提交说明。

另外，你也可以在 `commit` 命令后添加 `-m` 选项，将提交信息与命令放在同一行，如下所示：

```shell
$ git commit -m "add README.md"
```

尽管使用暂存区域的方式可以精心准备要提交的细节，但有时候这么做略显繁琐。 Git 提供了一个跳过使用暂存区域的方式， 只要在提交的时候，给 `git commit` 加上 `-a` 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤。

### 6.移除文件

要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交。 可以用 `git rm` 命令完成此项工作，并连带从工作目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。

如果只是简单地从工作目录中手工删除文件，运行 `git status` 时就会在 “Changes not staged for commit” 部分（也就是 *未暂存清单*）看到：

```shell
$ rm README.md
$ git status
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

然后再运行 `git rm` 记录此次移除文件的操作：

```console
$ git rm README.md
rm 'README.md'
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    deleted:    README.md
```

下一次提交时，该文件就不再纳入版本管理了。 如果要删除之前修改过或已经放到暂存区的文件，则必须使用强制删除选项 `-f`（译注：即 force 的首字母）。 这是一种安全特性，用于防止误删尚未添加到快照的数据，这样的数据不能被 Git 恢复。

另外一种情况是，我们想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。 换句话说，你想让文件保留在磁盘，但是并不想让 Git 继续跟踪。 当你忘记添加 `.gitignore` 文件，不小心把一个很大的日志文件或一堆 `.a` 这样的编译生成文件添加到暂存区时，这一做法尤其有用。 为达到这一目的，使用 `--cached` 选项：

```console
$ git rm --cached README
```

`git rm` 命令后面可以列出文件或者目录的名字，也可以使用 `glob` 模式。比如：

```console
$ git rm log/\*.log
```

注意到星号 `*` 之前的反斜杠 `\`， 因为 Git 有它自己的文件模式扩展匹配方式，所以我们不用 shell 来帮忙展开。 此命令删除 `log/` 目录下扩展名为 `.log` 的所有文件。 类似的比如：

```console
$ git rm \*~
```

该命令会删除所有名字以 `~` 结尾的文件。

## 3.撤销操作

### 1.撤销操作

有时候提交完了才发现漏掉了几个文件没有添加，或者提交的信息写错了。此时，可以运行带有 --amend选项的提交命令来重新提交：

```shell
$ git commit --amend
```

这个命令会将暂存区中的文件提交。如果自上次提交以来还未做任何修改，那么快照会保持不变，而你所修改的只是提交信息。文本编辑器启动后，可以看到之前的提交信息。编辑后保存会覆盖原来的提交信息。

例如，你提交后发现忘记了暂存某些需要的修改，可以像下面这样操作：

```shell
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

最终你只会有一个提交——第二次提交将代替第一次提交的结果。

### 2.取消暂存的文件

如果已将修改了两个文件并且想要将它们作为两次独立的修改提交，但是却意外地输入`git add *`暂存了它们两个。可以使用`git reset HEAD <file>...`来取消暂存。如下：

```shell
$ git reset HEAD README.md
```

### 3.撤销对文件的修改

如果不想保留对README.md文件的修改可以使用`git checkout`的操作来进行，如下：

```shell
$ git checkout README.md
```

在 Git 中任何 **已提交** 的东西几乎总是可以恢复的。 甚至那些被删除的分支中的提交或使用 `--amend` 选项覆盖的提交也可以恢复 。 然而，任何你未提交的东西丢失后很可能再也找不到了。

## 4.远程仓库的使用

### 1.查看远程仓库

如果想查看已经配置的远程仓库服务器，可以运行`git remote`命令。它会列出指定的每一个远程服务器的简写。如果已经克隆了自己的仓库，那么至少应该能看到origin——这是Git给克隆的仓库服务器的默认名字：

```shell
$ git clone https://github.com/raymeng27/goga
Cloning into 'goga'...
remote: Reusing existing pack: 1857, done.
remote: Total 1857 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (1857/1857), 374.35 KiB | 268.00 KiB/s, done.
Resolving deltas: 100% (772/772), done.
Checking connectivity... done.
$ git remote
origin
```

你也可以指定选项 `-v`，会显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL。

```shell
$ git remote -v
origin  https://github.com/raymeng27/studynotes.git (fetch)
origin  https://github.com/raymeng27/studynotes.git (push)
```

### 2.添加远程仓库

我们在之前的章节中已经提到并展示了 `git clone` 命令是如何自行添加远程仓库的， 不过这里将告诉你如何自己来添加它。 运行 `git remote add <shortname> <url>` 添加一个新的远程 Git 仓库，同时指定一个方便使用的简写

```shell
$ git remote
origin
$ git remote add pb https://github.com/paulboone/ticgit
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)
```

现在你可以在命令行中使用字符串 `pb` 来代替整个 URL。 例如，如果你想拉取 Paul 的仓库中有但你没有的信息，可以运行 `git fetch pb`：

```shell
$ git fetch pb
remote: Counting objects: 43, done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 43 (delta 10), reused 31 (delta 5)
Unpacking objects: 100% (43/43), done.
From https://github.com/paulboone/ticgit
 * [new branch]      master     -> pb/master
 * [new branch]      ticgit     -> pb/ticgit
```

现在 Paul 的 master 分支可以在本地通过 `pb/master` 访问到——你可以将它合并到自己的某个分支中， 或者如果你想要查看它的话，可以检出一个指向该点的本地分支。

### 3.从远程仓库中抓取与拉取

就如刚才所见，从远程仓库中获得数据，可以执行：

```console
$ git fetch <remote>
```

这个命令会访问远程仓库，从中拉取所有你还没有的数据。 执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。

如果你使用 `clone` 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为简写。 所以，`git fetch origin` 会抓取克隆（或上一次抓取）后新推送的所有工作。 必须注意 `git fetch` 命令只会将数据下载到你的本地仓库——它并不会自动合并或修改你当前的工作。 当准备好时你必须手动将其合并入你的工作。

### 4.推送到远程仓库

当你想分享你的项目时，必须将其推送到上游。 这个命令很简单：`git push <remote> <branch>`。 当你想要将 `master` 分支推送到 `origin` 服务器时（再次说明，克隆时通常会自动帮你设置好那两个名字）， 那么运行这个命令就可以将你所做的备份到服务器：

```console
$ git push origin master
```

只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。 当你和其他人在同一时间克隆，他们先推送到上游然后你再推送到上游，你的推送就会毫无疑问地被拒绝。 你必须先抓取他们的工作并将其合并进你的工作后才能推送。

### 5.查看某个远程仓库

如果想要查看某一个远程仓库的更多信息，可以使用 `git remote show <remote>` 命令。 如果想以一个特定的缩写名运行这个命令，例如 `origin`，会得到像下面类似的信息：

```console
$ git remote show origin
* remote origin
  Fetch URL: https://github.com/schacon/ticgit
  Push  URL: https://github.com/schacon/ticgit
  HEAD branch: master
  Remote branches:
    master                               tracked
    dev-branch                           tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

它同样会列出远程仓库的 URL 与跟踪分支的信息。 这些信息非常有用，它告诉你正处于 `master` 分支，并且如果运行 `git pull`， 就会抓取所有的远程引用，然后将远程 `master` 分支合并到本地 `master` 分支。 它也会列出拉取到的所有远程引用。

这是一个经常遇到的简单例子。 如果你是 Git 的重度使用者，那么还可以通过 `git remote show` 看到更多的信息。

```console
$ git remote show origin
* remote origin
  URL: https://github.com/my-org/complex-project
  Fetch URL: https://github.com/my-org/complex-project
  Push  URL: https://github.com/my-org/complex-project
  HEAD branch: master
  Remote branches:
    master                           tracked
    dev-branch                       tracked
    markdown-strip                   tracked
    issue-43                         new (next fetch will store in remotes/origin)
    issue-45                         new (next fetch will store in remotes/origin)
    refs/remotes/origin/issue-11     stale (use 'git remote prune' to remove)
  Local branches configured for 'git pull':
    dev-branch merges with remote dev-branch
    master     merges with remote master
  Local refs configured for 'git push':
    dev-branch                     pushes to dev-branch                     (up to date)
    markdown-strip                 pushes to markdown-strip                 (up to date)
    master                         pushes to master                         (up to date)
```

这个命令列出了当你在特定的分支上执行 `git push` 会自动地推送到哪一个远程分支。 它也同样地列出了哪些远程分支不在你的本地，哪些远程分支已经从服务器上移除了， 还有当你执行 `git pull` 时哪些本地分支可以与它跟踪的远程分支自动合并。

### 6.远程仓库的重命名与移除

你可以运行 `git remote rename` 来修改一个远程仓库的简写名。 例如，想要将 `pb` 重命名为 `paul`，可以用 `git remote rename` 这样做：

```console
$ git remote rename pb paul
$ git remote
origin
paul
```

值得注意的是这同样也会修改你所有远程跟踪的分支名字。 那些过去引用 `pb/master` 的现在会引用 `paul/master`。

如果因为一些原因想要移除一个远程仓库——你已经从服务器上搬走了或不再想使用某一个特定的镜像了， 又或者某一个贡献者不再贡献了——可以使用 `git remote remove` 或 `git remote rm` ：

```console
$ git remote remove paul
$ git remote
origin
```

一旦你使用这种方式删除了一个远程仓库，那么所有和这个远程仓库相关的远程跟踪分支以及配置信息也会一起被删除。