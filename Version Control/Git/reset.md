# **git reset**
> 事情是这样的，我在本地建立了一个git仓库，用来存储我的SDK源码，但是由于当前源码过于庞大，每次编译和烧录都需要过长的时间，我希望可以剪裁出一个精简版本的代码，专门用来进行各个部分的调试。但是在此之前，我对源码进行过一次commit，这次commit我也不再需要，希望可以将其删除，因此在这里阅读了`git reset`部分的`manual`。


`git reset` 指令用于重置当前HEAD到一个特定的状态。

## 摘要
```
    git reset [-q] [<tree-ish>] [--] <pathspec>...
    git reset [-q] [--pathspec-from-file=<file> [--pathspec-file-nul]] [<tree-ish>]
    git reset (--patch | -p) [<tree-ish>] [--] [<pathspec>...]
    git reset [--soft | --mixed [-N] | --hard | --merge | --keep] [-q] [<commit>]
```

## 描述
上述前三种形式，从 `<tree-ish>` 复制条目到索引中。在第四种形式中，通过匹配可选地修改索引和工作树来设置当前分支的头（ `HEAD` ）到 `<commit>` 。在所有格式中 `<tree-ish>` / `<commit>` 默认为 `HEAD` 。

### `git reset [-q] [<tree-ish>] [--] <pathspec>..., git reset [-q] [--pathspec-from-file=<file> [--pathspec-file-nul]] [<tree-ish>]`
这两种写法会把所有匹配 `<pathspec>` 的路径的索引条目重置到 `<tree-ish>`中它们的状态。（这对工作树或当前分支不构成影响。）

这意味着 `git reset <pathspec>` 与 `git add <pathspec>` 是相反的。这个指令相当于 `git restore [--source=<tree-ish>] --staged <pathspec>...` 。

在使用 `git reset <pathspec>` 来更新索引目录后，你可以使用 `git restore` 来存放索引之外的内容到工作树中。另外，使用 `git restore` 以及通过 `--source` 选项来指定一个 *commit* ，你可以一次性拷贝一个 *commit* 之外的路径目录到索引以及工作树中。

### `git reset (--patch | -p) [<tree-ish>] [--] [<pathspec>...]`
交互式的选择索引和 `<tree-ish>` （默认为 `HEAD` ）之间的差异块。被选中的块被用来逆转回到索引处。

这意味着 `git reset -p` 与 `git add -p` 相反，例如：你可以使用它来有选择性的重置区块。

### `git reset [<mode>] [<commit>]`
这个格式重置当前分支头到 `<commit>` 并且可能更新索引（将其重置到 `<commit>` 的树）并且工作树依赖于 `<mode>` 。如果 `<mode>` 省略了，默认为 `--mixed` 。 `<mode>` 必须是如下几种之一：

- `--soft` ：无论如何不会接触索引文件或者工作树（但是会重置 `HEAD` 到 `<commit>` ，就如其他模式那样）。这使得所有你修改过的文件都变成了 **"Changes to be committed"** （更改为已提交） ，就如 `git status` 将会表示的那样。
- `--mixed` ：重置索引文件但不操作工作树（例如：修改后的文件会被保留，但是不会被标记为已提交）并且报告那些未被更新。这个模式是默认行为。
- `--hard` ：重置索引和工作树，所有在 `<commit>` 之后于工作树中对已跟踪文件的更改都会被丢弃。
- `--merge` ：重置索引并更新工作树中在 `<commit>` 与 `HEAD` 之间不同的文件，但是保留保留那些在索引和工作树之间存在差异的文件（例如那些更改但没有 `add` 的文件）。如果一个文件在 `<commit>` 和未保存的改变之间存在差异，重置将会终止。
  
  换句话说， `--merge`做了一些类似于 `git read-tree -u -m <commit>` 做的事情，但是转移了为合并的索引条目。

- `--keep` ：重置索引条目并更新工作树中在 `commit` 和 `HEAD` 之间存在差异的文件。如果一个在 `<commit>` 和 `HEAD` 存在差异的文件有本地更改，重置将会终止。

## 可选项：
### `-q, --quite, --no-quite`
保持沉默，只报告错误。默认行为通过 `reset.quite` 配置选项设置。 `--quite` 和 `--no-quite` 将会覆盖默认行为。

### `--pathspec-from-file=<file>`
**Pathspec** 将被传递到 `<file>` 来替代命令行参数。如果 `<file>` 是完全的——那么标准输入将被使用。**Pathspec** 元素被 **LF** 或 **CR/LF** 分隔。按照配置变量 `core.quotePath` 的解释，**pathspec** 元素可以被引用。

### `--pathspec-file-nul`
只在 `--pathspec-from-file` 时有意义。**Pathspec** 元素被 **NUL** 字符分隔所有其他字符按照字面意思处理（包括换行和引号）。

### `--`
不解释任何其他参数作为选项

### `<pathspec>`
限制受操作影响的路径。

## 实例：
### 撤销 `add`
```shell
$ edit      #你做了一些修改，并完成了本地测试，感觉一切正常，打算提交代码
$ git add frotz.c filfre.c
$ mailx     #有邮件告知你需要重新拉取代码，且这些更新对你的代码存在影响
$ git reset #然而，你已经污染了索引（你的索引已经不匹配HEAD了）。
            #因此你回复对于这两个文件的索引的修改。你的修改依然保存在工作树中。
$ git pull git://info.example.com/ nitfol
```

### 撤销一个提交并重做
```shell
$ git commit ...
$ git reset --soft HEAD^    #你经常会发现自己的提交并不完整，或者你的提交信息有拼写错误
                            #保留工作树与reset之前一致
$ edit                      #将工作树中的文件修改正确
$ git commit -a -c ORIG_HEAD #重置拷贝旧的头到.git/ORIG_HEAD；
                             #通过启动它的日志信息来重写提交信息。
                             #如果你不需要编译信息，你可以替换为一个 -C 选项。
```

### 撤销一个提交，将其变为一个主题分支
```shell
$ git branch topic/wip   #你有一些提交，但是你发掘他们放在master分支上为之过早。
                         #此时，你想在一个主题分支上继续打磨这个部分，
                         #因此创建了一个不在当前HEAD下的topic/wip分支
$ git rest --hard HEAD~3 #倒回master分支来摆脱这三个提交
$ git switch topic/wip   #选择到topic/wip分支并且继续工作
```

### 永久撤销提交
```shell
$ git commit ...
$ git reset --hard HEAD~3 #最近的三次提交(HEAD，HEAD^，HEAD~2)比较糟糕，因此你
                          #不想再看到它们。不要这样做如果你已经将这些提交交给了别人。
```

### 撤销一个 `merge` 或者 `pull`
```shell
$ git pull                         #尝试从上游更新造成了很多的冲突，你还未准备好花费大量时间
                                   #来合并它们，因此你决定之后再做这件事
Auto-merging nitfol
CONFLICT (content): Merge conflict in nitfol
Automatic merge failed; fix conflicts and then commit the result.
$ git reset --hard                 #pull 还没有创建合并提交，因此 git reset --hard 是
                                   # git reset --hard HEAD的一个同义词，可以用来清理当
                                   #前索引文件和工作树的混乱
$ git pull . topic/branch          #合并一个话题分支到当前分支，会造成一个Fast-forward
Updating from 41223... to 13134...
Fast-forward
$ git reset --hard ORIG_HEAD       #但是你认为话题分支还未做好供公众使用的准备。 pull 或 
                                   #merge总是离开ORIG_HEAD的当前分支的原始尖端，因此将其
                                   #硬重置会使你的索引文件和工作树回到它的状态，并且重置分支
                                   #的尖端到这个提交
```

### 在一个污染的工作树中撤销一个 `merge` 或者 `pull` 
```shell
$ git pull                    #即使你的工作树中可能有一些本地修改，当你确定这些在其他
                              #分支中的改变不会与它们重叠，你就可以安全的使用git pull
Auto-merging nitfol
Merge made by recursive.
nitfol                |   20 +++++----
...
$ git reset --merge ORIG_HEAD #在检查完合并之后，你可能发现其他分支下的修改可能不令人满意。
                              #运行 git reset --hard ORIG_HEAD 将会使你返回你之前所在
                              #的位置，但这会放弃你本地的修改。 git reset --merge 会保留
                              #你的本地修改。
```

### 中断工作流
假设当给你在一个巨大的修改中被一个紧急修复需求中断。在你的工作树中的文件还不处于任何要提交的状态，但是你需要到另一个分支进行快速修复。
```shell
$ git switch feature   #你处在一个叫 feature 的分支中
$ work work work       #在此处被中断
$ git commit -a -m "snapshot WIP"   #此提交将被删除，因此可以使用一个丢弃的日志消息
$ git switch master       #回到master分支
$ fix fix fix             #进行你的修复
$ git commit              #提交真正的日志
$ git switch feature      #返回feature分支
$ git reset --soft HEAD^  #返回到WIP状态，这会从提交中移除WIP提交，并且设置你的工作树到你创建
                          #快照之前的状态
$ git reset               #在这个点，索引文件仍有所有你提交为snapshot WIP的WIP修改。
                          #这将更新索引来将你的WIP文件显示为未提交
```

