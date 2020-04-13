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



删除远程仓库

```shell
git remote remove origin
```



查看远程仓库的地址

```shell
git remote show origin
```

