# GitFlow 工作流指南

## Git 工作流简介

- 集中式工作流  - 小团队
- 功能分支工作流 - 8~12人小团队
- GitFlow 工作流
- Forking工作流  - 特别大的工作团队（比如跨国团队）
- Pull Requests  请求合并，同Merge Request

## Git 命令

### 配置用户名和邮箱

+ 命令
  + git config -- global user.name 'xxx'
  + git config -- global user.email xxx
  + git config -- list
  + git config user.name
  + git help 命令

### 管理git项目

+ git init

+ git add 文件名

+ git add .

+ git status

  ```shell
  mkdir git_demo
  cd git_demo
  git init
  git init git_demo #创建目录，并初始化
  $ touch index.html
  $ git status
  On branch master #当前分支
  
  No commits yet
  #没有被追踪的文件
  Untracked files:
    (use "git add <file>..." to include in what will be committed)
          index.html
  
  nothing added to commit but untracked files present (use "git add" to track)
  
  HP@LAPTOP-3JV4RUIN MINGW64 /d/workspace/git_demo (master)
  $ git add index.html #追踪该文件
  $ git status
  On branch master
  
  No commits yet
  
  Changes to be committed:
    (use "git rm --cached <file>..." to unstage)
          new file:   index.html
  #追踪所有没有追踪的文件
  git add .
  
  ```

### commit 的作用

将修改的文件生成一个版本

+ git commit
+ git commit -m '描述'
+ git commit -am '描述'     add和commit合并，只能操作已被追踪的文件。

### log 追踪

+ git log

+ git log -p -2  查看最近2次提交的log

+ git log --author=用户名

+ git log --oneline 每次提交只显示一行

+ git log --graph

+ git log --pretty=oneline

+ git log --pretty=format

  ```shell
  $ git log --pretty=format:"%h - %an, %ar : %s"
  6e53aff - Dongdonghe1981, 6 minutes ago : init ver
  221eed8 - Dongdonghe1981, 6 minutes ago : init
  21ad8af - Dongdonghe1981, 15 minutes ago : test
  %h : hash 值
  %an 作者
  %ar 多长时间之前
  %s 描述
  ```

### 追踪文件修改前后的区别

+ git diff 文件名
+ git diff --staged add 查看add阶段的不同

### 文件忽略

+ .gitignore 放到项目根目录下
+ /node_modules   忽略node_modules文件夹下所有文件
+ *.log 忽略.log结尾的文件
+ *.zip 忽略.zip结尾的文件
+ git rm -r --cached 文件.  将被追踪的文件删除掉，加入到忽略的文件中，再提交

### 一键还原

+ git checkout -- 文件名    恢复到上一次的状态，针对还没有add操作的文件

### 撤销追踪操作与文件还原

+ git reset HEAD 文件名      撤销当前文件的追踪，也就是撤销add操作
+ git checkout -- 文件名

### 版本回退

+ git reset --hard HEAD^      回退到上一个版本
+ git reset --hard HEAD^^    回退到上上个版本
+ git reset --hard hash号    回退到指定的hash版本
+ git reflog    指针理解

版本回退：v1 -> v2 -> v3   不保留版本号

### 回到旧版本

+ git log 
+ git checkout 哈希号 -- 文件名

回退某个版本: v1 -> v2 -> v3 -> v4  保留版本号

### 建立切换删除分支·

+ git branch         #查看分支
+  git checkout 分支名称          #切换分支
+ git checkout -b 分支名称           #建立和切换分支
+ git branch 分支名称 -d        #删除分支
+ git branch 分支名称 -D       #强制删除分支

如果分支上的文件有修改，-d 是不能删除分支的，需要m erge，或者-D强制删除。

### 合并分支

+ git merge 分支名

  将develop的内容merge到master的步骤

  1. 切换到master
  2. git merge develop 

### 解决合并时发生的冲突

+ git merge 分支名
+ git status 查看冲突原因
+ git merge --abort 忽略合并  在不清楚如何处理冲突的时候，选择撤销合并
+ 手动选择正确内容 
+ git commit ，输入如何处理冲突的说明，保存

```shell
<<<<<<< HEAD            #当前分支
master                  #内容
=======                 #分割
develop                 #内容
>>>>>>> develop         #要合并的分支

```

### 查看版本线图

+ git log
+ git log --oneline
+ git log --oneline --graph
+ git log --oneline --graph --all  查看所有分支
+ git log --oneline --graph -[number]  当前最新的几个版本线图

### 更多合并方法

+ git merge --no-off  分支名         不使用快转机制，保留merge线图

+ git merge --no-off --no-commit 分支名

  (master|MERGING)  合并完成，但并没有commit，如果测试完成，

  没有问题，再执行`git  commit`

+ git merge --squash 分支名  将被合并的版本进行压缩

  不使用`--squash`合并后的线图

  ```shell
  $ git log --oneline --graph
  *   8e7eb79 (HEAD -> master) Merge branch 'develop' m
  |\
  | * b2e6faa (develop) d v6
  | * b26d37a d v4
  | * cc98b2a d v3
  * | b92f689 Merge branch 'develop' master merge develop
  |\|
  | * 024d040 d v2
  |/
  * 6fd90b4 d v1
  * cd60e90 d v1
  * 1e48b83 first commit
  
  ```

  使用`--squash`的线图

  ```shell
  * afbb7e5 (HEAD -> master) Squashed commit of the following:
  *   b92f689 Merge branch 'develop' master merge develop
  |\
  | * 024d040 d v2
  |/
  * 6fd90b4 d v1
  * cd60e90 d v1
  * 1e48b83 first commit
  
  ```

  

+ git reset --hard ORIG_HEAD   回到上一次合并前的版本

### 一次性删掉不想要的分支

+ git branch --merged | egrep -v "(^\\*|master|develop)" | xargs git branch -d 

  ```shell
  #查看合并的分支
  $ git branch --merged
  * master
    v3
    v4
    v5
  #查看没有合并的分支
  $ git branch --no-merged
    develop
  
  #删除已经合并的分支
  $ git branch --merged | egrep -v "(^\*|master|develop)" | xargs git branch -d
  Deleted branch v3 (was 10a1953).
  Deleted branch v4 (was 10a1953).
  Deleted branch v5 (was 10a1953).
  
  ```

### 本地仓库推送到远端仓库

+ git push -u origin master

```shell
#添加远程仓库 origin 代替后面的url，即别名
git remote add origin https://github.com/Dongdonghe1981/newrepo.git
$ git remote
origin
#第一次向远程仓库提交分支的时候，一定要用--set-upstream
git push --set-upstream origin master  #上传到远端的master分支  --set-upstream等于-u
git checkout develop
git push --set-upstream origin develop  #上传到远端的develop分支

#查看追踪的远程仓库
$ git remote -vv
origin  https://github.com/Dongdonghe1981/newrepo.git (fetch)
origin  https://github.com/Dongdonghe1981/newrepo.git (push)
$ git remote
origin

#删除远程仓库
git remote remove origin
```

### GitHub作为项目服务器

1. 创建一个跟用户名完全相同.git.io的项目

2. 上传代码到该远程仓库
3. 在浏览器中直接输入创建的远程仓库的名字，就可以浏览该仓库中代码运行的效果

### 如何获取远程项目

+ git clone 远程仓库地址  项目名

  自动进行checkout

+ git clone --no-checkout

  不checkout，下载的目录是空，进到目录中，再执行`git checkout master`，有文件

  应用场景：master分支太大，可以checkout 指定的分支

+ git clone --bare 远程仓库地址    #只下载仓库信息

### Pull详解

+ git pull

  git fetch + git merge

+ git fetch    #将远程的内容拉取下来，但并不跟本地内容合并

+ git merge  #将拉取下来的内容跟本地合并

#### git push 发生冲突

```shell
$ git push (master)
To https://github.com/Dongdonghe1981/design-pattern.git
 ! [rejected]        master -> master (fetch first)

$ git pull (master|MERGING)

手动merge

$ git commit -am 'change local'

$ git push (master)

#有的公司可能设置为不同的git地址
$ git remote -v
origin  https://github.com/Dongdonghe1981/design-pattern.git (fetch)
origin  https://github.com/Dongdonghe1981/design-pattern.git (push)
```

### 删除远程分支，仓库迁移

+ git push origin --delete 分支名
+ git remote set-url origin 远程仓库的地址

```shell
#列出所有分支和远程追踪分支
git branch -a
* master  #本地分支
  remotes/origin/HEAD -> origin/master 
  remotes/origin/master #远程追踪分支
  remotes/origin/new1 #远程追踪分支
  remotes/origin/new2 #远程追踪分支

#本地不可以切换到远程分支
$ git checkout origin/new1
Note: switching to 'origin/new1'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 47c06ed change local

#直接写远程的分支名，就自动切换到远程分支
$ git checkout new1
Switched to a new branch 'new1'
Branch 'new1' set up to track remote branch 'new1' from 'origin'.

HP@LAPTOP-3JV4RUIN MINGW64 ~/Desktop/git_project/design-pattern2 (new1)
$ git branch -a
  master
* new1
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/new1
  remotes/origin/new2
#删除本地分支，先切换到其他分支
$ git branch -d new1
Deleted branch new1 (was 47c06ed).

#删除远程分支和远程追踪分支
git push origin --delete new1
```

仓库迁移，设置origin库的url为新的仓库的url

```shell
$ git remote -v
origin  https://github.com/Dongdonghe1981/design-pattern.git (fetch)
origin  https://github.com/Dongdonghe1981/design-pattern.git (push)

$ git remote set-url origin https://github.com/Dongdonghe1981/design-pattern2.git

$ git remote -v
origin  https://github.com/Dongdonghe1981/design-pattern2.git (fetch)
origin  https://github.com/Dongdonghe1981/design-pattern2.git (push)

#将本地仓库的内容提交到新的远程仓库
$ git push --all
```

### GitHub自动部署

1. 修改的代码提交到github仓库
2. github打包，上传到阿里云服务器
3. 阿里云服务器更新
4. 查看项目地址，发生变化

### rebase(变基)  -- 使git记录简洁

+ 将多个提交记录整合成一个记录

  注意：如果已经提交到远程仓库，最好不要进行rebase

  ```shell
  git rebase -i HEAD~3   #最新的3次记录进行合并
  vi 合并的版本前用s
  vi 标注合并后的注释
  ```

+ 将dev分支rebase到master分支

  ```shell
  $ git log --oneline --graph --all
  * 1ad2113 (master) master 1
  *   b284e77 Merge branch 'dev'
  |\
  * | 2119469 master txt
  | | * ebab521 (HEAD -> dev) dev branch commit 1
  | |/
  | * 4974de3 dev brach
  |/
  * e0a90f7 v1 & v2 & v3
  
  $ git checkout dev
  $ git rebase master
  Successfully rebased and updated refs/heads/dev.
  $ git checkout master
  $ git log --oneline --graph --all
  * f0c2911 (dev) dev branch commit 1
  * 1ad2113 (HEAD -> master) master 1
  *   b284e77 Merge branch 'dev'
  |\
  | * 4974de3 dev brach
  * | 2119469 master txt
  |/
  * e0a90f7 v1 & v2 & v3
  
  $ git merge dev
  $ git log --oneline --graph --all
  * f0c2911 (HEAD -> master, dev) dev branch commit 1
  * 1ad2113 master 1
  *   b284e77 Merge branch 'dev'
  |\
  | * 4974de3 dev brach
  * | 2119469 master txt
  |/
  * e0a90f7 v1 & v2 & v3
  
  ```

+ 本地commit，远程代码不一致时

  ```shell
  git fetch origin dev
  git rebase origin/dev
  ```

+ gitrebase有冲突的时候，解决冲突，再执行`git add .`，`git rebase --continue`，千万不要`commit`

