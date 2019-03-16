>作者：myprelude@github  
原文链接： https://github.com/myprelude/Web-FrameWork/blob/master/README.md  
转载请注明出处，保留原文链接和作者信息。
# git reabse
如果你是处女座的强迫症患者，或者看不惯git的提交进程就像黄河一样一会汇集在一起一会又分流到各地，又或者受不了很多无意义的commit；那么这篇文章就很适合你，至少是给你一个解决方案。

### 场景一：合并分支，不产生新的提交记录

**用法：**`git rebase 分支名称`

```
            master
              |
        C1---C2
              |
            mywork
```

如上图：我们以origin分支为基准新建一个mywork分支，我们在mywork分支提交了2个commit；现在需要将mywork分子合并到origin分支；事情往往就是没有那么一帆风顺的，此时的origin分支已经抢先被其他同事提交了commit；如下图：

```
                  master
                    |
    C1---C2---C3---C4
          \
          C5---C6
                |
              mywork
```

一般我们都是：
```
git checkout origin
git merge mywork
```
```
                  master
                    |
    C1---C2---C3---C4
          \          \
          C5---C6----C7
                      |
                    mywork
```

不错问题是解决了；但是产生了一个新的commit 如上图C7;强迫症受不了了；能不能直接将我们要提交的C5 C6直接添加到C4后面而不是生成一个新的提交C7,给人感觉就像是用胶水将他们贴在一起打上了补丁。

rebase 来解决：
```
git rebase origin   // 效果图如下
```
```
                  master
                    |
    C1---C2---C3---C4
          \         \
          C5---C6    C5'---C6'
                            |
                         mywork
```

仔细查看图片，我们是C5 C6的提交变成了C5‘ C6’，然后添加到C4的后面；之前的C5 C6被删除了。形成了一条笔直的线了。那么为啥rebase可以做到呢。

**rebase后发生了什么：**  
当执行rebase命令后，git会将当前分支提交的commit全部提到暂存区中，找到需要合并的分支，然后将暂存区中的commit取出来放到该分支后面（这个过程如果有冲突我们还是要首先解决冲突）。这个过程是不是像`git cherry-pick`命令,`git cherry-pick`命令是将我们需要的commit提取到对应的分支上。其实`git rebase`内部就是使用了`git cherry-pick`。（如果感兴趣的话可以去了解cherry-pick）。

**git pull --rebase**  
通常情况下我们都是和同事在一个分支开发，你准备push你本次提交发现远程分支已经发生变动，git提醒你git pull，一般我们git pull就可以了。但是git pull 是`git fetch` 和 `git merge`命令的简写,既然merge了那么就会产生让人发狂的无用commit；解决这个无用的commit只需将 `git merge` 换成 `git rebase`就可以了； 当然他们也有简写：git pull --rebase。

### 场景二：对多个提交记录进行合并、删除、修改。

**用法：**`git rebase -i <commit>`

```
              master
                |
C1---C2---C3---C4
      \
       C5---C6---C7
                  |
                 dev
```
如上：我们功能完成后，准备将dev分支合并到master分支时；发现我们dev上的三次提交其实就是完成一个功能，没有必要做三个commit提交，这样不便于以后代码的历史追踪。如何解决？

**在当前分支 dev  
git rebase -i C2     //这里是开闭区间 也就是选择C2这个commit就是操作C2以后的提交**

出现如下交互命令行
```
pick c32c9da 提交commit1
pick c332dff 提交commit2--
pick c21dffd 提交commit3----

# Rebase 665b060..c32c9da onto 665b060 (3 command)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
~
```
最上面的几行使我们需要对其操作的commit信息；

Command里面的命令含义分别是：
* pick操作，git会应用这个补丁，以同样的提交信息（commit message）保存提交
* reword操作，git会应用这个补丁，但需要重新编辑提交信息
* edit操作，git会应用这个补丁，但会因为amending而终止
* squash操作，git会应用这个补丁，但会与之前的提交合并
* fixup操作，git会应用这个补丁，但会丢掉提交日志
* exec操作，git会在shell中运行这个命令

我们需要合并这个三个commit为一个，将需要保留的commit前面使用pick，需要合并的前面改为s 或者 squash：
```
pick c32c9da 提交commit1
s c332dff 提交commit2--
s c21dffd 提交commit3----
```
OK!保存退出。如果没有冲突，命令行会出现如下：
```
# This is a combination of 3 commits.
# This is the 1st commit message:
提交commit1 
# This is the commit message #2: 
提交commit3---- 
# This is the commit message #3: 
update1 
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Sun Apr 22 08:54:55 2018 +0800
#
# interactive rebase in progress; onto a9269a3
# Last commands done (3 commands done):
#    s a7186d8 update#    s 7b16b28 update6666
#  No commands remaining.
```
如果需要修改提交信息可以直接修改，然后保存退出。

出现冲突了我们需要手动解决冲突；然后执行 git rebase --contiue,如果想终止rebase操作 执行 git rebase --abort。

### 注意:
git rebase 好像就是精心为我们优化提交信息简化分支的；很多开发者都喜欢在代码合并到主分支之前，对自己的提交历史进行修改、编辑、合并、让整个项目看上去更加整洁。记住git rebase 会用新的提交替换旧的提交，你的项目历史会像突然消失了一样。所有你永远不应该 rebase 那些已经推送到公共仓库的提交；尤其是多人开发，因为 rebase 引起了新的提交，Git 会认为你的 master 分支和其他人的 master 已经分叉了。两个 master 分支的唯一办法是把它们 merge 到一起，导致一个额外的合并提交和两堆包含同样更改的提交。不用说，这会让人非常困惑。

** 黄金准则：** git rebase 应该在自己独立分支上操作完成后再提交到远程仓库，千万不要在远程仓库上操作rebase。