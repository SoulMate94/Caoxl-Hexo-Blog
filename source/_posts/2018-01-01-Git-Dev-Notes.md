---
title: Git 开发记录
date: 2018-1-1 11:42:50
categories: Git
tags: Git
---
### 总结Git操作

> 参考别人的记录

> - [Git 命令无需记忆](http://kisss.cjli.info/cheatsheet/Git-Brief.html)
> - [Git 常见问题](http://kisss.cjli.info/auxiliary/Git-FAQ.html)

<!--more-->

### CentOS 更新 GIT
```
    yum install -y http://opensource.wandisco.com/centos/7/git/x86_64/wandisco-git-release-7-2.noarch.rpm
    yum update -y git
```

### .gitignore
 - 从版本库中移除（停止追踪）某个文件／夹
 ```
    # 停止追踪未修改且未放到暂存区的文件 file
    git rm --cached file
    # 停止追踪未修改且未放到暂存区的目录 dir
    git rm -r --cached dir
    # 停止追踪修改且已经放到暂存区的文件 file
    git rm -f --cached file
    # 停止追踪修改且已经放到暂存区的目录 dir
    git rm -rf --cached dir
 ```

*说明:* 一般情况下，项目最开始就需要在 .gitignore 中规定好哪些文件或者文件夹是不需要纳入 Git 控制的。但是难免会出现中途不想让 Git 控制某个文件的意图，

中途把文件／文件夹移除版本库的提交操作，仍然会存在于提交历史中。

- gitignore 文件写法示例
```
    # `#` 开头为注释
    # `*` 表示忽略所有文件
    *
    # `!` 表示排除某个文件
    !.gitignore
```

#### 保留子目录的某个文件
注意，如果要 gitignore 目录的所有文件，但是要保留目录（包括子目录）名，这样写是无效的:

```
    web/
    !web/static/.gitkeep
```

这样依然会把 web 目录的所有文件忽略掉，要保留 web 及其子目录 static 的正确的写法如下:
```
    web/*
    web/static/*
    !web/static/.gitkeep
```

*或者:*
```
    /vendor/*
    !/vendor/a
    !/vendor/a/b
    !/vendor/a/b/c
```


### git commit
 - 修改最后的提交说明
 ```
    git commit --amend
 ```
 - 添加新的更改
 ```
    git add . -A
    git commit -a --amend
    git commit -a --amend -m "deving at `date`"
 ```
 
### git fetch
```
    git checkout master
    git fetch origin master:mater_tmp
    git merge mater_tmp
```

### git rebase
 - 使用 rebase 合并分支
 和 merge 直接合并分支相比，rebase 可以使提交历史变得线性。比如，要把 test 分支合并到 master。
 ```
    git checkout test
    git rebase master    # 这一步和 merge 相同，可能会遇到冲突，也需要人工解决，人工解决后继续 rebase
    git rebase --continue
    git checkout master
    git merge test
 ```
 
 - 删除／合并没意义的提交内容
 ```
    # rebase 最近 100 次提交记录
    git rebase -i HEAD~100
    # VIM 操作
    :%s/pick/fixup/g    # 改为 fixup 模式
    :1    # 跳至第一行 修改 fixup 为 pick 或 reword 必须要保留一次提交 否则 git 会终止 rebase 进程
    # 终止 rebase 进程
    git rebase --abort
    # 继续 rebase 进程
    git rebase --continue
 ```
 
 - rebase 操作的含义：
 ```
    pick => use commit => 完整保留本次提交记录和信息
    reword => use commit, but edit the commit message => 会保留提交记录 但是会自动指引你到改变提交信息界面
    edit => use commit, but stop for amending => 保留提交记录 但是 rebase 进程会停止 需要手动 amend 后再 continue
    squash => use commit, but meld into previous commit => 会保留选中的提交记录文本
    fixup => like "squash", but discard this commit's log message => 会丢弃选中的提交记录文本
    exec => run command (the rest of the line) using shell
    drop => remove commit => 会删除提交时仓库对应的改动
 ```
 
 - rebase 进程是分支共享的
 意思是，你在 test 进行了一半的 rebase 操作，如果要在 dev 上执行新的 rebase 命令，则必须先执行 git rebase --abort 才可以，否则 git 会报错。
 
 
### git cherry
> Apply the changes introduced by some existing commits.
> 应用某些现有提交所引入的更改。
 
- 查看所有在 test 分支而不在 master 分支的提交
```
    git cherry -v master test
```

### cherry-pick
> A cherry-pick in Git is like a rebase for a single commit. It takes the patch that was introduced in a commit and tries to reapply it on the branch you’re currently on.

- 在某个分支应用整个仓库中存在的提交
```
    git cherry-pick <commit_hash>
```

- git revert
git revert is the converse of git chery-pick。后者用于将某次 commit 应用到某个分支，前者用于将某次 commit 从某个分支中移除。
- 移除某次非 merge-commit 并自动创建一次新的 commit hash

```
   git revert <commit-hash> 
```

- 移除某次非 merge-commit，不自动创建新的提交记录
```
    git revert -n <commit-hash>
    git revert --no-commit <commit-hash>
```

- 撤销当移除某次非 merge-commit 时自动创建了提交的记录的操作
```
    git reset HEAD~1 --soft
```

- 移除某次 merge-commit 并自动创建新的 commit hash
```
    # 指定 mainline 为合并提交记录中的第 1 个分支
    git revert -m 1 <commit-hash>
    # 指定 mainline 为合并提交记录中的第 2 个分支
    git revert -mainline 2 <commit-hash>
```
> 在你合并两个分支并试图撤销时，Git 并不知道你到底需要保留哪一个分支上所做的修改。从 Git 的角度来看，master 分支和 dev 在地位上是完全平等的，只是在 workflow 中，master 被人为约定成了「主分支」。
> 
> 于是 Git 需要你通过 m 或 mainline 参数来指定「主线」。merge commit 的 parents 一定是在两个不同的线索上，因此可以通过 parent 来表示「主线」。m 参数的值可以是 1 或者 2，对应着 parent 在 merge commit 信息中的顺序。


### 配置
```
    # 查看当前配置
    git config --local -l
    git config --help
```

### tag
 - 获取远程 tag
 ```
    # 允许 fetch 获取 tag
    git config --unset-all remote.origin.fetch
    git config --add remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'
    git config --add remote.origin.fetch '+refs/tags/*:refs/tags/*'
    # 获取远程所有 tags
    git fetch --tags
    # 获取远程某个 tag
    git fetch origin +refs/tags/<TAG_NAME>
 ```
 
### 重置／撤销
 - 重置单个已修改但未提交的文件（工作区）为上次提交时的状态
 ```
    # 1. 有风险的写法
    git checkout /path/to/modified_file
    # 2. 安全写法
    git checkout -- /path/to/modified_file
    # 3. 或者：移除未被跟踪的文件和目录
    git clean -fd
    # OR 对其他分支中的文件进行这种操作
    # man git-checkout
 ```
 
 由于 checkout 命令同样可以用于操作分支，因此，如果刚好出现要重置的文件名和某个存在的分支名同名的话，则最好使用第二种写法。
 
 - 重置当前工作区的所有未提交的改动（谨慎使用）
 ```
    git reset --hard
 ```
 
 - 重置已提交到版本库中的提交
 ```
    git reset --hard HEAD^      # 回退到前 1 个版本
    git reset --hard HEAD^^     # 回退到前 2 个版本
    git reset --hard HEAD~10    # 回退到前 10 个版本
    git reset --hard <hash_long>    # 通过完整 hash 回退到某个确定的版本
    git reset --hard <hash_short>   # 通过短 hash 回退到某个确定的版本
 ```
 
 - 重置已推送到远程仓库的提交：
 ```
    git reset --hard <commit-hash>
    git push -f origin master
 ```
 
 如果推送失败，只是 GitLab 或 GitHub 中设置了保护分支，可以暂时取消，会滚后。
 
 - 将改动从暂存区撤销到工作区
 ```
    git reset HEAD
 ```
 
 - 暂存改动
 ```
    git add -A
    git stash
    git stash clear
 ```
 
 - 查看暂存内容
 ```
    git stash show -p stash@{1}
 ```
 
### git diff
```
    git diff            # Diff the working copy with the index
    git diff --cached   # Diff the index with HEAD
    git diff HEAD       # Diff the working copy with HEAD
    # 查看两次提交记录的区别
    git diff <hash-1> <hash-2> > /path/to/file
```

### git branch
- 删除本地所有 master 和当前分支以外的分支
```
    git branch | grep -v "master" | xargs git branch -D
    git branch | egrep -v "(master|\*)" | xargs git branch -D
```
> Note that lowercase -d won’t delete a “non fully merged” branch.
> 注意小写字母D不会删除“非完全合并”分支。

### git show
 - 查看某次提交的改动详情
 ```
    git show <commit-hash>
 ```
 - 查看某次提交的改动简要统计
 ```
    git show <commit-hash> --stat
 ```
 
### git log
 - 查看历史提交改动详情
 ```
    git log -p
 ```
 - 统计某人代码量
 ```
    git log --author="$(git config --get user.name)" --pretty=tformat: --numstat | awk '{ add += $1 ; subs += $2 ; loc += $1 - $2 } END { printf "added lines: %s removed lines : %s total lines: %s\n",add,subs,loc }' -
 ```
 - 统计仓库所有开发者代码量
 ```
    git log --format='%aN' | sort -u | while read name; do echo -en "$name\t"; git log --author="$(git config --get user.name)" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -; done
 ```
 - 仓库提交者排名前 5（如果看全部，去掉 head 管道）
 ```
    git log --pretty='%aN' | sort | uniq -c | sort -k1 -n -r | head -n 5
 ```
 - 仓库提交者（邮箱）排名前 5
 ```
    git log --pretty=format:%ae | gawk -- '{ ++c[$0]; } END { for(cc in c) printf "%5d %s\n",c[cc],cc; }' | sort -u -n -r | head -n 5
 ```
 这个统计可能不会太准，因为很多人有不同的邮箱，但会使用相同的名字。
 
 - 贡献者统计
 ```
    git log --pretty='%aN' | sort -u | wc -l
 ```
 
 - 提交数统计
 ```
    # 提交总数
    git log --oneline | wc -l
    # 每个作者的提交数
    git shortlog -s
    git shortlog --numbered --summary
    git log --author="cxl" --oneline --shortstat
 ```
 
 - 添加或修改的代码行数
 ```
    git log --stat | perl -ne 'END { print $c } $c += $1 if /(\d+) insertions/;'
    git log --shortstat --pretty="%cE" | sed 's/\(.*\)@.*/\1/' | grep -v "^$" | awk 'BEGIN { line=""; } !/^ / { if (line=="" || !match(line, $0)) {line = $0 "," line }} /^ / { print line " # " $0; line=""}' | sort | sed -E 's/# //;s/ files? changed,//;s/([0-9]+) ([0-9]+ deletion)/\1 0 insertions\(+\), \2/;s/\(\+\)$/\(\+\), 0 deletions\(-\)/;s/insertions?\(\+\), //;s/ deletions?\(-\)//' | awk 'BEGIN {name=""; files=0; insertions=0; deletions=0;} {if ($1 != name && name != "") { print name ": " files " 个文件被改变, " insertions " 行被插入(+), " deletions " 行被删除(-), " insertions-deletions " 行剩余"; files=0; insertions=0; deletions=0; name=$1; } name=$1; files+=$2; insertions+=$3; deletions+=$4} END {print name ": " files " 个文件被改变, " insertions " 行被插入(+), " deletions " 行被删除(-), " insertions-deletions " 行剩余";}'
    git ls-files -z | xargs -0n1 git blame -w | ruby -n -e '$_ =~ /^.*\((.*?)\s[\d]{4}/; puts $1.strip' | sort -f | uniq -c | sort -n
 ```
 
 - 生成日报
 ```
    alias dg='git log --pretty=format:"- %s" --author="$(git config --get user.name)" --since=9am > ~/.today.md && subl ~/.today.md'
    git config alias.rb "log --pretty=format:'- %s' --author='$(git config --get user.name)' --since=9am"
 ```
 
### 核心概念
#### branch／tag／ commit
 - commit
 Git 中有且只有三大类型：blob、tree object、commit。
 - branch
 branch 并不是 Git 中过的特殊类型，一个 branch 无非是对一个被命名 commit 的引用，一个 branch 只是一个名字。
 > 因为一个提交可能有一个或者多个父代提交，而这些父代提交又有父代提交，这就允许将某一个提交就可以看作一个分支，因为它拥有全部的修改至它本身的历史。
 - tag
 tag 和 commit 本身也是相同的，唯一的不同是 tag 可以有自己的描述。（不然怎么能叫“标记”呢）
 
 
### 参考
 - [How to install latest version of git on CentOS 6.x/7.x](https://stackoverflow.com/questions/21820715/how-to-install-latest-version-of-git-on-centos-6-x-7-x)
 - [GIT pull/fetch from specific tag](https://stackoverflow.com/questions/3964368/git-pull-fetch-from-specific-tag)
 - [统计本地Git仓库中不同贡献者的代码行数的一些方法](http://eisneim.github.io/articles/2014-8-16-git-analize-caulculate-contribution.html)
 - [“git diff” does nothing](https://stackoverflow.com/questions/3580608/git-diff-does-nothing)
 - [Undo a particular commit in Git that’s been pushed to remote repos](https://stackoverflow.com/questions/2318777/undo-a-particular-commit-in-git-thats-been-pushed-to-remote-repos)
 - [Git 撤销合并](http://blog.psjay.com/posts/git-revert-merge-commit/)