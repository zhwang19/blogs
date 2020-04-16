## 配置git
下载git后，先配置git的用户信息
```
git config --global user.name "your name"
git config --global user.email "your email"
```
## 版本回退
首先先了解git的基本工作流程<br>
<img src="../images/workflow.png" width="450" height="255">
### reset
reset命令有三个选项：
* mixed		默认选项，git会先将仓库的master指针指向前一个快照，同时将该快照回滚到暂存区
* soft		git将master指针指向前一个快照，但是并不会将快照回滚到暂存区，相当于把仓库回退一个版本
* hard 		git将master指针指向前一个快照，同时将该快照回滚到暂存区，然后再将回滚后的暂存区内容还原到工作区

```
git reset --[mixed][soft][hard] HEAD~/snapshot_id
```
HEAD\~表示会提到HEAD指针会回退到上个版本，想退几个版本就加几个\~，或者只用一个\~，但是后面跟想回退版本的次数
也可以直接使用git log查看版本id，通过版本id精确回退
这里需要注意的是，虽然回滚版本的时候用的是HEAD，但是修改指针的时候是HEAD和master一起修改了，这一点很重要，详细看reset和checkout分支命令的区别
reset命令除了回滚快照，也可以检出快照中的文件到暂存区
```
git reset HEAD~/snapshot_id filename
```
注意：reset命令如果针对的是文件，那么选项只能是--mixed,无法对文件指定--soft和--hard选项
### checkout
从暂存区检出文件到工作区
```
git checkout -- filename
```
从快照检出文件到暂存区和工作区
```
git checkout snapshot_id -- filename
```
命令中的双横杠是可以省略的，这是因为checkout除了可以检出文件，还用于切换分支，如果恰好有一个分支名与filename同名，git是不知道到底是要检出文件还是要切换分支点的，所以加双横杠是为了确保checkout命令的操作是要检出文件，当然如果保证你的分支名不会与文件名发生同名冲突，那么双横杠是可以不加的
### reset和checkout检出文件的区别
reset和checkout都可以检出快照中的文件，并且不会改变HEAD的指向，但二者还是有区别的
reset由于对文件操作时只能使用--mixed选项，所以文件只能从仓库检出到暂存区
checkout则会将文件同时检出到暂存区和工作区
相比而言，reset比checkout命令更加安全
### 版本对比
工作目录文件与暂存区文件对比
```
git diff
```
比较两个历史快照的文件
```
git diff snapshot1_id snapshot2_id
```
比较工作目录和仓库快照的文件
```
git diff snapshot_id/HEAD
```
比较暂存区和仓库快照的文件
```
git diff --cached snapshot_id
```
如果不加快照id，则对比的是最新版本的快照文件
### 修改最后一次提交
如果只修改了很少量的文件，又不想再次commit新建一个快照，只想把修改的文件添加到最近的一次快照中，可以在commit命令中加上--amend选项
```
git commit --amend -m "description"
```
### 删除文件
rm命令删除工作区和暂存区的文件,如果文件已经被提交了，那么rm命令不会影响仓库中的该文件，可以用reset回退版本
```
git rm filename
```
另外，如果工作区和暂存区的文件内容不一致，那么是不会删除文件的，可以加入-f选项强制删除
```
git rm -f filename
```
如果只想删除暂存区的文件，保留工作区的文件，加入--caahed选项
```
git rm --cached filename
```
### 重命名文件
```
git mv oldfilename newfilename
```
## 分支
### 创建分支
```
git branch branch_name
```
此操作会新建一个名为branch_name的新分支，并且该分支会指向master分支指向的快照
创建分支的实质，是创建了一个名为branch_name的指针
```
git checkout branch_name
```
此操作会将HEAD指针指向当前的分支，切换不同分支就是通过checkout改变HEAD指针的指向
上面两条命令可以合并为
```
git checkout -b branch_name
```
需要注意的是：切换分支后，不仅仅是改变了HEAD指向，暂存区和工作区也要与当前分支保持一致，其内容也会发生变化，变为当前分支下HEAD指向的快照内容！
```
git log --decorate --oneline --all --graph
```
git log可以查看当前分支的快照，注意，是当前分支！其它分支的快照是看不见的，--decorate可以查看当前所在的分支,--graph可以图形化显示分支
### 合并分支
```
git merge slave_branch
```
将一条从分支合并到主分支
### 删除分支
```
git branch -d branch_name
```
### 匿名分支
如果尝试切换到一个不存在的分支，如
```
git checkout HEAD~
```
git会将HEAD指向master分支的前一个快照，并以匿名形式创建一个新的分支，在此分支所有的提交工作，当执行返回主分支的操作后，都不会保留（因为分支是匿名的，返回主分支后就找不到了）
### reset和checkout分支命令的区别
首先新创建一个分支feature
假设有一个分支feature，该分支与master指向不同的快照，当前在master中，那么请问，单独分别执行下面两条命令有什么不同
```
git checkout feature
```
与
```
git reset --hard feature
```
区别就是：在执行checkout后，只有HEAD指针指向了新的快照feature，而master指针指向的仍然是之前的快照
		 而在执行了reset后，HEAD和master指针全都指向了新快照feature