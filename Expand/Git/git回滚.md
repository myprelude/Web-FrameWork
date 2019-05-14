>作者：myprelude@github  
原文链接： https://github.com/myprelude/Web-FrameWork/blob/master/README.md  
转载请注明出处，保留原文链接和作者信息。
# git 回滚

### git checkout 

```
git checkout -- <file>
```

将工作目录的文件进行回滚。

```
# 编辑了main.css文件
突然发现这个不是我们想要的我们不想一行行删除代码
# git checkout -- main.css
```

### git reset

```
git reset <commit>/HEAD
```
将本地仓库中未提交的代码进行回滚;有三种回滚方式：

* --soft 将已提交的代码恢复到暂存区，工作区不发生改变。相当于回滚git commit 操作
* --mixed（默认）将暂存区的代码恢复到指定状态，工作目录不发生改变。相当于恢复git add 操作
* --hard 将暂存区和工作区代码全部回滚。相当于删除之前操作。`注意git reset只是回滚已跟踪的文件对于没有被跟踪的文件我们一般通过git clean来清理类似rm操作`

```
git reset <file> 
```
类似git checkout 将工作目录指定文件的改动进行回滚。

### git revert

```
git revert <commit>/HEAD
```
将代码回滚到指定的版本。

```
将代码回滚到上连个版本
# git revert HEAD~2
```
### 如何区分和使用

git checkout 是将本地工作目录的修改进行回滚。一般情况下我们都可以直接使用 git reset 替代。

git reset 由于会重写提交历史，为了不对其他人产生误解我们使用在自己的私人分支上的代码回滚。

git revert 是同产生新的提交记录来覆盖我们之前的提交，比较安全。所有用在公共分支的代码回滚上。
