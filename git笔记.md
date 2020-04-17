git学习网址  https://learngitbranching.js.org/?demo

## 本地仓库

### 创建分支，切换分支

git branch bugFix  创建一个新分支bugFix

git checkout bugFix  将当前分支切换到新分支



或者

git checkout -b bugFix 创建新分支bugFix并将当前分支切换到新分支



git branch -a 查看所有分支

git branch 查看当前使用分支，分支上带*的就是当前使用分支

git checkout 分支名  切换分支

### 合并分支

git merge

如当前分支是master

使用git merge bugFix ，将使得bugFix分支合并到master分支

**git merge期望1个参数**

git merge A  就是将A分支合并到当前分支

**而 git rebase 期望最多2个参数**

git rebase B    将当前分支，合并到B分支

git rebase B A    将A分支，合并到B分支

git rebase

若当前分支是bugFix

`git rebase master` ，将使得bugFix分支合并到master

当当前分支不是bugFix时，使用

`git rebase master bugFix` 将bugFix合并到master

HEAD

分离HEAD，让HEAD指向具体的提交ijlu，而不是分支名

![1576130362421](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576130362421.png)

git checkout C1

在命令执行之前，HEAD->master->C1

命令执行之后 HEAD->C1   master ->C1



### 相对引用

每个提交记录都会对应一个hash值，可以通过一个hash值的前几位，来定位一个提交记录。

使用相对引用，如

- `^`向上移动一个提交记录
- `~num` 向上移动num个提交记录，不加数字的话，等同于`^`

如： `git checkout master^` 会使得HEAD切换到master的父节点

`git checkout master^^`使得HEAD切换到master的祖父节点

`git checkout master~4`



移动分支，可以使用`-f`来让分支只想另一个提交

`git branch -f master HEAD~3`

会让master分支强制指向HEAD的第3级父提交

![1576130827321](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576130827321.png)

### 撤销变更

git reset

`git reset HEAD^`

![1576131844897](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576131844897.png)

将HEAD往前回退一个提交，但是这样的方法对远程分支是无效的

git revert

`git revert HEAD`

![1576131903079](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576131903079.png)



git cherry-pick

![1576132335553](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576132335553.png)



![1576132350122](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576132350122.png)



交互式 rebase

调整提交顺序，删除不想要的提交等

git rebase -i HEAD~4

以交互式的方式调整最近4次提交记录的顺序。还可以删除某个提交记录etc..





只取一个提交

![1576132957608](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576132957608.png)

![1576132967049](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576132967049.png)



git checkout master

git cherry-pick C4

可以使得只合并C4的提交

![1576133103201](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576133103201.png)





git commit --amend 可以修改上次提交的记录





若要对以前提交的某一次记录进行修改，可以先使用git rebase -i 对提交记录进行重新排序，把我们想修改的提交记录挪到最前面，然后修改，并用git commit --ammend ，再用git rebase -i 重新排序即可





git cherry-pick C1 C2 可以将C1,C2 取过来，追加到HEAD上（前提,C1,C2不能是HEAD的上游）



git 的tag，标签功能，其咎像提交树上的一个锚点，标识了某个特定的位置

`git tag v1.0 C1`

表示对C1提交记录添加一个标签，标签名为v1.0

![1576133928116](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576133928116.png)



git describe  找到离你最近的标签

![1576134083157](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576134083157.png)

![1576134115251](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576134115251.png)



选择父提交记录

![1576135152396](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576135152396.png)

![1576135171683](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576135171683.png)



![1576135187988](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576135187988.png)



![1576135215533](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576135215533.png)



通过

`git branch bugWork HEAD^^2~1`来直接在某个提交上创建一个分支

![1576135358013](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576135358013.png)





## 远程仓库

![1576135978813](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576135978813.png)





git fetch

![1576136294037](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576136294037.png)

![1576136306089](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576136306089.png)



![1576136344582](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576136344582.png)



![1576136357675](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576136357675.png)



git pull



![1576136479771](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576136479771.png)



![1576136849710](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576136849710.png)





git push

![1576137106085](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576137106085.png)





![1576137421167](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576137421167.png)

![1576137462123](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576137462123.png)



![1576137529270](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576137529270.png)



![1576137548097](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576137548097.png)



![1576137800835](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576137800835.png)



![1576137841373](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576137841373.png)



git pull = git fetch + git merge

git pull --rebase = git fetch + git rebase



为什么操作远程分支时不喜欢用merge ?

![1576139020738](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576139020738.png)





远程跟踪分支

![1576139259223](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576139259223.png)

![1576139273886](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576139273886.png)



![1576139327949](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\1576139327949.png)