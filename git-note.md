**前置条件：**

1. 注册Github账号，创建新的repository项目
2. 本地电脑安装git，生成key，并设置全区配置
3. 在github中添加ssh key
4. 在本地电脑git clone 项目




# 2 Git基础

## 2.1 取得项目的Git仓库

有两种取得 Git 项目仓库的方法。第一种是在现存的目录下，通过导入所有文件来创建新的 Git 仓库。第二种是从已有的 Git 仓库克隆出一个新的镜像仓库来。 

### 在工作目录中初始化新仓库

要对现有的某个项目开始用 Git 管理，只需到此项目所在的目录，执行：

```
$ git init
```

初始化后，在当前目录下会出现一个名为 .git 的目录，所有 Git 需要的数据和资源都存放在这个目录中。不过目前，仅仅是按照既有的结构框架初始化好了里边所有的文件和目录，但我们还没有开始跟踪管理项目中的任何一个文件。（在第九章我们会详细说明刚才创建的 `.git` 目录中究竟有哪些文件，以及都起些什么作用。）

如果当前目录下有几个文件想要纳入版本控制，需要先用 `git add` 命令告诉 Git 开始对这些文件进行跟踪，然后提交：

```
$ git add *.c
$ git add README
$ git commit -m 'initial project version'
```

稍后我们再逐一解释每条命令的意思。不过现在，你已经得到了一个实际维护着若干文件的 Git 仓库。

### 从现有仓库克隆

如果想对某个开源项目出一份力，可以先把该项目的 Git 仓库复制一份出来，这就需要用到 `git clone` 命令。如果你熟悉其他的 VCS 比如 Subversion，你可能已经注意到这里使用的是 `clone` 而不是 `checkout`。这是个非常重要的差别，Git 收取的是项目历史的所有数据（每一个文件的每一个版本），服务器上有的数据克隆之后本地也都有了。实际上，即便服务器的磁盘发生故障，用任何一个克隆出来的客户端都可以重建服务器上的仓库，回到当初克隆时的状态（虽然可能会丢失某些服务器端的挂钩设置，但所有版本的数据仍旧还在，有关细节请参考第四章）。

克隆仓库的命令格式为 `git clone [url]`。

## 2.2 记录每次更新到仓库

工作目录下面的所有文件都不外乎这两种状态：已跟踪或未跟踪。已跟踪的文件是指本来就被纳入版本控制管理的文件，在上次快照中有它们的记录，工作一段时间后，它们的状态可能是未更新，已修改或者已放入暂存区。而所有其他文件都属于未跟踪文件。它们既没有上次更新时的快照，也不在当前的暂存区域。初次克隆某个仓库时，工作目录中的所有文件都属于已跟踪文件，且状态为未修改。 

```
$ git status	# 查看当前文件的状态
$ git add [<file1> <file2> | .]	# 跟踪新文件
```

### 忽略某些文件

一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。我们可以创建一个名为 `.gitignore` 的文件，列出要忽略的文件模式。来看一个实际的例子：

```
$ cat .gitignore
*.[oa]
*~
```

第一行告诉 Git 忽略所有以 `.o` 或 `.a` 结尾的文件。一般这类对象文件和存档文件都是编译过程中出现的，我们用不着跟踪它们的版本。第二行告诉 Git 忽略所有以波浪符（`~`）结尾的文件，许多文本编辑软件（比如 Emacs）都用这样的文件名保存副本。此外，你可能还需要忽略 `log`，`tmp` 或者 `pid` 目录，以及自动生成的文档等等。要养成一开始就设置好 `.gitignore` 文件的习惯，以免将来误提交这类无用的文件。

文件 `.gitignore` 的格式规范如下：

- 所有空行或者以注释符号 `＃` 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配。
- 匹配模式最后跟反斜杠（`/`）说明要忽略的是目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（`!`）取反。

所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。星号（`*`）匹配零个或多个任意字符；`[abc]` 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；问号（`?`）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 `[0-9]` 表示匹配所有 0 到 9 的数字）。

我们再看一个 `.gitignore` 文件的例子：

```
# 此为注释 – 将被 Git 忽略
# 忽略所有 .a 结尾的文件
*.a
# 但 lib.a 除外
!lib.a
# 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
/TODO
# 忽略 build/ 目录下的所有文件
build/
# 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
doc/*.txt
```

### 查看已暂存和未暂存的更新

```
$ git diff 				# 查看尚未暂存的文件更新了哪些部分
$ git diff --cached 	 # 看已经暂存起来的文件和上次提交时的快照之间的差异
$ git diff --staged		 # 效果和上面的是相同的
```

### 提交更新

现在的暂存区域已经准备妥当可以提交了。在此之前，请一定要确认还有什么修改过的或新建的文件还没有 `git add` 过，否则提交的时候不会记录这些还没暂存起来的变化。所以，每次准备提交前，先用 `git status` 看下，是不是都已暂存起来了，然后再运行提交命令 `git commit`：

```
$ git commit
```

这种方式会启动文本编辑器以便输入本次提交的说明。（默认会启用 shell 的环境变量 `$EDITOR` 所指定的软件，一般都是 vim 或 emacs。当然也可以按照第一章介绍的方式，使用 `git config --global core.editor` 命令设定你喜欢的编辑软件。）

编辑器会显示类似下面的文本信息（本例选用 Vim 的屏显方式展示）：

```
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
# (use "git reset HEAD <file>..." to unstage)
#
# new file: README
# modified: benchmarks.rb
~
~
~
".git/COMMIT_EDITMSG" 10L, 283C
```

可以看到，默认的提交消息包含最后一次运行 `git status` 的输出，放在注释行里，另外开头还有一空行，供你输入提交说明。你完全可以去掉这些注释行，不过留着也没关系，多少能帮你回想起这次更新的内容有哪些。（如果觉得这还不够，可以用 `-v` 选项将修改差异的每一行都包含到注释中来。）退出编辑器时，Git 会丢掉注释行，将说明内容和本次更新提交到仓库。

另外也可以用 -m 参数后跟提交说明的方式，在一行命令中提交更新：

```
$ git commit -m "Story 182: Fix benchmarks for speed"
[master]: created 463dc4f: "Fix benchmarks for speed"
2 files changed, 3 insertions(+), 0 deletions(-)
create mode 100644 README
```

好，现在你已经创建了第一个提交！可以看到，提交后它会告诉你，当前是在哪个分支（master）提交的，本次提交的完整 SHA-1 校验和是什么（`463dc4f`），以及在本次提交中，有多少文件修订过，多少行添改和删改过。

记住，提交时记录的是放在暂存区域的快照，任何还未暂存的仍然保持已修改状态，可以在下次提交时纳入版本管理。每一次运行提交操作，都是对你项目作一次快照，以后可以回到这个状态，或者进行比较。

### 跳过使用暂存区域

尽管使用暂存区域的方式可以精心准备要提交的细节，但有时候这么做略显繁琐。Git 提供了一个跳过使用暂存区域的方式，只要在提交的时候，给 `git commit` 加上 `-a` 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤：

```
$ git status
# On branch master
#
# Changes not staged for commit:
#
# modified: benchmarks.rb
#
$ git commit -a -m 'added new benchmarks'
[master 83e38c7] added new benchmarks
1 files changed, 5 insertions(+), 0 deletions(-)
```

### 移除文件

要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交。可以用 `git rm` 命令完成此项工作，并连带从工作目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。

如果只是简单地从工作目录中手工删除文件，运行 `git status` 时就会在 “Changes not staged for commit” 部分（也就是*未暂存*清单）看到：

```
$ rm grit.gemspec
$ git status
# On branch master
#
# Changes not staged for commit:
# (use "git add/rm <file>..." to update what will be committed)
#
# deleted: grit.gemspec
#
```

然后再运行 `git rm` 记录此次移除文件的操作：

```
$ git rm grit.gemspec
rm 'grit.gemspec'
$ git status
# On branch master
#
# Changes to be committed:
# (use "git reset HEAD <file>..." to unstage)
#
# deleted: grit.gemspec
#
```

最后提交的时候，该文件就不再纳入版本管理了。如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 `-f`（译注：即 force 的首字母），以防误删除文件后丢失修改的内容。

另外一种情况是，我们想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。换句话说，仅是从跟踪清单中删除。比如一些大型日志文件或者一堆 `.a` 编译文件，不小心纳入仓库后，要移除跟踪但不删除文件，以便稍后在 `.gitignore` 文件中补上，用 `--cached` 选项即可：

```
$ git rm --cached readme.txt
```

后面可以列出文件或者目录的名字，也可以使用 glob 模式。比方说：

```
$ git rm log/\*.log
```

注意到星号 `*` 之前的反斜杠 `\`，因为 Git 有它自己的文件模式扩展匹配方式，所以我们不用 shell 来帮忙展开（译注：实际上不加反斜杠也可以运行，只不过按照 shell 扩展的话，仅仅删除指定目录下的文件而不会递归匹配。上面的例子本来就指定了目录，所以效果等同，但下面的例子就会用递归方式匹配，所以必须加反斜杠。）。此命令删除所有 `log/` 目录下扩展名为 `.log` 的文件。类似的比如：

```
$ git rm \*~
```

会递归删除当前目录及其子目录中所有 `~` 结尾的文件。

### 移动文件

不像其他的 VCS 系统，Git 并不跟踪文件移动操作。如果在 Git 中重命名了某个文件，仓库中存储的元数据并不会体现出这是一次改名操作。不过 Git 非常聪明，它会推断出究竟发生了什么，至于具体是如何做到的，我们稍后再谈。

既然如此，当你看到 Git 的 `mv` 命令时一定会困惑不已。要在 Git 中对文件改名，可以这么做：

```
$ git mv file_from file_to
```

它会恰如预期般正常工作。实际上，即便此时查看状态信息，也会明白无误地看到关于重命名操作的说明：

```
$ git mv README.txt README
$ git status
# On branch master
# Your branch is ahead of 'origin/master' by 1 commit.
#
# Changes to be committed:
# (use "git reset HEAD <file>..." to unstage)
#
# renamed: README.txt -> README
#
```

其实，运行 `git mv` 就相当于运行了下面三条命令：

```
$ mv README.txt README
$ git rm README.txt
$ git add README
```

如此分开操作，Git 也会意识到这是一次改名，所以不管何种方式都一样。当然，直接用 `git mv` 轻便得多，不过有时候用其他工具批处理改名的话，要记得在提交前删除老的文件名，再添加新的文件名。

## 2.3 查看提交历史

在提交了若干更新之后，又或者克隆了某个项目，想回顾下提交历史，可以使用 `git log` 命令查看。

默认不用任何参数的话，`git log` 会按提交时间列出所有的更新，最近的更新排在最上面。 每次更新都有一个 SHA-1 校验和、作者的名字和电子邮件地址、提交时间，最后缩进一个段落显示提交说明。`git log` 有许多选项可以帮助你搜寻感兴趣的提交，接下来我们介绍些最常用的。

```
# -p 选项展开显示每次提交的内容差异，用 -2 则仅显示最近的两次更新
$ git log -p -2		

# --stat仅显示简要的增改行数统计，每个提交都列出了修改过的文件，以及其中添加和移除的行数，并在最后列出所有增减行数小计。
$ git log --stat

# --pretty 选项，可以指定使用完全不同于默认格式的方式展示提交历史。另外还有 short，full 和 fuller 可以用，展示的信息或多或少有些不同
$ git log --pretty=oneline
$ git log --pretty=short

# 最有意思的是 format，可以定制要显示的记录格式，这样的输出便于后期编程提取分析
$ git log --pretty=format:"%h - %an, %ar : %s"
表 2-1 列出了常用的格式占位符写法及其代表的意义。

选项 说明
%H 提交对象（commit）的完整哈希字串
%h 提交对象的简短哈希字串
%T 树对象（tree）的完整哈希字串
%t 树对象的简短哈希字串
%P 父对象（parent）的完整哈希字串
%p 父对象的简短哈希字串
%an 作者（author）的名字
%ae 作者的电子邮件地址
%ad 作者修订日期（可以用 -date= 选项定制格式）
%ar 作者修订日期，按多久以前的方式显示
%cn 提交者(committer)的名字
%ce 提交者的电子邮件地址
%cd 提交日期
%cr 提交日期，按多久以前的方式显示
%s 提交说明

# 用 oneline 或 format 时结合 --graph 选项，可以看到开头多出一些 ASCII 字符串表示的简单图形，形象地展示了每个提交所在的分支及其分化衍合情况。
$ git log --pretty=format:"%h %s" --graph
```

表 2-2 还列出了一些其他常用的选项及其释义。

```
选项 说明
    -p 按补丁格式显示每个更新之间的差异。
    --stat 显示每次更新的文件修改统计信息。
    --shortstat 只显示 --stat 中最后的行数修改添加移除统计。
    --name-only 仅在提交信息后显示已修改的文件清单。
    --name-status 显示新增、修改、删除的文件清单。
    --abbrev-commit 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。
    --relative-date 使用较短的相对时间显示（比如，“2 weeks ago”）。
    --graph 显示 ASCII 图形表示的分支合并历史。
    --pretty 使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。
```

### 限制输出长度

除了定制输出格式的选项之外，`git log` 还有许多非常实用的限制输出长度的选项，也就是只输出部分提交信息。之前我们已经看到过 `-2` 了，它只显示最近的两条提交，实际上，这是 `-<n>` 选项的写法，其中的 `n` 可以是任何自然数，表示仅显示最近的若干条提交。不过实践中我们是不太用这个选项的，Git 在输出所有提交时会自动调用分页程序（less），要看更早的更新只需翻到下页即可。

另外还有按照时间作限制的选项，比如 `--since` 和 `--until`。下面的命令列出所有最近两周内的提交：

```
$ git log --since=2.weeks
```

你可以给出各种时间格式，比如说具体的某一天（“2008-01-15”），或者是多久以前（“2 years 1 day 3 minutes ago”）。

还可以给出若干搜索条件，列出符合的提交。用 `--author` 选项显示指定作者的提交，用 `--grep` 选项搜索提交说明中的关键字。（请注意，如果要得到同时满足这两个选项搜索条件的提交，就必须用 `--all-match` 选项。否则，满足任意一个条件的提交都会被匹配出来）

另一个真正实用的`git log`选项是路径(path)，如果只关心某些文件或者目录的历史提交，可以在 `git log` 选项的最后指定它们的路径。因为是放在最后位置上的选项，所以用两个短划线（`--`）隔开之前的选项和后面限定的路径名。

表 2-3 还列出了其他常用的类似选项。

```
选项 说明
    -(n) 仅显示最近的 n 条提交
    --since, --after 仅显示指定时间之后的提交。
    --until, --before 仅显示指定时间之前的提交。
    --author 仅显示指定作者相关的提交。
    --committer 仅显示指定提交者相关的提交。
```

来看一个实际的例子，如果要查看 Git 仓库中，2008 年 10 月期间，Junio Hamano 提交的但未合并的测试脚本（位于项目的 t/ 目录下的文件），可以用下面的查询命令：

```
$ git log --pretty="%h - %s" --author=gitster --since="2008-10-01" \
    --before="2008-11-01" --no-merges -- t/
    5610e3b - Fix testcase failure when extended attribute
    acd3b9e - Enhance hold_lock_file_for_{update,append}()
    f563754 - demonstrate breakage of detached checkout wi
    d1a43f2 - reset --hard/read-tree --reset -u: remove un
    51a94af - Fix "checkout --track -b newbranch" on detac
    b0ad11e - pull: allow "git pull origin $something:$cur
```

## 2.4 撤销操作

任何时候，你都有可能需要撤消刚才所做的某些操作。接下来会介绍一些基本的撤消操作相关的命令。请注意，有些撤销操作是不可逆的，所以请务必谨慎小心，一旦失误，就有可能丢失部分工作成果。

### 修改最后一次提交

有时候我们提交完了才发现漏掉了几个文件没有加，或者提交信息写错了。想要撤消刚才的提交操作，可以使用 `--amend` 选项重新提交：

```
$ git commit --amend
```

此命令将使用当前的暂存区域快照提交。如果刚才提交完没有作任何改动，直接运行此命令的话，相当于有机会重新编辑提交说明，但将要提交的文件快照和之前的一样。

启动文本编辑器后，会看到上次提交时的说明，编辑它确认没问题后保存退出，就会使用新的提交说明覆盖刚才失误的提交。

如果刚才提交时忘了暂存某些修改，可以先补上暂存操作，然后再运行 `--amend` 提交：

```
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

上面的三条命令最终只是产生一个提交，第二个提交命令修正了第一个的提交内容。

### 取消已经暂存的文件

使用```git add . ```命令加到暂存区域的文件，可以用如下命令使其回到已修改未暂存的状态

```
git reset HEAD <file>...
```

### 取消对文件的修改（慎重）

```
git checkout -- <file>...
```

注意：这条命令有些危险，使用该命令让文件恢复到修改前的版本，对文件的所有修改会丢弃！



## 2.5 远程仓库的使用

要参与任何一个 Git 项目的协作，必须要了解该如何管理远程仓库。远程仓库是指托管在网络上的项目仓库，可能会有好多个，其中有些你只能读，另外有些可以写。同他人协作开发某个项目时，需要管理这些远程仓库，以便推送或拉取数据，分享各自的工作进展。管理远程仓库的工作，包括添加远程库，移除废弃的远程库，管理各式远程库分支，定义是否跟踪这些分支，等等。本节我们将详细讨论远程库的管理和使用。

### 查看当前的远程库

```
# 它会列出每个远程库的简短名字
$ git remote	
# 也可以加上 -v 选项（译注：此为 --verbose 的简写，取首字母），显示对应的克隆地址：
$ git remote -v
```

### 添加远程仓库

要添加一个新的远程仓库，可以指定一个简单的名字，以便将来引用，运行 git remote add [shortname] \[url]：

```
$ git remote
  origin
$ git remote add pb git://github.com/paulboone/ticgit.git
$ git remote -v
  origin git://github.com/schacon/ticgit.git
  pb git://github.com/paulboone/ticgit.git
```

现在可以用字符串 `pb` 指代对应的仓库地址了。比如说，要抓取所有 Paul 有的，但本地仓库没有的信息，可以运行 `git fetch pb`：

```
$ git fetch pb
    remote: Counting objects: 58, done.
    remote: Compressing objects: 100% (41/41), done.
    remote: Total 44 (delta 24), reused 1 (delta 0)
    Unpacking objects: 100% (44/44), done.
    From git://github.com/paulboone/ticgit
    * [new branch] master -> pb/master
    * [new branch] ticgit -> pb/ticgit
```

现在，Paul 的主干分支（master）已经完全可以在本地访问了，对应的名字是 `pb/master`，你可以将它合并到自己的某个分支，或者切换到这个分支，看看有些什么有趣的更新。

### 从远程仓库抓取数据

```
$ git fetch [remote-name]
```

此命令会到远程仓库中拉取所有你本地仓库中还没有的数据。运行完成后，你就可以在本地访问该远程仓库中的所有分支，将其中某个分支合并到本地，或者只是取出某个分支，一探究竟。（我们会在第三章详细讨论关于分支的概念和操作。）

如果是克隆了一个仓库，此命令会自动将远程仓库归于 origin 名下。所以，`git fetch origin` 会抓取从你上次克隆以来别人上传到此远程仓库中的所有更新（或是上次 fetch 以来别人提交的更新）。有一点很重要，需要记住，**fetch 命令只是将远端的数据拉到本地仓库，并不自动合并到当前工作分支**，只有当你确实准备好了，才能手工合并。

如果设置了某个分支用于跟踪某个远端仓库的分支（参见下节及第三章的内容），可以使用 `git pull` 命令自动抓取数据下来，然后将远端分支自动合并到本地仓库中当前分支。在日常工作中我们经常这么用，既快且好。实际上，默认情况下 `git clone` 命令本质上就是自动创建了本地的 master 分支用于跟踪远程仓库中的 master 分支（假设远程仓库确实有 master 分支）。所以一般我们运行 `git pull`，目的都是要从原始克隆的远端仓库中抓取数据后，合并到工作目录中的当前分支。

### 推送数据到远程仓库

```
$ git push [remote-name] [branch-name]
```

如果要把本地的 master 分支推送到 `origin` 服务器上（再次说明下，克隆操作会自动使用默认的 master 和 origin 名字），可以运行下面的命令：

```
$ git push origin master
```

只有在所克隆的服务器上有写权限，或者同一时刻没有其他人在推数据，这条命令才会如期完成任务。如果在你推数据前，已经有其他人推送了若干更新，那你的推送操作就会被驳回。你必须先把他们的更新抓取到本地，合并到自己的项目中，然后才可以再次推送。有关推送数据到远程仓库的详细内容见第三章。

### 查看远程仓库信息

我们可以通过命令 `git remote show [remote-name]` 查看某个远程仓库的详细信息，比如要看所克隆的 `origin` 仓库，可以运行：

```
$ git remote show origin
```

上面的例子非常简单，而随着使用 Git 的深入，`git remote show` 给出的信息可能会像这样：

```
$ git remote show origin
    * remote origin
    URL: git@github.com:defunkt/github.git
    Remote branch merged with 'git pull' while on branch issues
    issues
    Remote branch merged with 'git pull' while on branch master
    master
    New remote branches (next fetch will store in remotes/origin)
    caching
    Stale tracking branches (use 'git remote prune')
    libwalker
    walker2
    Tracked remote branches
    acl
    apiv2
    dashboard2
    issues
    master
    postgres
    Local branch pushed with 'git push'
    master:master
```

它告诉我们，运行 `git push` 时缺省推送的分支是什么（译注：最后两行）。它还显示了有哪些远端分支还没有同步到本地（译注：第六行的 `caching` 分支），哪些已同步到本地的远端分支在远端服务器上已被删除（译注：`Stale tracking branches` 下面的两个分支），以及运行 `git pull` 时将自动合并哪些分支（译注：前四行中列出的 `issues` 和 `master` 分支）。

### 远程仓库的删除和重命名

在新版 Git 中可以用 `git remote rename` 命令修改某个远程仓库在本地的简称，比如想把 `pb` 改成 `paul`，可以这么运行：

```
$ git remote rename pb paul
```

注意，对远程仓库的重命名，也会使对应的分支名称发生变化，原来的 `pb/master` 分支现在成了 `paul/master`。

碰到远端仓库服务器迁移，或者原来的克隆镜像不再使用，又或者某个参与者不再贡献代码，那么需要移除对应的远端仓库，可以运行 `git remote rm`命令：

```
$ git remote rm paul
$ git remote
    origin
```

## 2.6 打标签

同大多数 VCS 一样，Git 也可以对某一时间点上的版本打上标签。

### 显示已有的标签

显示的标签按字母顺序排列，所以标签的先后并不表示重要程度的轻重。

```
$ git tag
```

我们可以用特定的搜索模式列出符合条件的标签。在 Git 自身项目仓库中，有着超过 240 个标签，如果你只对 1.4.2 系列的版本感兴趣，可以运行下面的命令：

```
$ git tag -l 'v1.4.2.*'
    v1.4.2.1
    v1.4.2.2
    v1.4.2.3
    v1.4.2.4
```

### 新建标签

Git 使用的标签有两种类型：轻量级的（lightweight）和含附注的（annotated）。轻量级标签就像是个不会变化的分支，实际上它就是个指向特定提交对象的引用。而含附注标签，实际上是存储在仓库中的一个独立对象，它有自身的校验和信息，包含着标签的名字，电子邮件地址和日期，以及标签说明，标签本身也允许使用 GNU Privacy Guard (GPG) 来签署或验证。一般我们都建议使用含附注型的标签，以便保留相关信息；当然，如果只是临时性加注标签，或者不需要旁注额外信息，用轻量级标签也没问题。

创建一个含附注类型的标签非常简单，用 `-a` （译注：取 `annotated` 的首字母）指定标签名字即可：

```
$ git tag -a v1.4 -m 'my version 1.4'
$ git tag
    v0.1
    v1.3
    v1.4
```

而 `-m` 选项则指定了对应的标签说明，Git 会将此说明一同保存在标签对象中。如果没有给出该选项，Git 会启动文本编辑软件供你输入标签说明。

可以使用 `git show` 命令查看相应标签的版本信息，并连同显示打标签时的提交对象。

```
$ git show v1.4
```

### 签署标签

如果你有自己的私钥，还可以用 GPG 来签署标签，只需要把之前的 `-a` 改为 `-s` （译注： 取 `signed` 的首字母）即可：

```
$ git tag -s v1.5 -m 'my signed 1.5 tag'
```

### 轻量级标签

轻量级标签实际上就是一个保存着对应提交对象的校验和信息的文件。要创建这样的标签，一个 `-a`，`-s` 或 `-m` 选项都不用，直接给出标签名字即可：

```
$ git tag v1.4-lw
```

### 验证标签

可以使用 `git tag -v [tag-name]` （译注：取 `verify` 的首字母）的方式验证已经签署的标签。此命令会调用 GPG 来验证签名，所以你需要有签署者的公钥，存放在 keyring 中，才能验证。

### 后期加注标签

你甚至可以在后期对早先的某次提交加注标签。比如在下面展示的提交历史中：

```
$ git log --pretty=oneline
    15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'experiment'
    a6b4c97498bd301d84096da251c98a07c7723e65 beginning write support
    0d52aaab4479697da7686c15f77a3d64d9165190 one more thing
    6d52a271eda8725415634dd79daabbc4d9b6008e Merge branch 'experiment'
    0b7434d86859cc7b8c3d5e1dddfed66ff742fcbc added a commit function
    4682c3261057305bdd616e23b64b0857d832627b added a todo file
    166ae0c4d3f420721acbb115cc33848dfcc2121a started write support
    9fceb02d0ae598e95dc970b74767f19372d61af8 updated rakefile
    964f16d36dfccde844893cac5b347e7b3d44abbc commit the todo
    8a5cbc430f1a9c3d00faaeffd07798508422908a updated readme
```

我们忘了在提交 “updated rakefile” 后为此项目打上版本号 v1.2，没关系，现在也能做。只要在打标签的时候跟上对应提交对象的校验和（或前几位字符）即可：

```
$ git tag -a v1.2 9fceb02
```

### push标签

默认情况下，`git push` 并不会把标签传送到远端服务器上，只有通过显式命令才能分享标签到远端仓库。其命令格式如同推送分支，运行 `git push origin [tagname]` 即可：

```
$ git push origin v1.5
    Counting objects: 50, done.
    Compressing objects: 100% (38/38), done.
    Writing objects: 100% (44/44), 4.56 KiB, done.
    Total 44 (delta 18), reused 8 (delta 1)
    To git@github.com:schacon/simplegit.git
    * [new tag] v1.5 -> v1.5
```

如果要一次推送所有本地新增的标签上去，可以使用 `--tags` 选项：

```
$ git push origin --tags
    Counting objects: 50, done.
    Compressing objects: 100% (38/38), done.
    Writing objects: 100% (44/44), 4.56 KiB, done.
    Total 44 (delta 18), reused 8 (delta 1)
    To git@github.com:schacon/simplegit.git
    * [new tag] v0.1 -> v0.1
    * [new tag] v1.2 -> v1.2
    * [new tag] v1.4 -> v1.4
    * [new tag] v1.4-lw -> v1.4-lw
    * [new tag] v1.5 -> v1.5
```

现在，其他人克隆共享仓库或拉取数据同步后，也会看到这些标签。



# 3 Git分支

​	Git 的分支可谓是难以置信的轻量级，它的新建操作几乎可以在瞬间完成，并且在不同分支间切换起来也差不多一样快。和许多其他版本控制系统不同，Git 鼓励在工作流程中频繁使用分支与合并，哪怕一天之内进行许多次都没有关系。理解分支的概念并熟练运用后，你才会意识到为什么 Git 是一个如此强大而独特的工具，并从此真正改变你的开发方式。 

## 3.1 何谓分支

为了理解 Git 分支的实现方式，我们需要回顾一下 Git 是如何储存数据的。或许你还记得第一章的内容，Git 保存的不是文件差异或者变化量，而只是一系列文件快照。

在 Git 中提交时，会保存一个提交（commit）对象，该对象包含一个指向暂存内容快照的指针，包含本次提交的作者等相关附属信息，包含零个或多个指向该提交对象的父对象指针：首次提交是没有直接祖先的，普通提交有一个祖先，由两个或多个分支合并产生的提交则有多个祖先。

为直观起见，我们假设在工作目录中有三个文件，准备将它们暂存后提交。暂存操作会对每一个文件计算校验和（即第一章中提到的 SHA-1 哈希字串），然后把当前版本的文件快照保存到 Git 仓库中（Git 使用 blob 类型的对象存储这些快照），并将校验和加入暂存区域：

```
$ git add README test.rb LICENSE
$ git commit -m 'initial commit of my project'
```

当使用 `git commit` 新建一个提交对象前，Git 会先计算每一个子目录（本例中就是项目根目录）的校验和，然后在 Git 仓库中将这些目录保存为树（tree）对象。之后 Git 创建的提交对象，除了包含相关提交信息以外，还包含着指向这个树对象（项目根目录）的指针，如此它就可以在将来需要的时候，重现此次快照的内容了。

现在，Git 仓库中有五个对象：三个表示文件快照内容的 blob 对象；一个记录着目录树内容及其中各个文件对应 blob 对象索引的 tree 对象；以及一个包含指向 tree 对象（根目录）的索引和其他提交信息元数据的 commit 对象。概念上来说，仓库中的各个对象保存的数据和相互关系看起来如图 3-1 所示：

![img](http://git.oschina.net/progit/figures/18333fig0301-tn.png)

 

图 3-1. 单个提交对象在仓库中的数据结构

作些修改后再次提交，那么这次的提交对象会包含一个指向上次提交对象的指针（译注：即下图中的 parent 对象）。两次提交后，仓库历史会变成图 3-2 的样子：

![img](http://git.oschina.net/progit/figures/18333fig0302-tn.png)

 

图 3-2. 多个提交对象之间的链接关系

现在来谈分支。Git 中的分支，其实本质上仅仅是个指向 commit 对象的可变指针。Git 会使用 master 作为分支的默认名字。在若干次提交后，你其实已经有了一个指向最后一次提交对象的 master 分支，它在每次提交的时候都会自动向前移动。

![img](http://git.oschina.net/progit/figures/18333fig0303-tn.png)

 

图 3-3. 分支其实就是从某个提交对象往回看的历史

那么，Git 又是如何创建一个新的分支的呢？答案很简单，创建一个新的分支指针。比如新建一个 testing 分支，可以使用 `git branch` 命令：

```
$ git branch testing
```

这会在当前 commit 对象上新建一个分支指针（见图 3-4）。

![img](http://git.oschina.net/progit/figures/18333fig0304-tn.png)

 

图 3-4. 多个分支指向提交数据的历史

那么，Git 是如何知道你当前在哪个分支上工作的呢？其实答案也很简单，它保存着一个名为 HEAD 的特别指针。请注意它和你熟知的许多其他版本控制系统（比如 Subversion 或 CVS）里的 HEAD 概念大不相同。在 Git 中，它是一个指向你正在工作中的本地分支的指针（译注：将 HEAD 想象为当前分支的别名。）。运行 `git branch` 命令，仅仅是建立了一个新的分支，但不会自动切换到这个分支中去，所以在这个例子中，我们依然还在 master 分支里工作（参考图 3-5）。

![img](http://git.oschina.net/progit/figures/18333fig0305-tn.png)

 

图 3-5. HEAD 指向当前所在的分支

要切换到其他分支，可以执行 `git checkout` 命令。我们现在转换到新建的 testing 分支：

```
$ git checkout testing
```

这样 HEAD 就指向了 testing 分支（见图3-6）。

![img](http://git.oschina.net/progit/figures/18333fig0306-tn.png)

 

图 3-6. HEAD 在你转换分支时指向新的分支

这样的实现方式会给我们带来什么好处呢？好吧，现在不妨再提交一次：

```
$ vim test.rb
$ git commit -a -m 'made a change'
```

图 3-7 展示了提交后的结果。

![img](http://git.oschina.net/progit/figures/18333fig0307-tn.png)

 

图 3-7. 每次提交后 HEAD 随着分支一起向前移动

非常有趣，现在 testing 分支向前移动了一格，而 master 分支仍然指向原先 `git checkout` 时所在的 commit 对象。现在我们回到 master 分支看看：

```
$ git checkout master
```

图 3-8 显示了结果。

![img](http://git.oschina.net/progit/figures/18333fig0308-tn.png)

 

图 3-8. HEAD 在一次 checkout 之后移动到了另一个分支

这条命令做了两件事。它把 HEAD 指针移回到 master 分支，并把工作目录中的文件换成了 master 分支所指向的快照内容。也就是说，现在开始所做的改动，将始于本项目中一个较老的版本。它的主要作用是将 testing 分支里作出的修改暂时取消，这样你就可以向另一个方向进行开发。

我们作些修改后再次提交：

```
$ vim test.rb
$ git commit -a -m 'made other changes'
```

现在我们的项目提交历史产生了分叉（如图 3-9 所示），因为刚才我们创建了一个分支，转换到其中进行了一些工作，然后又回到原来的主分支进行了另外一些工作。这些改变分别孤立在不同的分支里：我们可以在不同分支里反复切换，并在时机成熟时把它们合并到一起。而所有这些工作，仅仅需要 `branch` 和 `checkout` 这两条命令就可以完成。

![img](http://git.oschina.net/progit/figures/18333fig0309-tn.png)

 

图 3-9. 不同流向的分支历史

由于 Git 中的分支实际上仅是一个包含所指对象校验和（40 个字符长度 SHA-1 字串）的文件，所以创建和销毁一个分支就变得非常廉价。说白了，新建一个分支就是向一个文件写入 41 个字节（外加一个换行符）那么简单，当然也就很快了。

这和大多数版本控制系统形成了鲜明对比，它们管理分支大多采取备份所有项目文件到特定目录的方式，所以根据项目文件数量和大小不同，可能花费的时间也会有相当大的差别，快则几秒，慢则数分钟。而 Git 的实现与项目复杂度无关，它永远可以在几毫秒的时间内完成分支的创建和切换。同时，因为每次提交时都记录了祖先信息（译注：即 `parent` 对象），将来要合并分支时，寻找恰当的合并基础（译注：即共同祖先）的工作其实已经自然而然地摆在那里了，所以实现起来非常容易。Git 鼓励开发者频繁使用分支，正是因为有着这些特性作保障。

接下来看看，我们为什么应该频繁使用分支。

## 3.2 分支的新建与合并

现在让我们来看一个简单的分支与合并的例子，实际工作中大体也会用到这样的工作流程：

1. 开发某个网站。
2. 为实现某个新的需求，创建一个分支。
3. 在这个分支上开展工作。

假设此时，你突然接到一个电话说有个很严重的问题需要紧急修补，那么可以按照下面的方式处理：

1. 返回到原先已经发布到生产服务器上的分支。
2. 为这次紧急修补建立一个新分支，并在其中修复问题。
3. 通过测试后，回到生产服务器所在的分支，将修补分支合并进来，然后再推送到生产服务器上。
4. 切换到之前实现新需求的分支，继续工作。

### 分支的新建与切换

首先，我们假设你正在项目中愉快地工作，并且已经提交了几次更新（见图 3-10）。

![img](http://git.oschina.net/progit/figures/18333fig0310-tn.png)

 

图 3-10. 一个简短的提交历史

现在，你决定要修补问题追踪系统上的 #53 问题。顺带说明下，Git 并不同任何特定的问题追踪系统打交道。这里为了说明要解决的问题，才把新建的分支取名为 iss53。要新建并切换到该分支，运行 `git checkout` 并加上 `-b` 参数：

```
$ git checkout -b iss53
    Switched to a new branch "iss53"
```

这相当于执行下面这两条命令：

```
$ git branch iss53
$ git checkout iss53
```

图 3-11 示意该命令的执行结果。

![img](http://git.oschina.net/progit/figures/18333fig0311-tn.png)

 

图 3-11. 创建了一个新分支的指针

接着你开始尝试修复问题，在提交了若干次更新后，`iss53` 分支的指针也会随着向前推进，因为它就是当前分支（换句话说，当前的 `HEAD` 指针正指向 `iss53`，见图 3-12）：

```
$ vim index.html
$ git commit -a -m 'added a new footer [issue 53]'
```

![img](http://git.oschina.net/progit/figures/18333fig0312-tn.png)

 

图 3-12. iss53 分支随工作进展向前推进