# Git简单使用

Git可以在Linux，Mac OS，Windows下安装使用

**注意**：版本控制系统只能跟踪**文本文件**的改动，如`txt`文件，代码文件，版本控制系统可以告知每次的改动，如再第2行添加了一个单词，等等。但是如**图片**，**视频**这些二进制文件，虽也能由版本控制系统管理，但版本控制系统只知道文件从100KB变成了120KB，而无从得知具体做了哪些改动。



以在Windows下为例子，点击`Git Bash`，在命令行窗口下进行操作

使用之前，需要先设置一下`user`和`email`

`$ git config --global user.name "yogurtzzz"`

`$ git config --global user.email "1009784814@qq.com"`

这是因为Git是分布式的版本控制系统，每台机器都需要自报家门。`--global`参数表示这台机器上所有的Git仓库，都会使用这个配置，不过也可以对某个仓库指定不同的用户名和email



* 创建一个版本库

选择一个合适的地方，创建一个目录

`$ mkdir yogurtGitRepo`

进入该目录

`$ cd yogurtGitRepo`

`$ pwd`

`/e/yogurtGitRepo`

通过`git init`将这个目录交由git管理

`$ git init`

`Initialized empty Git Repository in /e/yogurtGitRepo/.git/`

* 编写一个文件，放到`yogurtGitRepo`目录下

```
/*readme.txt*/
My name is yogurt
```

* 将文件提交

  * `git add readme.txt`
  * `git commit -m "Add a self-introduction file."`
  * 注：可以多次使用`git add`，添加不同的文件，然后`git commit`一次性提交这些文件

* 对文件做了一些修改后

  ```cmd
  $ git status
  On branch master
  Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git checkout -- <file>..." to discard changes in working directory)
  
          modified:   readme.txt
  
  no changes added to commit (use "git add" and/or "git commit -a")
  ```

  

  ```cmd
  $ git diff readme.txt
  diff --git a/readme.txt b/readme.txt
  index 23de677..36a5917 100644
  --- a/readme.txt
  +++ b/readme.txt
  @@ -1,4 +1,5 @@
   My Name is Yogurt.I'm a primary programmer
   I add a new line to this file;
   my age is 23
  -my girlfriend is lijinxia
  \ No newline at end of file
  +my girlfriend is lijinxia
  +haha
  \ No newline at end of file
  
  ```

  `git add`

  `git commit -m "something" `

* `git log`查看提交的记录

  ```cmd
  $ git log
  commit 79506857cddc4a6876b4c0187d21f17fffb4b466 (HEAD -> master)
  Author: yogurtzzz <1009784814@qq.com>
  Date:   Sun Jun 9 10:39:22 2019 +0800
  
      add girlfriend info
  
  commit 1d5da7fdd347852afc4f05d171a899271e3ea598
  Author: yogurtzzz <1009784814@qq.com>
  Date:   Fri Jun 7 17:20:03 2019 +0800
  
      add my age info
  
  commit 48a110a88fcc989d437174fdbe88cdeaf6a1d83e
  Author: yogurtzzz <1009784814@qq.com>
  Date:   Fri Jun 7 16:30:11 2019 +0800
  
      Add to a new line.wow
  
  commit ea380e1ccd8ed6aa12147de703a3777fcd3f43b0
  Author: yogurtzzz <1009784814@qq.com>
  Date:   Fri Jun 7 16:24:32 2019 +0800
  
      wrote a readme file
  
  ```

* `git log --pretty=oneline`

  ```cmd
  $ git log --pretty=oneline
  79506857cddc4a6876b4c0187d21f17fffb4b466 (HEAD -> master) add girlfriend info
  1d5da7fdd347852afc4f05d171a899271e3ea598 add my age info
  48a110a88fcc989d437174fdbe88cdeaf6a1d83e Add to a new line.wow
  ea380e1ccd8ed6aa12147de703a3777fcd3f43b0 wrote a readme file
  ```

* `git reset` 版本回退

  ```cmd
  $ git reset --hard HEAD^
  HEAD is now at 1d5da7f add my age info
  ```

  回到未来的某个版本（只要记得版本号就ok）

  ```cmd
  $ git log --pretty=oneline
  1d5da7fdd347852afc4f05d171a899271e3ea598 (HEAD -> master) add my age info
  48a110a88fcc989d437174fdbe88cdeaf6a1d83e Add to a new line.wow
  ea380e1ccd8ed6aa12147de703a3777fcd3f43b0 wrote a readme file
  
  Administrator@YogurtPC MINGW64 /e/myGitRepository (master)
  $ git reset --hard 7950685
  HEAD is now at 7950685 add girlfriend info
  ```

* `git reflog` 记录每一次命令



小结

* `HEAD`指向的版本为当前版本，Git允许我们在版本的历史之间穿梭，使用`git reset --hard commit_id`
* 穿梭前，使用`git log`可以查看提交的历史，以便确认回退到哪个版本
* 重返未来，使用`git reflog`查看命令历史，以便确认回到未来的哪个版本



## Git内部结构

Git拥有一个暂存区，这是与其他版本控制系统（如SVN）的不同之处

* 工作区（Working Directory）

  即在电脑里看到的目录，比如`yogurtGitRepo`文件夹就是一个工作区

* 版本库（Repository）

  `yogurtGitRepo`里有一个隐藏的目录`.git`，这是Git的版本库，版本库里存了很多东西，如**暂存区**（stage 或 index），还有Git自动创建的第一个分支`master`，以及指向`master`的一个`HEAD`指针

  ![Git](https://www.liaoxuefeng.com/files/attachments/919020037470528/0)

往Git版本库里添加文件时，是分两步的：

* `git add` ：实际是将文件修改添加到**暂存区**（stage）
* `git commit`：把**暂存区**的所有内容提交到当前分支（现在是`master`分支）

例子：

* `git add readme.txt`
  `git add LICENSE`

  现在Git内部状态如下

  ![](https://www.liaoxuefeng.com/files/attachments/919020074026336/0)

* `git commit -m "demo"`

  现在Git内部状态如下

  ![](https://www.liaoxuefeng.com/files/attachments/919020100829536/0)

提交完成后，若没有对工作区做任何修改，那么工作区就是干净的

```cmd
Administrator@YogurtPC MINGW64 /e/myGitRepository (master)
$ git status
On branch master
nothing to commit, working tree clean
```



## 管理修改

Git跟踪并管理的是修改，而不是文件

例子：对`readme.txt`添加一个新行

```txt
haha
this is new line
```

然后`git add readme.txt`

接着，再对`readme.txt`进行修改

```txt
haha
this is new line ,can you see me ?
```

接着，`git commit -m "test"`

梳理一下操作流程

`第一次修改` -> `git add` -> `第二次修改` -> `git commit`

以为git管理的是修改，故只有第一次修改被`git add`到了**暂存区**，最后一步`git commit`将暂存区的内容提交到版本库

## 撤销修改

* 撤销工作区的修改

`git checkout -- readme.txt` （撤销工作区的修改）

若工作区做了修改，还没`git add`（还未放到暂存区），此时用`git checkout`，就能丢弃工作区的修改，使工作区的文件和版本库里的最新文件保持一致。

若工作区做了修改，并`git add`（提交到了暂存区），此时又做了修改，再用`git checkout`，就能丢弃工作区的修改，使工作区的文件和暂存区的保持一致。

总之，就是让工作区的文件回到最近一次`git commit`或`git add`时的状态



* 撤销暂存区的修改

那如何撤销掉暂存区的修改呢？

`git reset HEAD readme.txt` 

**这句将暂存区的修改回退到工作区**，此时再用`git status`查看，会看到`changes not staged for commit`，即修改已经回退到工作区，暂存区中已经清空，此时再用`git checkout -- readme.txt`即可



**小结**：

* 当你改乱了工作区的某个文件，想直接丢弃工作区的修改时，用`git checkout -- file`
* 当你不但改乱了工作区的某个文件，还添加到了暂存区，想丢弃修改，分两步，1. 用命令`git reset HEAD file`，就将修改回退到了工作区，再`git checkout -- file`

* 当你已经提交了不合适的修改到版本库，想要撤回本次修改，可用`git reset HEAD^1 file`（回退到上一次版本），前提是还没推送到远程库

## 删除文件

`git rm file`

删除文件并提交到暂存区

再`git commit -m "del something"` 将删除文件这一操作提交



## 远程仓库

打开Git-Bash，创建SSH-KEY

```cmd
$ ssh-keygen -t rsa -C "1009784814@qq.com"
```

会生成2个文件，`example`和`example.pub`

注册一个github账号，在个人设置中添加SSH KEY，将`example.pub`里的内容复制进去，即可





Git中文显示乱码解决：

在bash下输入:  `git config --global core.quotepath false`

[参考链接](https://blog.csdn.net/tyro_java/article/details/53439537)