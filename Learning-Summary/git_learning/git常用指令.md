# 1、上传三步曲

```shell
## 拉取代码：
git clone -b 分支名 地址

## 查询库状态：
git status

##更新远程库到本地：
git pull
eg：	git pull origin main		
本地库代码推送到远程库的分支（首先pull一下确保版本一致）：git push -u origin  "分支名"

## 上传三部曲
（1）git add		添加文件到暂存区
	eg：	git add (文件名)	添加单独的文件nige
	eg：	git add (.)		添加当前文件夹下所有文件 （. 不能省略）
	
（2）git commit	提交到本地仓库
	eg：	git commit -m “提交信息”		注：“提交信息”里面换成你需要，如“first commit”
	
		特殊指令：查看git当前分支上面都存了什么文件：	git ls-files  （添加到缓存区的文件）
												git ls-tree --name-only “branch”  （提交到本地库的文件）
（3）git push	推送到远程仓库
	eg：	git push origin main	注：origin 是远程库别名，其实就是你的仓库地址，main 表示要推送到的远程仓库分支（github默认是main，Gitee默认为master）
```

# 2、远程库别名设置

```shell
## 查询远程别名		
git remote -v	

## 增加远程库别名	
git remote add origin 地址 	
eg：	git remote add origin https://github.com/FPGAmaster-wyc/test.git
	
## 删除/修改远程库别名	删除：
git remote rm origin	
eg:修改：git remote set-url origin 地址
```

# 3、分支管理		

```shell
## 查询分支			
git branch -v
	
## 创建分支			
git branch 分支名

## 分支切换			
git checkout 分支名
## 创建并切换	
git checkout -b 分支名

## 删除分支			
git branch -d 分支名

## 合并分支			
git merge 分支名   注：首先切换到要合并的分支上

## 上传分支到远程库：
git push origin "分支名"
```

# 4、删除文件

```shell
## 只删除暂存区文件（add后的文件）	
git rm --cached file 
	eg：	git rm --cached FPGA.gif

## 删除暂存区和工作区文件（前提是add后）			
git rm -f file 
	eg： git rm -f FPGA.gif
```

# 5、日志查询

```shell
## 以一行的形式展示  
git log --oneline

## 展示所以的历史版本	
git reflog			（优点：即使reset之后，仍然能够通过reflog找到）
```

# 6、版本前进后退

```shell
## 版本后退
git reset
	1. 	Soft	尽在本地库移动，缓存区，工作区不变
	2.	Mixed	本地库移动，重置缓存区
	3.	Hard	三大工作区重置
	
一般使用 Hard指针进行前进和后退		git reset --hard 【索引值】		（索引值通过日志查看出来）
	eg：	git reset --hard 4810a8c
	
## 从暂存区恢复到工作区	
git restore file 
	eg：	git restore test.test
```

# 7、其他

```shell
## 追加提交		
git commit --amend 		（优点：最终你只会有一个提交——第二次提交将代替第一次提交的结果）
		适用于：提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了
```





