### git是个啥子？怎么用？

![](C:\Users\Administrator\Desktop\笔记\images\src=http___img2018.cnblogs.com_i-beta_1458513_202002_1458513-20200205133615849-552140910.png&refer=http___img2018.cnblogs.jpg)

秋总今天用word写了公司舆情v1.2.3版本的需求，然后跟我们开了个会讨论需求，经过一番激烈的辩论，我们现在有个功能出现了两个实现方案，于是我们提了一个很无理（很棒）的要求（让秋总先回去把两种方案都做出一个原型），于是聪明的秋总立刻想到了复制一份word备份，然后在这个word副本进行新的修改，保留原本的版本以防万一。谨慎（鸡贼）的秋总每改动一部分内容，就复制出了一个副本进行存档，于是现在秋总的桌面上有如下文件：

公司舆情需求文档(v1.2.3).doc

公司舆情需求文档(v1.2.3)-副本.doc

公司舆情需求文档(v1.2.3)-副本(2).doc

公司舆情需求文档(v1.2.3)-副本(3).doc

公司舆情需求文档(v1.2.3)-副本(4).doc

......

公司舆情需求文档(v1.2.3)-副本(n).doc

更要命的是，在不知道修改到哪版的时候，懒惰的秋总求助了另一位产品经理卷王秋让她帮忙找出自己这个方案设计不合理的地方并进行修改，卷王秋在改完后发给了秋总，这个时候秋总已经改到了公司舆情需求文档(v1.2.3)-副本(n+).doc，秋总已经忘记了自己是在改到第几版的时候让卷王秋帮忙修改的，想到合并卷王秋的修改却发现自己根本无从下手，于是秋总陷入了沉思......



以上的这些骚操作如果使用git进行管理会是这样的：

秋总在开完会后，使用git在开会前的那份需求文档所在文件夹下初始化了一个git仓库



```shell
cd mydocuments
ls
```

**mydocuments文件夹下(初始化前)**

`公司舆情需求文档(v1.2.3).doc`

```shell
git init
ls
```

**mydocuments文件夹下(初始化后)**

 `公司舆情需求文档(v1.2.3).doc`
`.git`

```shell
# 将.git文件当前所在文件夹下所有文件和内容添加到git本地仓库暂存区
git add .

# 将git本地仓库暂存区的所有文件和内容打上注释标签并提交到git本地仓库
git commit -m"公司舆情需求文档(v1.2.3).doc"
```

然后秋总在初版需求文档的基础上改出了第一版

```shell
git add .

git commit -m"公司舆情需求文档(v1.2.3)-副本"
```

改出了第二版，第三版，第四版......

```shell
# 第二版
git add .

git commit -m"公司舆情需求文档(v1.2.3)-副本(2)"

# 第三版
git add .

git commit -m"公司舆情需求文档(v1.2.3)-副本(3)"

# 第四版
git add .

git commit -m"公司舆情需求文档(v1.2.3)-副本(4)"
......

```

改到第七版的时候，秋总向卷王秋发起了援助请求，那么这时需要一个共享的远程仓库来供两人共同协作修改

```shell
# 第七版
git add .

git commit -m"公司舆情需求文档(v1.2.3)-副本(7)"
```

假设我们的远程仓库使用的是github

1. 在github上也创建一个叫作mydocuments的仓库
2. 创建完毕后，将本地的git仓库关联到远程仓库（通过一个url）
3. 将本地仓库推送到远程仓库（默认分支为master）

```shell
git remote add origin https://github.com/zhangqiuqi/mydocuments.git

# 推送到远程仓库，执行后需要输入github的账号和密码（使用ssh协议则不需要）
git push origin master
```

现在，在github远程仓库mydocuments中拥有与本地一模一样的git仓库（包括所有的历史提交版本）



接下来，卷王秋从github上将秋总提交上去的所有东西拉到自己电脑上

```shell
git clone https://github.com/zhangqiuqi/mydocuments.git
```

现在，卷王秋的电脑上有了一个叫作mydocuments的文件夹，文件夹中有内容如下：

`公司舆情需求文档(v1.2.3).doc`

`.git`

这意味着卷王秋拥有了一个与秋总一模一样的本地git仓库（包括秋总修改过的所有版本）

然后，**卷王秋**要在第七版上做修改(未来有可能秋总会在第n版上进行卷王秋修改版本的合并)，修改完后将修改的版本推送到远程仓库的一条新分支上

```shell
# 进入mydocuments文件夹
cd mydocuments

# 新起一条git本地仓库分支
git branch temp

# 将.git文件当前所在文件夹下所有文件和内容添加到git本地仓库暂存区
git add .

# 将git本地仓库暂存区的所有文件和内容打上注释标签并提交到git本地仓库
git commit -m"卷王秋的修改版本"

# 进行远程仓库关联
git remote add origin https://github.com/zhangqiuqi/mydocuments.git

# 推送到远程仓库的temp分支上（当远程仓库不存在temp分支时自动创建）
git push origin temp
```

此时，卷王秋通知秋总已经修改完毕并推送到了远程仓库的temp分支上，于是秋总需要将卷王秋的分支内容拉下来并合并到自己现在的第n个修改版本上

```shell
# 当前位于git仓库master分支的版本n上
git pull origin temp
```

上面的git pull命令等同于

```shell
# 拉取远程仓库的temp分支
git fetch origin temp

# 将temp分支的内容合并到当前分支中
git merge temp
```

到这里，秋总已经完成了她的所有修改，现在秋总的本地git仓库中存放了n个版本的`公司舆情需求文档(v1.2.3).doc`文件，当前的第n个版本合并了卷王秋在第七版基础上进行修改的内容



当然了，秋总这时想回到第九个版本，然后将卷王秋在第七版基础上修改的内容合并到第九版中，这也是可以的

```shell
# 找到所有的历史提交版本信息，从提交信息中找到打了`公司舆情需求文档(v1.2.3)-副本(9)`注释标签的相应版本id
git log

# 工作区从第n版回滚到第九版 
git reset --hard `${commitId}`

# 将temp分支的内容合并到当前版本中
git merge temp
```

到这里，秋总实现了她的版本控制骚操作。



**所以，git是个分布式版本控制系统（详情请百度）**







