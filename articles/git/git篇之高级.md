#### 代码合并
合并的本质
+ 将两个及以上代码快照合并；
+ `git`逐行检查冲突，并尝试自动解决
+ 解决失败，报错

解决冲突

+ 手工选择正确版本
+ 严格按照命令行提示进行冲突解决， `--continue/--skip/--abort`

merge 和 rebase 解决冲突的方式主要有三种
+ create a merge commit
+ squash and merge
+ rebase and merge

#### create a merge commit
使用格式

```
    git merge [branch name]
```
合并后的示意图：

![merge 后的路线图](https://github.com/FredaFei/blogs/blob/master/articles/git/images/merge.jpg)

*优点：*
+ 操作简单；
+ 记录完整的历史；

*缺点：*
+ 历史记录杂乱无章；
+ 对bisect不友好；

```
    //放弃合并，代码回到合并前的状态
    git merge --abort
```

#### squash and merge
使用格式
```
    git merge [branch name] --squash
```
合并后的示意图：

![squash 后的路线图](https://github.com/FredaFei/blogs/blob/master/articles/git/images/squash.jpg)

*优点：*
+ 将所有变更合在一起，易于阅读，对bisect友好；
+ 想回滚或revert方便；

*缺点：*
+ 丢失所有历史记录；


#### rebase and merge
使用格式

```
    git rebase [branch name]
    // When you have resolved this problem
    git rebase --continue
    // If you prefer to skip this patch
    git rebase --skip
    // To check out the original branch and stop rebasing 
    git rebase --abort
```
合并后的示意图：

![rebase 后的路线图](https://github.com/FredaFei/blogs/blob/master/articles/git/images/rebase.jpg)


*优点：*
+ 分支历史是一条直线，清楚直白；
+ 对bisect友好；

*缺点：*
+ 操作较为复杂；
+ 如果冲突可能要重复解决（或者`squash`）；
+ 会干扰别人（遵循只在自己的分支`rebase`其他分支，共享分支不要`force push`）；

使用场景
+ 定期将你的分支和主干分支同步

#### 使用merge还是rebase?
+ 在任何共享的分支里，不要force push
+ 自己独占的分支，尽量用`rebase`


``` 
    git pull = git fetch + git merge
    git pull --rebase = git fetch + git rebase

    git checkout -b [branch name] = git branch [branch name] + git checkout [branch name]

    git commit --amend
    git reset origin/master --hard
    //thefuck github
```