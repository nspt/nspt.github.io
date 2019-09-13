---
layout: default
title: Git：基本设置，基本操作与工作原理
---
- ## [1.让 Git 接手项目的历史管理](#1)
- ## [2.Git 的配置项](#2)
- ## [3.尝试提交和查看历史](#3)
	- ### [3.1记录新文件](#3-1)
	- ### [3.2记录修改](#3-2)
	- ### [3.3查看提交历史](#3-3)
- ## [4.Git 的基本原理](#4)
	- ### [4.1Git 如何存储历史](#4-1)
	- ### [4.2三个区域：工作区、暂存区和仓库](#4-2)
	- ### [4.3详解提交过程](#4-3)
	- ### [4.4记录文件的删除与重命名](#4-4)
- ## [额外内容](#extension)
	- ### [git log的其它常用参数](#extension)

<br/><br/><br/>

[上一章](./git_learning_1.html)我们了解了版本控制软件 Git 以及它的安装方法，而这一章我们将看到利用 Git 对项目进行版本控制需要哪些操作，以及这些操作背后的原理是什么。

不过在我们实际操作 Git 之前，需要说明的是，Git 虽然是“版本控制系统”，但其实管理的是“提交历史”。一般项目做出一定量的改动，比如修正了一个BUG后，我们就会进行一次提交（commit），commit 相当于告诉 Git：将项目当前的状态记录下来。换句话说，一次 commit 就产生项目的一个“历史节点”，而 Git 管理的就是由 commit 组成的历史。我们通过 commit 历史，就可以查看项目的历次改动，在必要时还可以将项目回退至某个 commit。

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/commit_history.jpg"/><br/>
commit历史示意图
</div>

值得强调的是，Git 并不会像个监控摄像头一样随时记录你在项目中的所有动作。所以当你想通过 Git 保存工作进度时，请 commit。如何 commit ？我们很快就会看到

<br id="1"/>

## 1.让Git接手项目的历史管理
Git 可以随时开始接手项目的历史管理，不论是从零开始的项目，还是已经有了一定进展的项目。想让 Git 开始管理某个项目，我们只需要进入该项目目录，然后执行 `git init` 命令即可：

```bash
$ cd MyProject
$ git init
Initialized empty Git repository in /home/nspt/MyProject
$ 
```

执行完 `git init` 后，通过命令 `ls -a` 查看项目中的所有文件和目录可以发现，项目中多出了一个隐藏目录：.git。这个 .git 目录就是 Git 的数据存放处，其中存储着本项目的所有历史、Git 配置和其它 Git 信息：

```bash
$ ls -a
./  ../  .git/
$ cd .git/
$ ls
config  description  HEAD  hooks/  info/  objects/  refs/
$ 
```

> _目前我们不需要在意 .git 目录的具体结构和存储内容，正常使用 Git 的情况下，对其抱有视而不见的态度即可（一般也不会看到 .git，因为以 '.' 作为名称开头的目录在 Unix/Linux 中是“隐藏目录”）_

<br id="2"/><br/>

## 2.Git的配置项
成熟的软件都会有可修改的配置项，以满足不同用户、不同情景的使用需求，Git 也不例外。

Git 的配置项分为三个级别：系统级、用户级、项目级。从字面意思就能明白，系统级配置影响整个系统的所有用户，用户级配置只影响本用户，项目级配置则只影响某个被 Git 管理的项目。此外，各个配置项优先级顺序是：项目级 > 用户级 > 系统级，以应对各级别配置项出现冲突的情况。

* Git 的系统级配置信息位于 /etc/gitconfig（默认情况下该文件可能不存在，仅在添加了系统级配置后出现）。想要在系统级 Git 配置中将配置项 attr 设为 value，使用命令：  
  `git config --system <attr> <value>`
* Git 的用户级配置信息位于 ~/.gitconfig，即用户 HOME 目录下的 .gitconfig 文件，想要在用户级 Git 配置中将配置项 attr 设为 value，使用命令：  
  `git config --global <attr> <value>`
* Git 的项目级配置信息位于项目目录中的 .git/config，即项目的 Git 仓库中的 config 文件，想要在项目级 Git 配置中将配置项 attr 设为 value，使用命令：  
  `git config --local <attr> <value>`

一般在 Linux 命令示例中，用<>包围的内容（如上面的 attr、value）为必须给出的自定义参数，用[]包围的内容则是可以给出也可以不给出的自定义参数。

如果使用 `git config` 时没有附加 `--system`、`--global` 或 `--local`，那么其默认修改项目级配置。Git 的配置非常丰富且复杂[<sup>1</sup>](#annotation)，本文不做过多介绍，下面只介绍两个必须配置项和一个建议配置项：

1. 用户信息配置（必须设置）

	上文提到过，Git 管理的实际上是由 commit 组成的“提交历史”，而在多人协作的项目中，commit 可能来自于不同的用户，为了方便日后查看历史，Git 强制要求每一次 commit 都必须声明该 commit 是由哪个用户完成的，以及该用户的邮箱是什么，即用户基本信息。为了实现这一点，Git 要求用户在 commit 前设置好两个配置项：user.name 和 user.email，当 commit 时，commit 的用户信息将根据 user.name 和 user.email 填写。

	一般我们会在用户级完成这两个配置项的设置：

	```bash
	git config --global user.name "nspt"
	git config --global user.email "xiewei9608@gmail.com"
	```

	> _如果你希望以另一个身份为某个项目做贡献，只需在该项目中设置项目级的 user.name 和 user.email，即可覆盖用户级的配置。_

2. 默认文本编辑器（建议设置）

	除了用户名和邮箱，Git 还要求每一次 commit 都给出“提交信息”，用于解释该 commit 提交的原因、目的等，在我们进行 commit 时，Git 会打开一个文本编辑器供我们输入提交信息，而打开哪一个文本编辑器则允许我们自定义。Git 所打开的文本编辑器会根据 core.editor 配置项决定，一般我们将该配置项设置为用户级：

	```bash
	git config --global core.editor "vim"
	```

	> _如果希望使用带有图形界面的文本编辑器，在 Ubuntu 可以设为 gedit。如果没有设置 core.editor，那么 Git 会采用系统默认文本编辑器，比如在 Ubuntu 中为 nano_

`git config --list` 可以查看当前的所有配置项，如果附带参数 `--show-origin`，则配置项的来源也会显示。

<br id="3"/><br id="3-1"/>

## 3.尝试提交和查看历史
完成 Git 基本配置，并在 MyProject 中执行 `git init` 后，我们的项目 MyProject 就可以开始利用 Git 了。假设我们的项目是一个从零开始的项目，目的是实现一个简单的复利计算器，用于计算在月利率固定的情况下，随着月存款的变化，总收益的变化。

>_Git 不仅仅可以用于软件项目的管理，对于 Git 来说它只是记录项目中内容的改变而已，内容是代码、普通文本还是二进制文件都无所谓，只是对于纯文本，Git 可以更好地显示其内容的改动情况。当然，一般情况下 Git 都是用于软件项目管理。_

>_所谓复利计算器只是一个“噱头”，我们只是希望通过将一个很简单的 C++ 程序分成很多步骤来完成，以模拟长期项目开发的情景，进而解释 Git 的各种功能罢了。_

### 3.1记录新文件
首先假设我们在 main.cpp 中搭好程序的基本框架：
```bash
$ vim main.cpp #用任意文本编辑器，编辑并保存 main.cpp
$ cat main.cpp
#include <iostream>

using namespace std;

int main()
{
    return 0;
}
$ 
```

搭好程序基本框架后，我们可以进行首次提交，让 Git 产生一个“历史节点”。这可以通过 `git add` 和 `git commit` 完成，我们先将希望被 Git 记录的新文件通过 `git add <file>` “标记”起来，然后执行一次 `git commit`：

```bash
$ git add main.cpp
$ git commit -m "Build the framework"
[master (root-commit) faa4aca] Build the framework
 1 file changed, 8 insertions(+)
 create mode 100644 main.cpp
$ 
```

>_可以通过 `git add .` 将当前目录下的所有文件一次性 add 起来（也可以通过 `git add file1 file2 ...` 将希望被 Git 管理的文件 `add` 起来）。`git add <dir>` 等同于对 dir 目录下的所有文件执行 `git add` 命令。_

>_`git commit` 不需要执行多次，一次 commit 执行一次即可_

`-m "Build the framework"` 的意思是本次 commit 不要打开文本编辑器，直接以 `-m` （可以理解为 message 的缩写）后面的参数 "Build the framework" 作为本次 commit 的提交信息。如果不使用 `-m` 参数，直接以 `git commit` 的形式进行 commit，则 Git 会打开配置项 core.editor 所指定的编辑器，并在其中以注释形式给出本次 commit 所带来的改动：

```bash
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch master
#
# Initial commit
#
# Changes to be committed:
#       new file:   main.cpp
#
```

“#” 开头的行为注释行，不会出现在 commit 的提交信息中。通过编辑器输入提交信息后，保存、退出，即可完成 commit。

`git commit` 执行成功后，一次 commit（包含项目快照的提交记录）就记录到了 Git 仓库之中，而我们的 main.cpp 也就被 Git “记录”了起来。

我们很快就会看到如何查看 Git 已经记录了的历史，在那之前，我们先来看看，如果对 main.cpp，一个已经被 `git add` 过的文件进行了修改，该如何进行 commit。

<br id="3-2"/>

### 3.2记录修改
现在假设我们为 main.cpp 添加了新代码，使 main.cpp 修改成了下面的样子：

```cpp
#include <iostream>
using namespace std;
int main()
{
    double deposit_m, interest_rate_m, total_income;
    return 0;
}
```

>_第5行就是我们新加入的代码，虽然只有一行，但足以代表一次修改。_

如果我们希望 Git 记录下 main.cpp 的“新样子”该怎么办？很简单，再做一次 commit，让 Git 记录一个新的“历史节点”：

```bash
$ git add main.cpp
$ git commit -m "Define variable"
[master 6ecec75] Define variable
 1 file changed, 1 insertion(+)
$ 
```

或许你会奇怪，main.cpp 之前已经被 `git add` 过了，Git 不是应该已经记录了它的存在吗？为什么不能直接 commit，而要对它再进行 `git add` ？这是因为 `git add` 针对的是 **改动**，其含义是 **将一个改动添加到暂存区**。记录一个未被记录的新文件，修改一个已记录了的文件，**甚至删除一个已记录的文件**，都是 **改动**，所以对待它们的方式是一样的：`git add`。另外，`git commit` 只会将被 `git add` 了的改动加入到本次 commit 中。关于暂存区的概念，我们在第四节会有更详细的解释，现在我们先来看看如何查看 commit 历史。

<br id="3-3"/>

### 3.3查看提交历史
项目中有 commit 历史后，我们便可以通过 `git log` 查看本项目的提交历史了：

```bash
$ git log
commit 6ecec75cb6f20aa35b717d4294dfcee65ea17464 (HEAD -> master)
Author: nspt <xiewei9608@gmail.com>
Date:   Thu Aug 1 16:11:38 2019 +0800

    Define variable

commit faa4acad0dc998bfbde640641b5d80972d0b945b
Author: nspt <xiewei9608@gmail.com>
Date:   Thu Aug 1 16:10:30 2019 +0800

    Build the framework

$ 
```

不难看出，`git log` 列出的历史是按时间倒序排列的，也就是时间越近的 commit 越上面，同时每一个 commit 都给出了作者、提交时间、提交信息。commit 后面的（十六进制）数字就是 commit id，可以用于唯一标识 commit，其详细介绍我们很快就会谈到。而 HEAD、master 的含义，我们在谈到 Git 分支时再做解释，目前可以暂时忽略。

想要查看某次 commit 所带来的具体改动内容，可以通过 `git show <commit id>` 的形式：

```bash
$ git show 6ecec75
commit 6ecec75cb6f20aa35b717d4294dfcee65ea17464 (HEAD -> master)
Author: nspt <xiewei9608@gmail.com>
Date:   Thu Aug 1 16:11:38 2019 +0800

    Define variable

diff --git a/main.cpp b/main.cpp
index f42c7aa..526dbd2 100644
--- a/main.cpp
+++ b/main.cpp
@@ -4,5 +4,6 @@ using namespace std;
 
 int main()
 {
+       double deposit_m, interest_rate_m, total_income;
        return 0;
 }
$ 
```

> _你可能会觉得奇怪，为什么 `git show` 后面的 commit id 是不完整的 6ecec75，却没有出错？那是因为目前以 6ecec75 开头的 commit id 只有一个。Git 可以通过 commit id 的起始部分确认完整的 commit id，以及对应的 commit 对象，只要这个起始部分足以确定一个独一无二的 commit id。以后我们还会接触到 tree id、blob id，它们和 commit id 一样，可以被 Git 通过起始部分确认完整的 id，只要这个起始部分是独一无二的。_

虽然 `git show 6ecec75` 展示内容的格式有点复杂，但我们依然可以从中看出该次 commit 改动的内容是什么：“diff --git a/main.cpp b/main.cpp” 意思是 main.cpp 做出了修改，修改内容则是 “+ double deposit_m, interest_rate_m, total_income;”，前面的 “+” 号表明这是新增的一行。如果你的命令行是支持颜色显示的，那么改动内容会以特殊显示显示。

如果通过支持 Git 的图形工具查看改动内容，那么结果会更直观，实际使用 Git 时若需要查看改动内容，用图形工具会是更好的选择。

> _除了专门用于 Git 的图形工具外，一些编辑器也可以借助插件实现对 Git 的支持，比如 VS Code。_

我们还可以通过 `git log -p` 或 `git log --patch` 按时间倒序查看所有 commit 的改动，这两个命令是等价的，会以 `less` 的工作模式按时间倒序滚动展示各个 commit 的改动。

> _`less` 是一个 Unix/Linux 工具，用于滚动查看很长的文本，基本使用方法为：J键下滚，K键上滚，Q键退出。_

`git log` 还支持以更简洁或更丰富的形式展示 commit 的改动，甚至支持自定义展示格式，不过由于并不常用，此处不作介绍，以免内容过于冗长。不过 `git log` 所支持的历史检索功能还是比较有用的，比如查出哪些 commit 改动了特定代码、特定文件等，希望对此有所了解的，可以前往本章结尾的[额外内容](#extension)处查看。

接下来的第四节中，我们将要介绍 Git 是如何看待 commit 和 commit 历史的，以及 Git 的关键设计：三个区域。明白 Git 的 “历史观” 以及 “三个区域” 的设计对于使用好 Git 至关重要。

<br id="4"/><br id="4-1"/>

## 4.Git的基本原理
### 4.1Git如何存储历史
我们已经提过，对于 Git 来说，项目历史就是由一次次 commit 组成的，并且每一次 commit 都是对当时项目状态的一个记录：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/commit_history.jpg"/><br/>
commit历史示意图
</div>

但我们每一次提交时，Git 到底做了什么呢？实际上，每一次 `git commit` 就是让 Git 创建一个 **commit 对象**，一个存储信息的结构。上图中的一个个 commit 就是一个个 commit 对象, Git 管理的历史其实就是 commit 对象组成的历史。

每一个 commit 对象都存储了 commit id、作者（Author）、提交时间（Date）、提交信息（Message）、上一个 commit（Parent）的 id 和一个 **项目快照** （Snapshot）：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/commit_object.jpg"/><br/>
<code>MyProject</code> 第二次提交所创建的 commit 对象示意图
</div>

其中 commit id 是通过对 commit 对象中的所有数据进行 hash 运算后得到的一串数字，这一串数字的特点就是：两个数据不同的 commit 对象，hash 后得到的数字几乎不可能相同[<sup>2</sup>](#annotation)。所以，这一串数字可以用于区分 commit 对象，就像是 commit 对象的身份证。除了项目历史上第一个 commit 对象，其它 commit 对象都保存着上一个 commit 对象的 id，因此我们只要能获取到一个 commit 对象，我们就能顺藤摸瓜般获取到从该 commit 开始，过去的所有历史。

> _Git 所采用的 hash 算法为 SHA-1，不论用于计算的数据量是多少，经过 SHA-1 运算后总是生成一个固定长度的数：160位的二进制数，不过一般以40位十六进制数的形式呈现出来，就像我们通过 `git log` 看到的那样。_

一个 commit 对象所存储的项目快照可以理解为该 commit 对象创建时，即我们执行 `git commit` 时的完整项目，相当于将当时的项目完整复制了一份并保存起来作为项目快照。当我们想要将项目回退到某个版本，即某个 commit 历史节点时，就可以通过将项目快照复制到项目目录以快速地完成。同理，查看某个 commit 所带来的改动，可以通过将该 commit 与其父 commit 进行项目快照的比对来完成。

**但是事实上 Git 不会在每个 commit 对象中保存一份完整项目！** 假如一个项目有1G大小，那么每次 commit 都在 .git 目录中存储一份1G的数据？这太浪费了。大型项目有很多文件，可能一次 commit 只是修改了其中一部分，或者完全没有修改已有内容而是添加/删除了文件，这种情况下大部分数据都是和过去一样的，如果每次 commit 都完整复制一遍整个项目，那未免太愚蠢。Git 实际上会对文件进行 **复用**。

假设我们添加了两个新文件：A 和 B，然后将他们 add 并 commit ，形成了 commit 1 。接着我们修改了 B ，并将修改后的文件 B' add 、 commit，形成了 commit 2。那么在 commit 1 中存储的项目快照将包含 A 、 B，而 commit 2 中存储的项目快照包含的则是“对 A 的引用”和 B'：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/file_reuse.jpg"/><br/>
文件复用示意图
</div>

commit 2 中“对A的引用”，就是对文件 A 的复用。当我们需要将项目回退到 commit 2 时，借助“对A的引用”，我们依然可以快速找到并恢复文件 A。相比于文件本身，“对文件的引用”小的可怜，只需要几十个字节就够了，通过复用，Git 可以节省下大量的空间，同时让使用者感觉每次 commit 都保留了完整的项目。不过因为其实际上没有保存完整项目，所以我们称 commit 对象中保存的是项目快照。

> _“对A的引用”，即 “对已记录文件的引用” 到底是什么，我们将在 Git 底层原理中介绍，不过在此可以做个小小提示：如果对文件 A 进行 SHA-1 运算，那我们就可以获得文件 A 的 id，一串唯一标识文件 A 的数字。_

> _实际上 commit 1 中也没有保存 A，其保存的也是 “对A的引用”，准确来说 commit 对象都没有实际保存文件，文件均是单独保存的，commit 对象只保存 “对文件的引用”，这样 commit 对象的尺寸就可以尽可能地小，而且解开文件与 commit 对象的强绑定关系。_

平时使用 Git 时，完全不需要关心底层到底如何存储历史数据，简单理解为每个 commit 对象都包含该历史节点下的完整项目即可。

<br id="4-2"/>

### 4.2三个区域：工作区、暂存区和仓库
虽然对 commit 对象的介绍让我们明白了 Git 是如何存储历史的，但 `git add` 与 `git commit` 究竟是如何完成 commit 对象的创建的，以及 `git add` 在提交过程中到底起到什么效果，目前仍不明确。要想搞清楚这部分知识，就需要对 Git 的工作方式或者说设计理念有所了解。

从逻辑上说，Git 将项目划分为三个区域：工作区（working tree）、暂存区（staging area）和仓库（repository）。

- 工作区就是项目目录下，除了 .git 目录以外的所有内容，也就是项目本身，我们实际工作的区域。
- 仓库就是负责存储 commit 历史的地方，对应的操作是 `git commit`。
- 暂存区就是一个项目快照，存储的是 **下一次 commit 所使用的项目快照**，其目的就是为 commit 提供一个 **缓存区**，对应的操作是 `git add`。

>_逻辑上仓库和暂存区是两个区域，当实际上二者的数据都位于 .git 目录中。_

工作区和仓库都很好理解，但是暂存区是干什么的？要想解释暂存区的意义，我们就得先明白一个需求：我们并不希望 Git 记录项目中的所有文件！

以 MyProject 为例，假设我们想要对代码进行测试，那么我们可能会将其编译成程序 MyProgram：

```bash
$ g++ main.cpp -c -o main.o
$ g++ main.o -o MyProgram
$ ls
main.cpp  main.o  MyProgram
$ 
```

通过 `ls` 可以看到，编译导致项目中出现了目标文件 main.o 和程序文件 MyProgram，而从项目管理的角度来说，这两个文件不需要被 Git 记录的，一方面只要有源代码，我们随时可以通过编译获得这些二进制文件，另一方面二进制文件的改动对于人类来说是难以解读的，记录二进制文件的改动几乎毫无意义（至少在软件项目中）。除了编译过程生成的二进制文件外，像文本编辑器的临时缓存文件、开发人员自己的记录文件和某些日志文件等，也都不需要被 Git 记录。

假设没有暂存区，也即没有 `git add` 只有 `git commit`，那么我们每次 commit 都得把工作区的所有内容记录下来形成项目快照及 commit 对象，此时那些我们不希望被记录的文件，也会被记录到 commit 历史中去。这不仅会导致仓库存储的数据量更大，还会导致查看某次 commit 带来的改动（`git show` 或 `git log -p`）时无用内容太多（我们并不想关心一个二进制程序的改动），从而影响使用。

而有了暂存区，我们就可以通过 `git add`，仅仅把希望被 Git 记录的改动添加到暂存区，然后通过 `git commit` 仅仅将暂存区内的改动添加到 commit 历史中。

除此之外，如果我们一次性完成了多个工作，但是逻辑上希望分成多次 commit 时，也可以借助暂存区的帮助。比如当我们一次性写了两个模块的代码，但是希望分成两次 commit 以细分改动历史时，就可以这样做：

```bash
$ git add module_a
$ git commit -m "Add module a"
$ git add module_b
$ git commit -m "Add module b"
```

之后通过 `git log` 查看历史时，模块 A 和模块 B 就是分两次 commit 提交的了，万一哪一个模块出了 BUG，定位起来也会更快。

上面解释的是暂存区存在的意义，即 **为 commit 提供缓存区，控制需要提交的内容**。但是，虽然我们的说法是 “把改动通过 `git add` 添加到暂存区”，实际上暂存区存储的是却一个项目快照，而不是单纯的改动。理解这一点非常重要，因此我们在此再次强调一遍：**暂存区其实就是一个项目快照**。执行 `git commit` 时 Git 新建的 commit 对象中的项目快照，就是复制的暂存区。

下面我们以 MyProject 的提交历史为例，看看这三个区域具体经历了什么变化，来加深对三个区域的理解。

首先，我们通过 `git init` 初始化了 MyProject，此时三个区域均为空，如下图：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/step1.jpg"/><br/>
<code>git init</code> 之后三个区域示意图
</div>

接着，我们编写了 main.cpp，此时仓库和暂存区依然为空，工作区有了 main.cpp，如下图：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/step2.jpg"/><br/>
编写 main.cpp 之后三个区域示意图
</div>

然后，我们通过 `git add` 将 main.cpp 加入到了暂存区，此时工作区和暂存区都有了 main.cpp（记住，暂存区就是一个项目快照，为了方便理解你甚至可以忽略 “快照”），但仓库依然为空，如下图：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/step3.jpg"/><br/>
<code>git add main.cpp</code> 之后三个区域示意图
</div>

最后，我们通过 `git commit` 生成了一个 commit 对象，这一步操作会将暂存区复制为新的 commit 对象的项目快照，再将新的 commit 对象加入仓库，此时三个区域的情况如下图：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/step4.jpg"/><br/>
首次 <code>git commit</code> 之后三个区域示意图
</div>

接着我们又对 main.cpp 进行了修改，假设改动后的文件为 main.cpp'，那么改动后，三个区域情况如下图：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/step5.jpg"/><br/>
修改 main.cpp 之后三个区域示意图
</div>

再次 `git add`，将改动后的 main.cpp，即 main.cpp' 加入到暂存区（将暂存区中的 main.cpp 替换成 main.cpp'）：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/step6.jpg"/><br/>
将 main.cpp' <code>git add</code> 之后三个区域示意图
</div>

最后，通过 `git commit` 完成我们的第二次提交，也就是创建第二个 commit 对象，这一步同样是先将暂存区拷贝为第二个 commit 对象的项目快照，再将 commit 对象加入仓库：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/step7.jpg"/><br/>
第二次 <code>git commit</code> 之后三个区域示意图
</div>

<br id="4-3"/>

### 4.3详解提交过程
通过画图，我们可以将项目的提交过程（即三个区域的变化）解释的非常清楚，但是我们不可能每次都通过画图的形式来查看 commit 的过程，或者说查看三个区域，尤其是暂存区的当前情况，所以我们需要用到一个新工具：`git status`。

`git status` 会显示目前项目的状态，比如哪些文件是新文件、哪些文件被修改了、哪些文件已经添加到暂存区了等等，相当于以文字形式表示当前三个区域间的状态。下面我们来看看一次提交过程中的不同阶段，`git status` 会有什么不同的结果。

我们刚才已经通过示意图看到了两次提交后的 MyProject 的三个区域情况如下：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/step8.jpg"/><br/>
第二次 <code>git commit</code> 之后三个区域示意图
</div>

那么，在两次提交后的 MyProject 中使用 `git status`，会得到如下内容：

```bash
$ cd MyProject
$ git status
On branch master
nothing to commit, working tree clean
$ 
```

> _“On branch master”的意思是我们现在位于 master 分支，这个可以暂时不管，等介绍 Git 分支时自然会明白。_

“nothing to commit” 的意思就是现在没什么可提交的，因为暂存区中的项目快照和仓库中最新的 commit 对象的项目快照一样。

“working tree clean” 的意思则是现在没什么可暂存的，因为工作区（working tree）的项目内容和暂存区中项目快照的内容一样。

可见，`git status` 是以文字形式描述三个区域的比对情况。

假设我们现在改动 main.cpp'，改动后的文件称为 main.cpp''，并且编译一次程序：

```bash
$ vim main.cpp
$ cat main.cpp
#include <iostream>
using namespace std;
int main()
{
        double deposit_m, interest_rate_m, total_income;

        cout<<"Please enter the monthly deposit"<<endl;
        cin>>deposit_m;
        cout<<"Please enter the monthly interest rate"<<endl;
        cin>>interest_rate_m;

        return 0;
}
$ g++ main.cpp -c -o main.o
$ g++ main.o -o MyProgram
```

那么三个区域的情况就会变成：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/step9.jpg"/><br/>
将 main.cpp' 修改成 main.cpp'' 并编译后三个区域示意图
</div>

此时 `git status` 会显示：
```bash
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   main.cpp

Untracked files:
  (use "git add <file>..." to include in what will be committed)

    MyProgram
    main.o
no changes added to commit (use "git add" and/or "git commit -a")
$ 
```

“Changes not staged for commit” 意思是下面的文件在暂存区中存在，但是工作区中的该文件做出了改动，导致和暂存区中的对应文件不一样，“modified: main.cpp” 则表明了改动是对 main.cpp 做了内容上的修改。（注意，改动并不仅仅包含对内容的修改，我们之前提到过，删除一个已记录文件也是一种改动，类似的，重命名等操作也是改动）

“Untracked files” 意思是下面的文件 main.o 和 MyProgram 在工作区存在，但暂存区中没有，属于未追踪文件（Untracked file）。

“no changes added to commit” 意思依然是现在没什么可 commit 的，这是因为暂存区的项目快照和仓库中最新的 commit 对象的项目快照还是一样的。不过因为工作区有新文件及改动了的文件，所以后面有提示 “use 'git add' and/or 'git commit -a'）”。

> _`git commit -a` 的意思是免去手动 `git add`，直接将所有改动一次性加入暂存区并进行 commit，相当于 `git add .` 后立马执行 `git commit`。_

同时我们可以看到，`git status` 还给出了对操作的提示，比如 “use 'git add <file>...' to update what will be committed” 告诉我们如果想要将改动了的文件加入到暂存区（从而准备被加入到下次 commit 中），要用 `git add <file>`，同理，“use 'git add <file>...' to include in what will be committed” 告诉我们对于未追踪文件，也是用 `git add` 加入到暂存区。

Git 的命令提示还是比较完善的，有时候我们即便不知道某个操作如何完成，也可以借助 Git 的提示达到目的，比如上面的提示：“use 'git checkout -- <file>...' to discard changes in working directory”，就告诉了我们该如何放弃对某个文件的改动，也就是将某个文件恢复到原来的样子（即暂存区中对应文件的样子）。不过关于撤销改动等操作，我们此处暂不做介绍，下一章再做详细解释，如果感兴趣的话，可以先按照 Git 的提示进行尝试。

现在，让我们将改动后的 main.cpp'' 加入暂存区：

```bash
$ git add main.cpp
```

此时三个区域的情况如下图：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/step10.jpg"/><br/>
将 main.cpp'' 加入暂存区后三个区域示意图
</div>

通过 `git status` 看到的内容如下：

```bash
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    modified:   main.cpp

Untracked files:
  (use "git add <file>..." to include in what will be committed)

    MyProgram
    main.o
$ 
```

因为修改后的 main.cpp'' 被我们加入到暂存区了，所以 `git status` 表示 main.cpp 是 “Changes to be commited”，也就是将要被 commit 的内容。

此时执行 `git commit`：

```bash
$ git commit -m "Prompt user to enter data"
[master 01370b9] Prompt user to enter data
 1 file changed, 6 insertions(+)
$ 
```

则三个区域会变成：

<div style="text-align:center">
<img src="./imgs/git_learning_blog/2/step11.jpg"/><br/>
<code>git commit</code> 后三个区域示意图
</div>

`git status` 将显示：

```bash
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

    MyProgram
    main.o

nothing added to commit but untracked files present (use "git add" to track)
$ 
```

因为此时工作区、暂存区和仓库中最新的 commit 对象中存储的都是同样的 main.cpp''，所以 `git status` 既没有提示 “Changes not staged for commit”，也没有提示 “Changes to be commited”，而 MyProgram 和 main.o 由于在暂存区中不存在，所以被标记为 “Untracked files”。

如果明白了三个区域，尤其是暂存区的概念，那么 `git status` 的内容就很好理解。同样，借助对 `git status` 输出内容的分析，也可以加深对三个区域概念的理解。

之前我们多次提到过，新增、修改甚至删除文件，都是改动，并且已经展示过记录新文件、修改文件的实际操作，下面我们就来看看，对于 “删除文件” 这样特殊的改动，究竟该如何记录。

<br id="4-4"/>

### 4.4记录文件的删除与重命名
在第三节中我们提到，不论是新增文件、修改文件还是删除文件，都是改动的一种形式，而 `git add` 就是针对改动，所以即便是删除文件，记录它的方式依然是 `git add`。但是当时我们并没有给出删除文件并记录的示例，因为在当时我们还未介绍三个区域（工作区、暂存区、仓库）的概念，对文件删除的记录理解起来会有些麻烦，不过现在我们已经知道三个区域的概念了，也就可以对记录删除文件做个尝试。

假设我们将 main.cpp 进一步修改后，希望将其暂存并 commit 时，不小心用了 `git add .` 命令，从而将 main.o 和 MyProgram 都给加入了暂存区，然后一起提交了：

```bash
$ cat main.cpp
#include <iostream>

using namespace std;

int main()
{
        double deposit_m, interest_rate_m, total_income;

        cout<<"Please enter the monthly deposit"<<endl;
        cin>>deposit_m;
        cout<<"Please enter the monthly interest rate"<<endl;
        cin>>interest_rate_m;

        cout<<"The total income after one year is:"<<endl;
        cout<<"Not finished"<<endl;
        return 0;
}
$ git add .
$ git commit -m "Oops,add some useless file"
[detached HEAD a4e5e62] Oops,add some useless file
 3 files changed, 2 insertions(+)
 create mode 100644 MyProgram
 create mode 100644 main.o
$ 
```

但是正如我们之前所说，我们并不希望中间文件和可执行文件等二进制文件被 Git 记录，所以我们需要将他们删除掉。

正常删除一个文件就是 `rm <file>`，我们先看看将一个已追踪文件 `rm` 后三个区域会是什么样：

```bash
$ rm MyProgram
$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    MyProgram

no changes added to commit (use "git add" and/or "git commit -a")
$ 
```

不出所料，MyProgram 的删除，被 Git 视为工作区与暂存区的一处不同，即一处改动。既然是改动，我们就可以通过 `git add` 将这个改动记录到暂存区：

```bash
$ git add MyProgram
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        deleted:    MyProgram
$ 
```

通过 `git add` 将 MyProgram （的删除改动）添加到暂存区，其实就是删除暂存区中的 MyProgram（再次强调，暂存区就是一个项目快照），或者说是用工作区的 MyProgram （不存在的文件）替换掉暂存区的 MyProgram。`git add MyProgram` 执行后暂存区和工作区的内容一致，所以 Git 没有给出 “Changes not staged for commit” 的提示。但是暂存区和仓库中最新的 commit 对象的项目快照有出入，即暂存区删除了 MyProgram 而最新的 commit 对象中有 MyProgram，所以 Git 提示了 “Changes to be committed”。

最后，我们通过 `git commit`，就可以将 MyProgram 的删除改动提交到仓库中去：

```bash
$ git commit -m "Remove MyProgram"
[master 5c8c510] Remove MyProgram
 1 file changed, 0 insertions(+), 0 deletions(-)
 delete mode 100644 MyProgram
$ 
```

上述记录文件删除的过程，分为了两个步骤：普通的 `rm` 和 `git add`，实际上我们也可以借助 `git rm MyProgram` 一次到位，其效果相当于 `rm MyProgram` 后立马执行 `git add MyProgram`。

除了通过 `rm` 实际删除一个文件，并将该删除改动记录起来，我们还可以通过 `git rm --cached <file>` 将一个文件从暂存区中删除，但保留工作区的该文件，下面以 main.o 为例：

```bash
$ git rm --cached main.o
rm 'main.o'
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        deleted:    main.o

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        main.o
$ 
```

从 “Changes to be committed” 可以看出，暂存区中的 main.o 已经被删除了，而从 “Untracked files” 可以看出，工作区依然保留着 main.o。此时提交，就会产生一个项目快照中没有 main.o 的 commit 对象，同时工作区依然保留 main.o：

```bash
$ git commit -m "Remove main.o"
[master cf46d26] Remove main.o
 1 file changed, 0 insertions(+), 0 deletions(-)
 delete mode 100644 main.o
$ ls
main.cpp  main.o
$ 
```

除了文件的删除，还有一种比较特殊的改动我们也曾提到：文件的重命名。其实文件的重命名会被 Git 视作两个已提到过的步骤：1.删除原文件 2.新建一个与原文件内容一样但文件名不一样的文件。现在我们将 main.cpp 重命名为 x.cpp，看看文件的重命名会被 Git 如何看待：

```bash
$ mv main.cpp x.cpp #此处将 main.cpp 重命名为 x.cpp
$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    main.cpp

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        main.o
        x.cpp

no changes added to commit (use "git add" and/or "git commit -a")
$ 
```

从上面 `git status` 的输出结果可以看出，Git 认为目前的情况是 main.cpp 被删除了，而工作区多出了一个 x.cpp。接着，我们只需要将 main.cpp 的删除改动添加到暂存区，然后将 x.cpp 作为新追踪文件添加到暂存区，即可完成一次文件的重命名：

```bash
$ git add main.cpp
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        deleted:    main.cpp

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        main.o
        x.cpp
$ git add x.cpp
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        renamed:    main.cpp -> x.cpp

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        main.o
$ 
```

由于重命名后的文件在内容上与原文件一摸一样，所以 Git 可以识别出这是一次重命名文件的改动，从而在 `git status` 显示 “renamed: main.cpp -> x.cpp”，如果希望提交的话，此时执行 `git commit` 即可，不过我们省略这一步。

<br/><br/>

通过本章学习到的 `git add` 和 `git commit`，我们可以记录新的改动，创建新的历史节点；通过 `git log` 和 `git show`，我们可以查看历史提交，以及它们带来的改动；而通过 `git status`，我们可以随时查看目前的项目状态，也即三个区域的情况。

但是，依然有很多问题我们尚未说明如何解决，比如：

1. 我有很多不想被 Git 记录的文件，但它们都会在 `git status` 中被视为 “Untracked files”，非常碍眼，我该怎么让 Git 忽略掉它们？
2. 我忘记我存入暂存区的修改内容具体是怎样的了，我想在 commit 之前确认一遍，该怎么查看？
3. 已经加入暂存区的改动，我想撤销或继续修改，该怎么办？
4. 我想将项目回退到某个历史节点，该怎么做？
5. 我只想将某个文件回退到某个历史节点的样子，该怎么做？
...

[下一章]()我们将介绍新的 Git 工具（不是高级工具，依然是基本工具），并解决这些问题。

<br/><br id="extension"/>

## 额外内容
### git log的其它常用参数
`git log` 支持对 commit 历史进行检索，或者说“过滤”，下面是其比较可能用到的过滤：

1. 如果你想找出改动了特定代码（改动特定代码包含加入该代码、删除改代码、改动改代码）的那些 commit，可以通过参数 `-S` 实现，比如查找改动了变量 "deposit_m" 的 commit，可以用命令 `git log -S "deposit_m"`
2. 如果你想找出改动过某个文件的 commit ，可以在 `git log` 命令的末尾添加 `-- file_path`，其中 file_path 即文件的路径（相对于当前位置，或绝对路径），比如查看改动过当前目录下 main.cpp 的 commit：`git log -- ./main.cpp`。
3. 如果你想找出在提交信息中包含某段特定语句的 commit，可以通过参数 `--grep` 实现，比如查找提交信息包含 “Remove” 的 commit：`git log --grep "Remove"`。
4. 如果你想找出某个时间之后，或某个时间之前的所有 commit，可以借助参数 `--since` 和 `--before`，比如
`git log --since="2019-01-15" --before="2019-02-15"` 会找出 2019 年 1 月 15 日至 2019 年 2 月 15 日之间的所有 commit。`--since` 和 `--before` 可以单独使用，也可以支持类似 `2 years 1 month 1 day 3 minutes ago` 形式的时间表示。

上述过滤可以互相结合，以实现精确的检索。

<br id="annotation"/>

注释：

> 1\. 没有人会去记 Git 的所有配置项，一般情况下我们只在需要利用 Git 配置完成某个目的时，再查询 Git 如何配置，比如“禁止删除服务器上的 commit 历史”。完整的 Git 配置文档可在 [Git 配置](https://stackoverflow.com/questions/9392365/how-would-git-handle-a-sha-1-collision-on-a-blob) 查询。

> 2\. 理论上参与 SHA-1 运算的数据不一样，也可以出现一样的结果数字，但是这种情况对于正常使用来说，完全不用担心，如果你对出现了该情况时 Git 会有什么表现感兴趣，可以参考 stackoverflow 网站上的这个[问题](https://stackoverflow.com/questions/9392365/how-would-git-handle-a-sha-1-collision-on-a-blob)。
