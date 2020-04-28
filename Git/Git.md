

# 1 本地仓库

## 1.1 本地仓库操作

~~~
1、创建账号和邮箱：
（1）设置及查看账号：
git config --global user.name "road-vast"
git config --global user.name
（2）设置及查看邮箱：
git config --global user.email "2247820352@qq.com"
git config --global user.email

2、创建仓库
（1）建立一个空目录（仓库）：
mkdir pro_git
（2）进入项目目录：
cd pro_git
（3）Git仓库初始化（让Git知道，它需要来管理这个目录）：
git init（产生一个“.git”的隐藏目录，需要点击查看-》勾选隐藏的项目）

3、Git常用指令操作：
（1）查看当前状态：
git status
（2）添加到缓存区：
git add 文件名（当前所在的目录称为工作区）
	（可以同时添加多个文件到缓存区
		语法1：git add 文件名
		语法2：git add 文件名1 文件名2 ...
		语法3：git add .            [添加当前目录到缓冲区中]
	）
（3）提交至版本库：
git commit -m "注释内容"
（4）若是想要修改已经存至版本库的文件，则重新执行（2）（3）操作即可
~~~



## 1.2 时光穿梭机---版本回退

~~~
步骤
1 查看版本（两种方式）：
第一种方式：git log
第二种方式：git log --pretty=oneline
2 回退操作：
git reset --hard 提交编号
3 回复版本回退：
git reflog
~~~

# 2 远程仓库

## 2.1创建远程仓库

~~~
进入https://github.com/网站，然后点击“Start a  project”，填写Repository（远程仓库名），其他不用改也行，（这里创建一个名为shop的远程仓库）
~~~

## 2.2两种常规使用方式

### 2.2.1基于http协议

~~~
（1）在本地创建一个空目录并进入此目录，这里名为shop
（2）使用clone指令克隆线上仓库到本地
git clone 线上仓库地址（如下图路径）
（3）在仓库上做对应的操作（提交暂存区、提交本地仓库、提交线上仓库、拉取线上仓库）
	1）其他操作跟本地仓库一样
	2）提交线上仓库：git push（如下图表示提交成功，可进入线上仓库查看）
	3）拉取线上仓库（线上仓库有更新，本地和线上仓库应该保持一致）：
		第一步（模拟线上仓库有更新）：先在线上仓库建立一个新的文件：点击Create new file-》输入文件			名-》文件内容-》添加注释-》提交
		第二步：拉取线上仓库：git pull
~~~

![image-20200426113540638](D:\笔记\Git\images\线上仓库地址.png)

​																		【shop线上仓库地址】

![image-20200426120005689](D:\笔记\Git\images\提交线上仓库成功.png)

​																		【shop提交线上仓库成功】



![image-20200426121143315](D:\笔记\Git\images\线上仓库新建文件.png)

### 2.2.2基于SSH协议（推荐）

~~~
与https协议相比，只是影响github对于用户的身份鉴权方式，对于github的其他操作没有影响。
步骤：
（1）创建公私钥对文件
第一步：执行指令（需要先自行安装openSSH）：ssh-keygen -t rsa -C "注册邮箱"
第二步：去指定的公钥文件（id_rsa.pud）中粘贴公钥
第三步：去Github中点击Clone or download-》user ssh-》add a new public key-》上传公钥（如下图）
（2）执行后续git操作，操作与先前一样，使用clone指令等
~~~

  ![image-20200426130608993](D:\笔记\Git\images\创建公私钥.png)

​																			【创建公私钥】

![image-20200426131331943](D:\笔记\Git\images\粘贴公钥.png)																			【上传公钥】

### 2.3分支管理

![image-20200426133108036](D:\笔记\Git\images\master分支.png)

~~~
分支相关的指令：
查看分支：git branch（“*”表示当前所在的分支）
创建分支：git branch 分支名
切换分支：git checkout 分支名
创建并切换带对应的分支：git checkout -b 分支名
删除分支：git branch -d 分支名
	注意：删除时一定要退出所在的分支
合并分支：git merge 被合并的分支名
	操作：
	1 现在dev分支下的readme.txt下新增一行内容，然后提交到本地仓库。
	2 切换到master分支，发现新增的一行在master中没有（分支之间互不影响）
	3 将dev分支中的内容合并到master分支中
		在master分支下执行命令：git merge dev
	4 线上更新master分支：git push
~~~

![image-20200426133747933](D:\笔记\Git\images\查看分支.png)

​																					【查看分支】

![image-20200426134004810](D:\笔记\Git\images\创建分支.png)

​																					【创建分支】

![image-20200426134231263](D:\笔记\Git\images\切换分支.png)

​																					【切换分支】

![image-20200426134809549](D:\笔记\Git\images\添加一行内容.png)

​													【dev分支分支下readme.txt文件新增一行内容】

![image-20200426135314895](D:\笔记\Git\images\切换到master分支观察readme文件.png)

​												【切换到master分支观察readme.txt文件】

![image-20200426135741633](D:\笔记\Git\images\合并分支.png)

​																					【合并分支】

![image-20200426140037455](D:\笔记\Git\images\线上更新master分支.png)

​																					【线上更新master分支】

![image-20200426140329580](D:\笔记\Git\images\删除分支.png)

​																				【删除分支】

### 2.4冲突的产生与解决

~~~
案例：模拟产生冲突：
（1）同事在下班后修改了线上仓库的代码
（2）第二天上班，我没有进行 git pull 操作
（3）我修改了自己本地仓库的内容
（4）上传到线上仓库时报错
解决冲突：
（1）先使用git pull将线上修改的和自己修改的进行合并；
（2）然后和同事商量，删除不必要的内容，重新上传到线上仓库
~~~

![image-20200426142637270](D:\笔记\Git\images\同事小A修改的内容.png)

​																【同事小A下班后修改了线上仓库的内容】

![image-20200426142958979](D:\笔记\Git\images\次日上班写的内容.png)

​																【我次日上班写的内容】![image-20200426143551314](D:\笔记\Git\images\上传到线上仓库报错.png)

​																【上传到线上仓库报错】

![image-20200426144926670](D:\笔记\Git\images\解决冲突之git pull.png)

​																【解决冲突之git pull】

# 3 忽略文件

~~~
在进行添加所有文件到暂存区的操作中，忽略掉某些文件不加进暂存区：
新建一个.gitignore文件：
touch .gitignore
	在文件中新建规则：
	#忽略掉js目录下的文件：
		/js/
~~~

![image-20200426165740482](D:\笔记\Git\images\忽略文件规则.png)

​																		【忽略文件的规则】