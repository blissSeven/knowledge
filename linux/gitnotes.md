# git notes
## git commit 提交  
## git branch 创建分支  
* git branch -f master HEAD~3 强制让master分之指向HEAD的第前三个提交  
* git branch -f bugFix c6    master分之指向c6提交  
* git branch -u o/master foo foo分之跟踪o/master  
* git branch -u o/master 当前分之跟踪o/master  
* git branch feature master  在master提交处 新建分之  
## git checkout  
* git checkout newBranch切换/检出到分之  
* git checkout -b newBranch 创建新分支，并切换到该分支  
* git checkout -b newBranch c1 在c1节点处，创建新分支并切换  
* git checkout master^ HEAD指向master所指向的那次提交的父节点  
* git checkout master~3 HEAD指向master的往上数3个的节点  
* git checkout -b notmaste o/master 新建分之，跟踪远程分之o/master,本地master分之不再跟踪o/master  
## git merge 
* git merge a  将a分之合并到当前所在的分之,当前所在分之继续向下走，形成一个新的提交  
* git merge a b 将b分之合并到a分支，a分之向下走  
## git rebase 
* git rebase master 将当前分之rebase到 master  
* git rebase master bugFix 将bugFix rebase到master  
* git rebase -i HEAD~4 交互式调整记录顺序，删除提交，合并提交,4代表4次提交,从当前HEAD到HEAD~4  
* git rebase -i startpoint endpoint startpoint先提交，endpoint后提交,不包括startpoint,并将rebase后的结果放到startpoint下,startpoint分之不会移动  
  
        pick 保留该commit  
        reword 保留commit,但是编辑commit message  
        edit 保留commit,但是修改该提交，不仅仅修改注释  
        squash 将该commit和上一个commit合并  
        fixup 将该commit和上一个commit合并，但是不保留该commit提交的注释  
        exec 执行shell命令  
        crop 丢弃commit  
## HEAD
* HEAD状态  
    cat .git/HEAD 查看HEAD指向  
    git symbolic-ref HEAD 如果HEAD指向一个引用,查看指向  
    ^向上移动1个提交记录  
    ~3向上移动3个提交记录  
* 分离HEAD状态  
    当HEAD指向一个提交记录而不是分之的时候  

## git reset
* git reset HEAD~1 废弃当前分之的最新一次提交,变更处于未加入暂存区状态   
## git revert
* git revert HEAD 撤销最近一个提交并分享给别人,会生成一个新的提交，同被撤销提交的父节点一样  
## git cherry-pick
* git cherry-pick C1 C2 C3 将C1 C2 c3(HASH值)按照次序复制到当前所在位置HEAD下面,同时HEAD指向的标签会移动到最新  
* git cherry-pick c2 能将任何提交记录 取过来追加到HEAD上(不是HEAD的上游提交就可以)  
## git tag
* git tag v1 C1 在C1提交建立标签v1,不能在v1上commit  
* git checkout v1 检出v1,处于HEAD分离状态  
## git describe
* git describe <ref> ref 任何能被识别成提交记录的引用,没有指定，检出当前位置   
	tag-numcommits-g<hash>  tag 离ref最近的标签，numcommits ref同tag相差多少个提交，  hash ref所表示的提交记录的hash  

## git fetch
* git fetch 将本地仓库远程分之更新为远程仓库相应分之的最新状态，不会更新master分之  
* git fetch origin foo 远程foo分之到本地o/foo,本地foo不移动  
* git fetch origin foo～1:bar 远程foo~推送到本地bar，没有bar分之则创建  
* git fetch origin  :创建本地bar分之   
## git pull
* git pull   git fetch;git merge o/master  将o/master合并到master，master向前走一步  
* git pull origin foo  git fetch origin foo;git merge o/foo  
* git pull orgin master:foo  git fetch origin master:foo  git merge foo  将foo分之 merge到当前分之  
* git pull --rebase   git fetch;git rebase o/master 将当前分之 rebase到o/master ,master向前  
## git push
* git push < remote >  < place >  
* git push origin master 切换到master分之，获取所有提交，再到远程仓库origin找到master分之，将远程仓库没有的提交提交上去place 提示 本地master提交到远程仓库master  
* git push origin <src>:<dst> 将本地src分之推送到远程dst分之  
* git push origin foo^:master  
* git push origin  :foo删除远程foo分之  
## git stash
* git stash save 'message'
* git stash list 
* git stash show 显示做了哪些改动，整体的，默认第一个。第二个`git stash show stash@{1}`
* git stash show -p 显示第一个存储的改动
* git stash apply 应用某个存储，但不会把存储从存储列表中删除，默认第一个。第二个`git stash apply stash@{1}`
* git stash pop 恢复之前缓存的目录，将缓存堆栈中对应的stash删除，将对应修改应用到当前工作目录下。
* git stash drop丢弃stash中数据，默认第一个
* git stash clear 清除所有缓存的stash



