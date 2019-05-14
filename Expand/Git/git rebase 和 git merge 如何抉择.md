>作者：myprelude@github  
原文链接： https://github.com/myprelude/Web-FrameWork/blob/master/README.md  
转载请注明出处，保留原文链接和作者信息。
# git reabse 和 git merge 我们该如何抉择

```
                  master
                    |
    C1---C2---C3---C4
          \
          C5---C6
                |
              mywork
```

`git merge`
```
git checkout mywork
git merge master
                  master
                    |
    C1---C2---C3---C4
          \          \
          C5---C6----C7
                      |
                    mywork
```
`git rebase`
```
git checout mywork
git rebase master
                  master
                    |
    C1---C2---C3---C4
          \         \
          C5---C6    C5'---C6'
                            |
                         mywork
```
上面我们可以看见出来 git merge 和git rebase 都是将mywork分支合并到master分支上，唯一不同的是 git merge会生成一个merge commit(如上C7)，而git rebase 是复制一份mywork分支上的提交C5 C6，然后将复制的C5’ C6‘直接拼到master分支上面。

git merge 的方式我们很容易知道我们分支是怎么一步步演变的；如果分支比较多的话，分支和分支之间都有merge和并的话，那么我们对git提交记录就像一张网。如下：
```
              master
                    |
    C1---C2---C3---C4
          \          \
          C5---C6----C7
            |  /      |
            C8      mywork
            |
           fixed
```
git rebase 是喜欢简洁的git提交记录的救星；将我们改动直接copy到我们要合并的分支上面去，总个历史记录都是线性的在前进。假如我们要对开源贡献代码的话，一个都会用rebase去将我们代码合并到开发分支在提交，这样整个流程就很清晰。其实git rebase 不仅仅是可以合并代码还可以对我们多次提交的 commit 进行修改（详情见git rebase -i）。
### 注意点
git rebase 的好处和优点虽然，但是我们在使用时还是需要注意一点，git rebase会修改commit的信息生成新的commit，如果你把 master 分支 rebase 到你的 feature 分支上会发生什么：

这次 rebase 将 master 分支上的所有提交都移到了 feature 分支后面。问题是它只发生在你的代码仓库中，其他所有的开发者还在原来的 master 上工作。因为 rebase 引起了新的提交，Git 会认为你的 master 分支和其他人的 master 已经分叉了。

同步两个 master 分支的唯一办法是把它们 merge 到一起，导致一个额外的合并提交和两堆包含同样更改的提交。不用说，这会让人非常困惑。。在使用rebase的时候我们牢记，不要在远程公共分支上操作rebase，只要控制这个红线我们就可开心的使用rebase了。