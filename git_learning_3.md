- ## [1.忽略不需要被Git管理的文件：.gitignore](#1)
- ## [2.查看、撤销、继续改动当前改动](#2)
	- ### [2.1查看当前改动内容](#2-1)
	- ### [2.2暂存后继续改动](#2-2)
	- ### [2.3撤销暂存了的改动](#2-3)
- ## [3.回退到某个历史节点](#3)
	- ### [3.1回退整个项目](#3-1)
	- ### [3.2回退某个文件](#3-2)
	- ### [3.3为历史节点打标签](#3-3)
- ## [4.初探 HEAD](#4)
- ## [5.checkout 与 reset 的区别](#5)
- ## [6.初探分支](#6)
- ## [额外内容：小工具——Git别名](#extension)

<br/><br/><br id="1"/>

## 1.忽略不需要被 Git 管理的文件：.gitignore
上一章我们提到一个问题，就是项目中很容易出现我们不需要借助 Git 进行管理的文件，如目标文件、可执行文件、临时文件等，而这些文件的存在会导致我们不能使用像 `git add .` 这样的快捷的、针对目录的命令，并且通过 `git status` 查看当前三个区域的状态时也会显示出太多未追踪文件。所以，让 Git 忽略这些文件就成了一个迫切的需求，幸好 Git 提供了一个方法：.gitignore文件。

我们可以在项目目录下创建一个名字叫 .gitignore 的文件：

```bash
$ cd MyProject
$ touch .gitignore
```

然后在 .gitignore 中写入我们希望 Git 忽略的文件的文件名，一行一个：

```bash
$ vim .gitignore  #用任意文本编辑器编辑 .gitignore，以一行一个的形式记录希望忽略的文件的文件名
$ cat .gitignore
main.o
MyProgram
$ 
```

被 .gitignore 文件记录了的文件，会被 Git 忽略，不再显示在 `git status` 中，同时像 `git add .` 这样的针对目录的操作，也会忽略掉记录在 .gitignore 中的文件。比如我们在 .gitignore 中记录了 main.o 和 MyProgram 后，我们通过 `git status` 查看改动情况时，未追踪文件中就没有了 main.o 和 MyProgram 的踪影：

```bash
$ ls
main.cpp  main.o  MyProgram
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .gitignore

nothing added to commit but untracked files present (use "git add" to track)
$ 
```

> _.gitignore 虽然会被 Git 用于检查应该忽略的文件，但是其本身并不会默认被 Git 记录历史，所以目前也是以未追踪文件的身份存在。_

下面我们看看执行 `git add .` 会怎么样：

```bash
$ git add .
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   .gitignore
$
```

从 “Changes to be committed” 可以看出，项目目录下的 main.o 和 MyProgram 都没有被加入暂存区，原因就是他们在 .gitignore 中被记录了。

.gitignore 不仅支持精确忽略，即根据文件名对指定文件进行忽略，还支持模式匹配。举例来说，假设我们的项目很大，编译测试时会生成很多的 .o 文件，而我们又不想（也不应该）将所有的 .o 文件逐一记录在 .gitignore 中，那么我们只需要在 .gitignore 中记录一行 *.o 即可，其中 \* 表示 “任意字符串”，也就是说 main.o、x.o、y.o、z.o 等以 .o 结尾的文件，均符合 *.o 模式，从而都会被 Git 忽略掉。

.gitignore 支持的模式，或者说基本规则如下：
1. 空行和以 # 开头的行会被忽略。（所以我们可以在 .gitignore 中添加自己的注释，只需要将 # 作为注释内容所属的行的起始字符即可）
2. 支持标准的 glob 模式匹配（见本章[额外内容](#extension)）