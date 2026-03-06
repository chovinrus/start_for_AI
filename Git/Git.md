# Git

#### 参考视频

git基础，读文档

https://space.bilibili.com/364122352/lists/290009?type=season

手画图理解关键难点

https://space.bilibili.com/35894086/lists/941763?type=season

#### Git本地命令

配置身份

git config --global user.name "Your Name"

git config --global user.email "you@example.com"

初始化 & 首次提交

git init

git add . # 把改动放进暂存区

git commit -m "feat: add user service" # 生成一次快照查看状态与差异

git status # 哪些文件被改动

git diff # 具体改了什么

git log --oneline --graph # 简洁提交历史回滚与修正

git restore -- &lt;file&gt; #恢复工作区文件到最近一次提交的版本

git rm 删除某文件

#### git回滚操作

git revert 创建一个新的提交来“反向操作”之前的提交，以此撤销更改。这是一种“安全”的撤销方式，尤其适用于公共分支

git reset 回滚当前分支head到某一版本

git checkout 切到当前分支某个版本

#### Git查看日志

一行简洁

git log --oneline

图形化分支图

git log --oneline --graph --decorate --all

最近 n 条

git log -n 5

只看某文件

git log -p -- README.md

按作者过滤

git log --author="Alice"

时间区间

git log --since="2024-01-01" --until="2024-12-31"

#### git远程仓库

第一次添加

origin 是远程仓库的别名，可以随便起，但大家都习惯叫 origin。

地址支持 HTTPS、SSH、本地路径、Git 协议等任意合法 Git URL。

查看/验证

如果地址填错了，可修改：

若要同时添加第二个远程（例如上游官方仓库）：

绑定后就能 git fetch、git push、git pull 与远程仓库交互了。

IDEA为项目添加远程仓库

enable vcs之后稍等一会，添加对应的远程库名字和url即可

或者创建时从git克隆

ssh实现github免密授权，在github仓库账号添加客户端的ssh公钥即可，注意如果22端口不通可以使用**443端口**

也可以直接在.ssh目录下设置克隆的端口为443

#### git分支

创建分支 git branch &lt;branch-name&gt;

删除分支 git branch -d &lt;branch-name&gt;

合并分支 git merge &lt;branch-name&gt;

切换分支 git switch &lt;branch-name&gt;

查看所有分支 &lt;branch-name&gt;

#### git远程操作

git fetch 刷新远端分支的状态

git pull fetch并合并

git push 推送当前分支到远端分支

git stash 隐藏当前工作区的修改

git stash pop 恢复最近一次保存的更改并从 stash 中移除

#### ff fast-forward

fast-forward（快进）就是：Git 发现“当前分支”可以直接把指针滑到“目标分支”的最新提交，中间没有任何分叉，于是干脆不生成新的合并提交，让历史保持一条直线。

──────────────────

1 触发条件

本地分支必须是目标分支的祖先，即：

复制

A---B (main)

\\

C---D (origin/main)

此时 main 和 origin/main 没有分叉，只有 origin/main 领先 2 个提交。2 结果

执行 git merge origin/main 后，main 指针直接滑到 D，历史变成一条直线：

A---B---C---D (main, origin/main 同时指向 D)。不产生新的 merge commit

#### git merge和rebase 的区别

要注意rebase保留的本地版本节点是新节点，哈希值变化了，内容和原版本节点相同

#### git stash

两种使用场景：切换到其他分支修改内容，不想commit当前修改；push但未commit之前pull远端代码

不同分支对相同文件内容出现冲突一定要在本地解决，不能出现远端冲突，远端合并一般由领导完成（pull request请求时尽量不要出现冲突）。解决的方式是先pull再push。所以要习惯看到远端有commit和自己分支任务相关时及时拉取到本地

#### pull过程处理冲突的选项

- \--rebase 有冲突手动merge
- \--ff-only 只允许fast forward，ff失败则pull失败
- \--no-ff 即使快进也要求merge
- \--squash 压缩远端分支相对共同祖先的变化为一个commit
- \--no-commit 合并中，合并结果仅保留在工作区，需要手动commit
- \--no-verify 绕开git hooks 钩子函数

#### idea的patch选项

核心概念：什么是 Patch？

一个 Patch（补丁文件），通常以 .patch 或 .diff 为后缀，是一个文本文件，它按照特定格式描述了代码的差异（diffs）。它记录了哪些文件被修改了，以及每个文件中哪些行被添加、删除或更改。它本身不包含完整的文件内容，只包含变化。

在 IDEA 中，“Patch” 功能允许你以这种“补丁”为单位来操作你的更改，而不是以“文件”或“提交”为单位。

create patch

一个 Patch（补丁文件），通常以 .patch 或 .diff 为后缀，是一个文本文件，它按照特定格式描述了代码的差异（diffs）。它记录了哪些文件被修改了，以及每个文件中哪些行被添加、删除或更改。它本身不包含完整的文件内容，只包含变化。

在 IDEA 中，“Patch” 功能允许你以这种“补丁”为单位来操作你的更改，而不是以“文件”或“提交”为单位。

patch apply

将你尚未提交的本地更改（或某个特定提交中的更改）保存为一个独立的 .patch 补丁文件。这个文件可以发送给他人，或者自己备份起来。

apply时版本和create时版本不一致产生冲突git的解决方式

1.  大多数时候，我们无法保证环境完全一致。目标文件可能已经被其他人修改过。这时，Git 会尝试执行一个 “三路合并” 的智能过程：
2.  查找上下文： Git 会读取补丁中的“上下文”行，并在目标文件中寻找这些行。成功应用： 如果上下文行还存在且没有变化，Git 就能成功地将更改应用在正确的位置。
3.  冲突处理： 如果上下文所在的行区域已经被修改（例如，别人修改了同一行），Git 无法再自动判断如何合并，就会产生 “冲突（Conflict）”。

#### gitflow工作流

一个非常经典、结构化且成熟的分支管理模型，尤其适用于有计划发布周期和长期维护需求的项目。核心思想是使用严格的分支模型来定义不同的开发阶段和目的，为整个开发周期提供了一个清晰、规范的框架。它围绕着两个核心分支和若干辅助分支进行。

git flow工作流和source tree的使用：https://www.bilibili.com/video/BV1Ho4y1S7FL/?spm_id_from=333.337.search-card.all.click&vd_source=eae2f511976ee44b67dde481a31be83b

#### 生产力工具tortoisegit和sourcetree

tortoisegit

https://www.bilibili.com/video/BV1gV4y1k7TV/?spm_id_from=333.337.search-card.all.click&vd_source=eae2f511976ee44b67dde481a31be83b

sourcetree

https://www.runoob.com/git/source-tree-intro.html

**Interactively Rebase from Here**

**Interactively Rebase from Here**（从此处进行交互式变基）是一个强大的 Git 功能，它允许你**重新整理、修改甚至重写你当前的提交历史**。简单来说，它可以让你把一系列提交记录像整理积木一样重新摆放、合并或修改，从而在推送到远程仓库前，让你的提交历史更加清晰和整洁