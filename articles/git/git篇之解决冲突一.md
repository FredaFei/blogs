最近在项目的版本管理上碰到了一些问题，下面记录下，便于日后作为经验参考。

### 场景复现

现有远程develop、team两个总分支，develop分支的src目录下有A.ts、a.ts两个文件，team分支的src目录下仅有A.ts文件。
还有远程develop-a、develop-b、develop-a-team、develop-b-team分支。其中develop-a、develop-b的代码是需要合并到远程develop，
develop-a-team、develop-b-team分支的代码是需要合并到远程team。最后将develop分支合并到team分支。
因develop-a分支的人因操作不正确，把本地的.git目录下对a.ts文件的状态记录被破坏，又将破坏的代码提交到远程的develop，再提交到team，导致合并过远程develop、team分支的人，
对a.ts文件的操作均无法正确提交。

### 解决方案
使用在线web idea，删除有问题分支（develop）的a.ts文件，保证远程所有有问题的分支中的a.ts文件均被移除。
团队中的每个成员先提交下自己的代码，对现有项目备份，再在各自电脑中找到该项目所在目录，删除所有文件（包括.git）,再执行

```
git fetch 
git reset --rebase
```
保证本地与远程同步了，最后push之前的commit到对应的分支

### 思考

+ 以上解决方案在原理上和直接clone一份新的远程代码类似，为何不clone一份新的？

这个咋一看是这样没错，但是直接clone一份新的，在已经基于坏的.git状态做的代码变更，会丢失部分commit，需要手动移植过来。

在远程仓库中，使用在线web idea，若未解决所有有问题的分支，坏的状态未清理干净的情况下，直接clone一份新的远程代码，只要切到有问题的分支，.git状态将再次被污染。（猜测）
