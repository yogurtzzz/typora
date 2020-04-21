从远程仓库检出一个新的分支

```shell
git checkout -b sprint-101 origin/sprint-101
```



在远端新建仓库后

在本地进入到工程目录，先将本地目录交由git管理

```shell
git init
git add .
git commit -m "init"
```

然后，将本地仓库和远程仓库建立关联

```shell
git remote add origin  git@github.com:yogurtzzz/xxx.git
```

最后，进行第一次push

```shell
git push -u origin master
```



若不行的话， 再尝试将本地仓库和远程仓库做一下关联

```shell
git branch --set-upstream-to=origin/master
```

若远程仓库已经被删掉了，则原本follow远程仓库的本地仓库则没有了upstream，所以需要删除远程仓库

删除远程仓库

```shell
git remote remove origin
```



查看远程仓库的地址

```shell
git remote show origin
```





将多个提交记录合并成一个

比如有3个提交记录

commitId  msg

abcd1      "add 1 line"

abcd2      "add 2 line"

abcd3     "add 3 line"

比如要保留adcd1，而把abcd2和abcd3合并成一个commit，可以这样做

```shell
git rebase -i abcd1
```

进入vi编辑器

```shell
pick abcd2
squash abcd3
```

保存退出，修改多个commit msg为一个

完毕