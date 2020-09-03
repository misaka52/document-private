# Git

### 1、简介

git是一个分布式版本控制系统。

> 集中式 vs 分布式
>
> cvs和svn都是集中式版本控制系统，版本库是集中存放在中央服务器的，使用的时候从中央服务器取的最新版本开发然后提交到中央服务器。
>
> ![central-repo](https://www.liaoxuefeng.com/files/attachments/918921540355872/0)
>
> 
>
> 分布式版本控制系统，
>
> ![distributed-repo](https://www.liaoxuefeng.com/files/attachments/918921562236160/0)
>
> 

> git init  //初始化一个git仓库，会生成一个.git文件，作为git的版本库



### 2、本地git仓库

#### 工作区、暂存区和版本库

- 工作区：本地文件区域

- 暂存区：stage或index，一般存放在 ".git/index"下，也叫暂存区

- 版本库：管理git仓库文件的区域

  ![img](https://www.runoob.com/wp-content/uploads/2015/02/1352126739_7909.jpg)

  HEAD指向的是当前分支所在的节点（commit）
  
  ![Git 下文件生命周期图。](https://git-scm.com/book/en/v2/images/lifecycle.png)
  
  

#### git add

跟踪文件，将文件从工作区添加至暂存区

git add . 		// 将当前文件夹下（已跟踪）的文件添加至暂存区

git add -A .		//将当前文件夹下所有文件添加至暂存区

#### git checkout

将暂存区的文件恢复至工作区时状况

#### git commit

将暂存区中的文件添加至仓库里，每次commit产生一个节点，生成唯一commitId（短commitId为6位）

git commit 			将暂存区的文件提交至仓库中

git commit -a 		将跟踪过的文件的变化提交至仓库中

git commit --amend		修改上一次的提交信息

#### reset/revert(回滚)

1. git reset

   回滚至指定节点

   --soft			仅重置git仓库内容。修改已add，待提交。

   --mixed		（默认选项）重置git仓库内容、暂存区。工作区已修改未提交到暂存区

   --hard		  重置git仓库内容、工作区。修改代码丢失

   当回滚至之前某节点时，若想要提交至远程仓库，需git push -f 强制推送

2. git revert

   回滚指定节点代码，用一个新增节点来表示回滚操作

#### git merge

​	git merge \<branch>		将指定分支代码合并至当前分支

#### git rm

git rm file		git版本库删除文件

git rm --cached file	将文件从版本库移到工作区，不再跟踪

#### git stash

将修改临时保存在贮藏区，贮藏区类似于栈（一个项目git仓库一个），先进后出

git stath 		暂存未提交区域

git stash apply [\<stash>] 		恢复指定stash(不填\<stash>默认选择栈顶元素，stash@[0])

git stath drop [\<stash>]		删除指定stash（不填\<stash>默认选择栈顶元素，stash@[0])

git stash pop [\<stash>]		恢复并删除指定stash（不填\<stash>默认选择栈顶元素，stash@[0])

```shell
# 查看stash存储区内容
git stash list
stash@{0}: WIP on test2: f1ae341 2-1
stash@{1}: WIP on test: 6c4d9fb Merge branch 'test2' into test
stash@{2}: WIP on test: 6c4d9fb Merge branch 'test2' into test
```





### 3、 远程仓库

#### git clone

复制项目到本地

- http方式：获取项目http链接， git clone \<url>，需要输入账号密码

- ssh方式

  - 创建ssh key，查看本地家目录是否存在.ssh目录，目录下是否有id_rsa和id_rsa.pub 私钥公钥两个文件，没有可以生成

    > ```
    > $ ssh-keygen -t rsa -C "youremail@example.com"
    > ```

  - 将公钥id_rsa.pub的信息添加至git上ssh key中

  - git clone \<SSH URL>

#### git remote

远程数据源，可配置多个

git remote -v 		查询所有数据源信息

git remote add \<name> \<url> 		增加新数据源

git remote set-url \<name> \<newurl>		修改数据源地址

#### pull & fetch & push

![img](http://kmknkk.oss-cn-beijing.aliyuncs.com/image/git.jpg)

**git fetch**   

​	将远程仓库代码拉到本地仓库

**git pull**

​	git pull = git fetch + git merge，产生冲突时需要手动解决

**git push**

​	将本地仓库代码推送到远程仓库

​	git push \<remote> \<本地分支> : \<远程仓库分支>	若远程仓库分支不存在该分支，则创建新分支；若\<本地分支>不填，相当于推送空分支至远程空库，强制删除分支

​	git push -u \<remote> \<branch> 		push并设置默认remote，常用于push项目到远程空项目上



### 4、分支，合并，冲突，变基

**branch**

分支也类似于一个指针

git branch		查看分支

git branch \<name>		创建分支（和当前分支一致）

git checkout -b \<name>		创建+切换分支

git branch -d \<branch>		删除分支（无法删除未merge保存的分支）

git branch -D \<branch>		强制删除分支

**合并**

两分支合并，可能产生冲突，例如将dev分支合并到master分支

​	1、若master分支只是单纯的落后于dev若干节点，自动合并，不产生新节点

> fast-forward模式，当一个分支落后于一个分支合并时，落后分支直接指向最新分支，不产生commit记录
>
> merge时默认fast-forward模式，可使用--no-ff 关闭此模式。关闭后merge就是落后节点式的合并也会产生commit记录

​	2、若两分支修改过同一个文件，但修改代码的行数不一致，产生冲突，能自动合并，并生成一个commit节点表示merge

​	3、若两分支修改过同一个文件同一行数，产生冲突，无法自动合并

​		1）产生修改相同行数的文件，若没有手动解决。无法自动合并（冲突文件为Unmerged状态，需要add），下面是修改同一地方冲突情况

```
<<<<<<< HEAD
【master分支修改内容】
=======
【dev分支修改内容】
>>>>>>> dev
```

> git merge --abort 		取消冲突合并

​		2）其他能自动解决冲突的文件一直暂存区

ps：idea手动解决冲突，对于修改相同行数的文件，先用魔法棒解决简单冲突，再手动解决无法自动解决的冲突

  <img src="D:/yuanshancheng3/Documents/JD/office_dongdong/yuanshancheng/Image/1577435159400_src" alt="img" style="zoom: 200%;" /> 

**变基**

参考：[https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA](https://git-scm.com/book/zh/v2/Git-分支-变基)

修改提交历史，将分叉的节点（commit）整合到一条直线上，使其更加清晰直观

git rebase实现

注：网上对变基有各种争议

变基：优点：使提交历史更清晰直观；缺：修改历史，失真，本身是种“亵渎”，并且可能出现问题（修改提交历史，提交到远程仓库，影响其他人的开发）

只对尚未推送或已经分享给别人的本地修改执行变基操作



### 5、其他

#### tag

标签，快照，不可修改，标签按照字母排序

git tag \<tag> [\<commitId>]		新建标签

#### git config

```
git config --global color.ui true  		开启颜色提示，提高工作效率
```

#### .gitignore

git忽略指定文件

- 所有空行或#开头的行都会被git忽略
- ! 开头的文件不能忽略
- 可以使用标准的 glob 模式匹配。

> glob模式：	
>
> *匹配零个或多个任意字符
>
> ？匹配任意一个字符
>
> [abc] 匹配abc任意一个字符
>
> [0-9] 匹配所有0到9的数字
>
> 使用两个*号表示匹配中间任意目录







参考：

1. https://www.liaoxuefeng.com/wiki/896043488029600 
2. https://git-scm.com/book/zh/v2 （官网资料）
3. https://www.runoob.com/git/git-workspace-index-repo.html
4. https://www.cnblogs.com/runnerjack/p/9342362.html