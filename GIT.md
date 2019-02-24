commit 之后版本回退

````java
git reset --hard head~1
git reset --hard commit_id
````



生成秘钥

`ssh-keygen -t rsa -C "youremail@example.com"`

1、先有本地库，再有远程库 

本地仓库与远程仓库连接

`git remote add origin git@github.com:michaelliao/learngit.git`

关联后，使用命令`git push -u origin master`第一次推送master分支的所有内容；

此后，每次本地提交后，只要有必要，就可以使用命令`git push origin master`推送最新修改；



2、最好的方式是先创建远程库，然后从远程库克隆

`git clone git@github.com:michaelliao/gitskills.git`

GitHub给出的地址不止一个，还可以用`https://github.com/michaelliao/gitskills.git`这样的地址。实际上，Git支持多种协议，默认的`git://`使用ssh，但也可以使用`https`等其他协议，`ssh`速度最快

在IDEA不使用的代理时

1、使用git@git.cnsuning.com:svcmng/svcmng_git.git 提示 invalid user or password

![image-20190218153241084](/Users/houchao/workspace/Note/jpg/image-20190218153241084.png)

2、使用http://git.cnsuning.com/svcmng/svcmng_git 可以正常 clone

在 IDEA 使用代理时，

1、使用git@git.cnsuning.com:svcmng/svcmng_git.git 提示   Could not read from remote repository.

2、使用http://git.cnsuning.com/svcmng/svcmng_git 可以正常 clone

![image-20190218150124093](/Users/houchao/workspace/Note/jpg/image-20190218150124093.png)





如何分别使用 ssh 和 http



SSH Key 是干嘛的



配置和管理多个 ssh key



git 配置

`git config --global user.name ""`

`git config --global user.email ""`





切换分支





![image-20190219225026439](/Users/houchao/workspace/Note/jpg/image-20190219225026439.png)

和远端仓库没有关联好，推不上去