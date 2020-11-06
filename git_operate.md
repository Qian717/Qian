一、克隆与提交

​		$ git clone https://github.com/Qian717/Qian

​		$ cd Qian/

​		$ git config --global user.name "***"  //绑定用户名

​		$ git config --global user.email “***” //绑定邮箱

​		$ touch test.txt  //新建文件

​		$ git add test.txt //把test.txt文件添加到缓存区

​		$ git commit -m "add test.txt"   // -m 后的参数 " add test.txt "是对这个提交的描述

​		$ git push  //提交

------

二、删除仓库文件

​	方法一、先删除本地仓库

​			1.首先删除本地仓库文件 git rm <file>此时本地文件没有了，github仓库上还有此文件。

​			2.提交并添加备注 git commit -m "备注"

​			3.把删除的文件恢复到最新版本 git checkout

​			4.将本地内容推送到远程仓库git push



​	方法二、直接删除远程仓库（后补）

-----

三、状态查看

​			$git status //查看仓库状态

​			$git log //查看本仓库的提交日志

​			$git log --pretty=short //只显示提交信息的第一行

​			$git log README.md //显示指定目录、文件的日志

​			$git log -p README.md //显示文件的改动，可查看文件的提交日志和提交前后的差别

​			$git diff //查看工作树和暂存区的差别

​			$git diff --cached //查看暂存区和仓库的差别

​			命令git checkout – first.md意思就是，把first.md文件在工作区的修改全部撤销，这里有两种情况：

​			一种是first.md自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

​			一种是first.md已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

​			Ls -ah  //查看所有文件包括隐藏文件

​			用git编辑文件后保存退出操作，Esc+shift键， **: wq**

​			Rm -rf  //强制删除文件以及非空目录

 

------


工作区、暂存区、仓库

​		Git管理的文件分为：工作区，版本库，版本库又分为暂存区 stage和暂存区分支 master(仓库)
​		git add  把文件从工作区>>暂存区， git commits把文件从暂存存>>仓库
​		git diff  查看工作区和暂存区差异
​		git diff --cached 查看暂存区和仓库差异
​		git diff  HEAD查看工作区和仓库的差异
​		git add  的反向命令 -git checkout  撤销工作区修改，即把暂存区最新版本转移到工作区
​		git commite的反向命令 git reset HEAD  就是把仓库最新版本转移到暂存区

-----

上传图片（后补）

 

