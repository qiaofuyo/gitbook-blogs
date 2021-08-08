# git使用说明

* git:分布式版本控制系统
* svn:集中式版本控制系统

## 安装

* 方法一：git官网：git-scm.com
* 方法二：msysgit：一个项目
* 方法三：[可视化Git Bash下载](https://gitforwindows.org/)

## git本地仓库操作（使用方法一安装）

1. 安装后需设置一些基本信息

    a、设置用户名：git config --global user.name "姓名"

    b、设置用户邮箱：git config --global user.email "联系邮箱"

   ```text
    注意：该配置会在github主页上显示谁提交了该文件
   ```

    c、配置ok之后，我们用如下命令来看看 **是否配置成功**

   ```text
    git config --global --list
    *注意*：git  config --global 参数，有了这个参数表示你这台机器上所有的git仓库都会使用这个配置，当然你也可以对某个仓库指定不同的用户名和邮箱
   ```

2. 初始化一个新的git本地仓库 a、创建文件夹\(此文件夹是项目文件夹\)
   * 方法一：可以鼠标右击-》点击新建文件夹test1

     * 方法二：使用cmd命令新建：$ mkdir test1

     b、在文件内初始化git（创建git本地仓库）

     * 方法一：点击test1文件下进去之后 -&gt; 鼠标右击选择Git Bash Here -&gt; 输入 $ git init
     * 方法二：直接输入 $ cd test1 -&gt; 输入 $ git init

     > 克隆仓库： 创建一个本地仓库的克隆版本： $ git clone /path/to/repository 如果是远端服务器上的仓库，命令会是这个样子： $ git clone 仓库地址
3. 向本地仓库中添加文件
   * 通过$ git add '文件名'（git add \*：提交所有文件； git add .：提交修改过的文件） 把指定文件添加到暂存区
   * 通过$ git commit -m '代码提交信息' 将文件从暂存区提交到本地仓库
4. 修改仓库文件\(项目代码\)
   * 方法一：用编辑器打开index.html进行修改-&gt;$ git status 查询是否修改的状态-&gt;添加到暂存区-&gt;提交操作
   * 方法二：使用git命令。$ vi '文件名'，然后在中间写内容，最后提交操作
5. 删除仓库文件
   * 方法一：在编辑器中直接把要删除的文件删除掉
   * 方法二：使用git删除：$ git rm '文件名'，然后提交操作

## git远端仓库操作

1. 在 github.com 上创建项目（远端仓库）
2. 克隆远端仓库的项目

    $ git clone 仓库地址

3. 本地仓库代码提交到远端仓库
   * 方法一：$ git push origin master
   * 方法二：如果没有克隆现有仓库，并欲将连接远程服务器$ git remote add origin &lt;仓库地址&gt; =&gt;如此便能推送$ git push origin master,可以把 master 换成你想要推送的任何分支。

## 远端仓库的分支

分支是用来将特性开发绝缘开来的。在创建仓库的时候，master 是“默认的”。在其他分支上进行开发，完成后再将它们合并到主分支上。 1. 创建一个叫做“feature\_x”的本地分支，并切换过去 $ git checkout -b feature\_x

1. 切换回本地主分支 $ git checkout master
2. 删掉新建的本地分支 $ git branch -d feature\_x
3. 删除远程仓库的分支 $ git branch -r 查看远程分支 $ git branch -r -d origin/远程分支名 \#删除本地的**远程跟踪分支** ，即本地不在连接到远程仓库的该分支，但该分支还在远程仓库上存在着 $ git push origin :远程分支名 \#删除远程分支
4. 推送分支 除非你将本地分支推送到远端仓库，不然该分支就只存在本地仓库： $ git push origin 分支名

## 更新与合并

1. 更新本地仓库

   ```text
    $ git pulll                        #拉取所有的分支
    $ git pulll < 分支名 >                #拉取单独的分支
   ```

   ```text
    $ git fetch < 本地仓库名 >    < 远程分支名 >        #拉取单独的分支，此时尚未接收远程仓库的内容
    $ git checkout -b < 新建本地分支名 >  < origin/远程分支名 >        #上一步拉取到的分支内容同步进本地分支里
   ```

2. 合并其他分支到当前分支（例如 master） $ git merge 分支名 自动合并并非次次都能成功，有可能导致冲突，这时候就需要修改这些文件来手动合并这些冲突了，改完之后，需要执行如下命令以将它们标记为合并成功 $ git add '文件名' 在合并改动之前，也可以使用如下命令查看： $ git diff  

## 删除仓库

1. Linux

```text
$ ls -a        #找到目录下隐藏的文件
$ rm -rf .git
```

1. Windows

> 目录下直接删

## 常用命令

#### git clone        克隆

```text
> git clone <版本库的网址> 
> git clone <版本库的网址> <本地目录名> // 指定本地目录名
> git clone -o romoteBranchName <版本库的网址> //指定远程分支名称
```

#### git remote        远程

```text
> git remote //命令列出所有远程主机
> git remote -v //列出所有远程主机并展示远程主机的网址
> git remote show <主机名> //查看远程分支的详细状况
> git remote add <主机名> <网址> //添加远程主机名
> git remote rm <主机名> // 删除远程主机
> git remote rename <原主机名> <新主机名> // 修改远程主机名
```

#### git branch        分支

```text
> git branch //查看本地分支 现在所在的分支会有 * 号标注
> git branch -r //查看远程分支
> git branch -a //查看所有分支（本地+远程）
```

#### git checkout        切换分支/拉取分支/新建分支/丢弃修改

```text
> git checkout 分支名 //切换到指定分支
// 切换时加origin 相当于 把远程分支拉取到本地仓库
> git checkout origin/远程分支名
// 指定本地分支切出新分支并切换。不指定分支时根据当前分支切新分支
> git checkout -b newBrach 远程分支/本地已有分支 
// 丢弃未推送到本地仓库和暂存缓冲区的内容
> git checkout 丢弃的文件名（.）
```

#### git merge        合并

```text
> git merge 本地目标分支名 //合并本地分支
> //合并本地分支，用于取 fetch 后的内容
> git merge 远程分支/本地分支
```

#### git pull        拉取

```text
> //把指定远程主机名远程分支的内容拉取到指定的本地分支
> git pull <远程主机名> <远程分支名>:<本地分支名>
> //把指定分支内容拉取到当前本地分支，相当于先 fetch 再 merge
> git pull <远程主机名> <远程分支名>
```

#### git push        推送

```text
> //把指定本地分支的 commit 推到指定的远程主机远程分支上
> git push <远程主机名> <本地分支名>:<远程分支名>
> //把本地分支推送与之存在"追踪关系"的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。
> git push <远程主机名> <远程分支名> 
> //删除指定的远程分支，等同于推送一个空的本地分支到远程分支
> git push <远程主机名>:<远程分支名>
> //指定默认主机，下次直接 git push 即可
> git push -u <远程主机名> <本地分支名>
```

#### git log        查看日志

```text
> //把提交记录打印出来，会显示出commit message
> git log
```

## 分支命名规范

git 分支分为集成分支、功能分支和修复分支，分别命名为 develop、feature 和 hotfix，均为单数。不可使用 features、future、hotfixes、hotfixs 等错误名称。

* master（主分支，永远是可用的稳定版本，不能直接在该分支上开发）
* develop（开发主分支，所有新功能以这个分支来创建自己的开发分支，该分支只做只合并操作，不能直接在该分支上开发）
* feature-xxx（功能开发分支，在develop上创建分支，以自己开发功能模块命名，功能测试正常后合并到develop分支）
* feature-xxx-fix\(功能bug修复分支，feature分支合并之后发现bug，在develop上创建分支修复，之后合并回develop分支。PS:feature分支在申请合并之后，未合并之前还是可以提交代码的，所以feature在合并之前还可以在原分支上继续修复bug\)
* hotfix-xxx（紧急bug修改分支，在master分支上创建，修复完成后合并到 master）

注意事项：

* 一个分支尽量开发一个功能模块，不要多个功能模块在一个分支上开发。
* feature 分支在申请合并之前，最好是先 pull 一下 develop 主分支下来，看一下有没有冲突，如果有就先解决冲突后再申请合并。

## 提交规范（commit message）

> [https://zhuanlan.zhihu.com/p/100773495](https://zhuanlan.zhihu.com/p/100773495)
>
> 使用流行范围广的Angular规范

```text
<1*类型>[2可选的作用域]: <3*描述>

[4可选的正文]

[5可选的脚注]
```

```text
<*type>(<scope>): <*subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

### 1. type

type为必填项，用于指定commit的类型，约定了feat、fix两个主要type，以及docs、style、build、refactor、revert五个特殊type，其余type暂不使用。

```text
# 主要type
feat:     增加新功能
fix:      修复bug

# 特殊type
docs:     只改动了文档相关的内容
style:    不影响代码含义的改动，例如去掉空格、改变缩进、增删分号
build:    构造工具的或者外部依赖的改动，例如webpack，npm
refactor: 代码重构时使用
revert:   执行git revert打印的message

# 暂不使用type
test:     添加测试或者修改现有测试
perf:     提高性能的改动
ci:       与CI（持续集成服务）有关的改动
chore:    不修改src或者test的其余修改，例如构建过程或辅助工具的变动
```

当一次改动包括主要type与特殊type时，统一采用主要type。

### 2. scope

scope也为必填项，用于描述改动的范围，格式为项目名/模块名，例如：node-pc/common rrd-h5/activity，而we-sdk不需指定模块名。如果一次commit修改多个模块，建议拆分成多次commit，以便更好追踪和维护。

### 3. body

body填写详细描述，主要描述改动之前的情况及修改动机，对于小的修改不作要求，但是重大需求、更新等必须添加body来作说明。

### 4. break changes

break changes指明是否产生了破坏性修改，涉及break changes的改动必须指明该项，类似版本升级、接口参数减少、接口删除、迁移等。

### 5. affect issues

affect issues指明是否影响了某个问题。例如我们使用jira时，我们在commit message中可以填写其影响的JIRA\_ID，若要开启该功能需要先打通jira与gitlab。参考文档：[http://docs.gitlab.com/ee/user/pro](https://link.zhihu.com/?target=http%3A//docs.gitlab.com/ee/user/pro)…

填写方式例如：

```text
re #JIRA_ID
fix #JIRA_ID
```

