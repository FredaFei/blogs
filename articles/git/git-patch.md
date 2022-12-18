+ 如何根据commit_id获知当前改动的文件内容

```
  git show commit_id
```
+ 如何根据commit_id应用该commit到其他分支或者其他项目

搜索关键字 `git create a patch from commit` [how-can-i-generate-a-git-patch-for-a-specific-commit](https://stackoverflow.com/questions/6658313/how-can-i-generate-a-git-patch-for-a-specific-commit)
```
  // 创建一个patch文件
  git format-patch -1 [commit_id]
  // 将file.patch文件放置到最终需要应用的分支或者其他项目
  mv file.patch [path]
  // apply patch
  git am < file.patch
```

+ `git diff` 对比差异

+ `git commit . --amend -m [message]` 将当前修改与上一次的提交信息合并

+ `git reflog` Reference logs（参考日志) 

场景：当前版本代码报错，但上午提交时并未出现该问题。排查方式就是查看上午之后的所有提交记录，二分法查找，做代码回滚和差异比较。
1. git reflog
2. git reset --hard [commit_id]
3. git show [commit_id]
4. 依次重复2、3步，直到找出关键问题改动
