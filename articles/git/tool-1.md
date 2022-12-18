+ git 移除某文件

```
git rm -r --cache [资源]
//exp: 将dist 目录从git版本管理中移除
git rm -r --cache dist
```

+ 目录占位
有一空目录`views`，在其目录下新建`.keep`文件。git提交时，该目录会不会因为系统不同，而导致目录不存在

+ 前端包管理工具有你npm、cnpm、pnpm 、yarn，在项目开发中，需要固定以其中一个作为管理工具。为避免误操作管理工具不统一导致项目包管理有问题，可以在自己的bash脚本中做如下修改:

```
// .bashrc 默认包管理使用pnpm
alias npm='pnpm'
alias cnpm='pnpm'
alias yarn='pnpm'
```

```
alias 反悔，只需要在前面加上\
\npm
```

+ 文件或者目录重命名大写与小写，推荐操作：先重命名为一个随意的名称，再重命名为正确的名称。
原因：在不同的操作系统中，文件或者目录的大小写有的有区别有的没有，如果直接修改为最终的名称，在某些系统中是会识别失败的。

+ `git commit -v`打开vim，查看add后的文件变更详情，上面写上commit message信息提交。
+ linux 命令添加参数支持格式`-- --[name]=[value]`, exp:`-- --base=/a/b`
+ 执行`git commit -m [message]`命令后，发现message填写有误，做如下操作可以订正：`git reset [id]`，再重新执行提交流程。
+ 创建tag`git tag -a <tag_name> -m "message"`，exp:`git tag -a v3.0 -m "New release for v3.0"`
+ 根据tag创建分支`git checkout -b <newbranch> <tagName>`，exp:`git checkout -b v3.0-fix v3.0`
+ 根据commit 创建tag`git tag -a <tag_name> <commit_sha> -m "message"`，exp:`git tag -a v3.0 de12af -m "New release for v3.0"`
