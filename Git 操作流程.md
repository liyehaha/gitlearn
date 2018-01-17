# Git 操作流程

**1: 检查当前分支。**执行`git branch`命令可以查看所在目录下的分支列表，当前分支会在分支名字前面用`*`	表示

	```
	$ git branch
	  master
	* test-1
	```
	
**2: 创建自己的开发分支。**执行`git checkout -b <分支名称>`可以切换到分支`<分支名称>`，如果这个分	支不存在，则会自动创建。或者使用`git branch <分支名称>`创建分支后，再执行`git checkout <分支名称>`	切换到该分支。两种方式都可以使用
	
	```
	$ git checkout -b test-1
	Switched to a new branch 'test-1'
	```
	或者
		
	```
	$ git branch test-1
	$ git checkout test-1
	Switched to branch 'test-1'
	```
	```
	#查看分支列表
	$ git branch -a
	* master
	  remotes/origin/master
	  remotes/origin/test-2
	  
	#带*的代表当前分支，以remotes开头的代表远程的分支 "remotes/<远程仓库>/<分支名称>"
	```
**需要注意的是，创建的分支都是基于当前目录所在分支为基础新建的分支，而不是基于master。如果需要在	master	的代码版本基础上创建分支，需要先切换到master分支，再创建分支**

**3: 将修改的代码加入到版本控制中。**使用`git add`命令可以添加文件或目录到当前的版本管理中，使用`git rm`	可以将文件或文件夹从当前的版本管理中移除。只有在版本管理中的文件才可以被git管理（提交\<pull\>，检出	\<checkout\>，或合并\<merge\>）。
	
	```
	#这里假设我们修改了test目录下的1.txt文件。可以通过执行以下命令增加版本控制：
	git add test/1.txt
	#或者
	git add test/
	```
		
	```
	#如果我们删除了test目录下的1.txt文件。可以通过执行以下命令移除版本控制：
	git rm test/1.txt
	```
	
**4: 将修改提交到仓库。**使用`git commit`命令可以把已经添加到版本管理中的文件，提交到仓库中(本地或远		程)。
	
	```
	$ touch 123
	$ git add 123
	$ git commit -m 'test'
	[test-1 bbefd2d] test
	 1 file changed, 0 insertions(+), 0 deletions(-)
	 create mode 100644 123
	$ git push
	fatal: The current branch test-1 has no upstream branch.
	To push the current branch and set the remote as upstream, use
	
	    git push --set-upstream origin test-1
	# 这里由于是第一次像远程分支test-1提交代码，需要使用--set-upstream命令
	# 将本地分支和远程分支绑定。
	$ git push --set-upstream origin test-1
	```
	
**5: 切换到master分支，并合并代码。**执行完上面的步骤以后，当前的开发分支的最新版(HEAD版本)已经有修	改好的代码了，现在需要将代码提交到远程。首先要切换到master分支，执行`git checkout master`。切换后，	为了保证本地master分支的代码和远端master的代码同步，需要先使用`git pull`更新一下。更新好以后就可以使	用`git merge`命令合并代码了。
	
	```
	$ git checkout master
	Switched to branch 'master'
	Your branch is up-to-date with 'origin/master'.
	$ git pull
	Already up-to-date.
	$ git merge test-1
	Auto-merging 123
	CONFLICT (content): Merge conflict in 123
	Automatic merge failed; fix conflicts and then commit the result.
	```
	<a color="red">CONFLICT (content): Merge conflict in 123
	Automatic merge failed; fix conflicts and then commit the result.</a></br>
	这里出现了一个错误，原因是因为有冲突，提示`Automatic merge failed`，这时候我们就需要解决冲突后在重新	commit文件，然后再提交到远程的master。
		
	```
	<<<<<<< HEAD
	123
	=======
	234
	>>>>>>> test-1
		
	# 这是冲突的文件内容，其中<<<，===，>>>代表了代码冲突的分隔符。
	# <<<和===之间的代码是当前master分支的内容，===和>>>之间的代码是合并过来的代码内容，
	# 具体的修改可以和其他开发人员确认或者使用git diff命令对比后再操作。解决完冲突后要删掉分隔符。
	# 修改完成后的代码如下:
		
	123
	234
	```
	这时候我们重新将文件添加到版本控制、提交到仓库、提交到远程master就可以了。<a color="red">（如果没有冲突，直接执行`git push origin master`命令就可以了，不需要重新提交）</a>
		
	```
	$ git add 123
	$ git commit -m '解决冲突'
	[master 91a1471] 解决冲突
	$ git push origin master
	Counting objects: 3, done.
	Delta compression using up to 8 threads.
	Compressing objects: 100% (2/2), done.
	Writing objects: 100% (3/3), 311 bytes | 311.00 KiB/s, done.
	Total 3 (delta 0), reused 0 (delta 0)
	To https://github.com/liyehaha/gitlearn.git
	   0e8720e..91a1471  master -> master
	```
	
**6: 提交完成后的操作。**当所有代码都提交到远程的master分支后，我们可以先检查一下，是否有漏掉的文件。
	可以使用`git status`命令查看一下。
	
	```
	$ git status
	On branch master  #代表当前分支
	Your branch is up-to-date with 'origin/master'.  #代表当前分支对应的远程分支
	
	nothing to commit, working tree clean
	```
	
	当看到最后一句话<a color="red">nothing to commit, working tree clean</a>时，说明本地的代码已	经全部提交完成。</br>
	
	</br>
	
	当我们再继续新的开发时，还需要切换的开发分支。这时候由于远程的开发分支`origin test-1`中的代码已经不是	master的最新代码了，建议先删除掉本地和远程的开发分支`test-`，然后再重新基于master的最新代码创建分支	后，进行开发。具体操作如下:
	
	```
	$ git branch
	* master
	  test-1
	$ git branch -D test-1
	Deleted branch test-1 (was 67da65a).
	$ git push origin :test-1
	To https://github.com/liyehaha/gitlearn.git
	 - [deleted]         test-1
	$ git branch
	* master
	```
	`git branch -D <分支名称>`命令可以删除一个本地分支</br>
	`git push origin :<分支名称>`命令可以删除一个远程分支</br>
	
	当全部删除完成后，我们就可以从第一步开始从头操作了！