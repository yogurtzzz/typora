`git clean -d -f ` 丢弃所有untracked的文件 （-d 清除untrack的目录 -f清除untrack的文件）

`git checkout . ` 丢弃工作区中对已有文件的修改



git bash 中文若显示类似 `\347\224\265\345`这样的中文乱码，可以使用

`git config --global core.quotepath false`



本地新建一个仓库后，和远端仓库关联起来:

先添加远程仓库

```shell
git reomte add origin git@github.com:yogurtzzz/spring_in_action_5th_demo.git
```

后push到远程仓库

```shell
git push -u origin master
```



将远程仓库的 origin/master 分支和本地的master分支关联起来

```shell
git branch --set-upstream-to=origin/master master
```

