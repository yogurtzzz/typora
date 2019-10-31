在idea中使用git，无法进行pull和push，一直报错说Permission denied(public key)

然而在Git Bash中是可以正常操作的

百度谷歌了N多方法依然不奏效



最后，在知乎看了一个答案..终于解决

* 首先，清除所有的key-pair

  `ssh-add -D`

  `rm -r ~/.ssh`

  并删除存放key的目录

* 重新生成密钥对

  `ssh-keygen -t rsa -C "1009784814@qq.com"`

* 在github上重新添加该ssh

* 测试

  `ssh -T git@github.com`



之前失败原因分析：

之前生成ssh key的时候，对私钥指定了密码

之后重新生成ssh key的时候，没有对私钥指定密码

