# git reset与git revert

> 一句话描述git reset与git revert的区别：
`git revert`是用一次**新的提交**来回滚之前的**某次**提交，`git reset`是直接**回退到**某个版本，**删除**该版本之后的提交。

在git使用中，要对已提交的内容回退，是使用`git reset`还是使用`git revert`，这两个命令有什么区别，在这里做下整理、说明，方便了解。

## 概念回顾
在开始之前，我们先回顾几个概念：
- Working Directory（工作目录） 
- GIT Index（Git索引）
- GIT Directory（GIT库目录） 

哪些操作能够改变git index中的内容？ 
- `git add <path>...`会将working directory中的内容添加进入git index。 
- `git reset HEAD <path>...`会将git index中path内容删除，重新放回working directory中。 

## 准备工作
1. `git clone`一份代码，并创建测试分支；
2. 在测试分支创建3次提交；
3. 为了方便查看提交日志，设置alias.lg；
```bash
$ git config alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

$ git lg
* e05910d - (HEAD, simple-1) add third line to simple.log (12 minutes ago) <iamsujun>
* 80b7d6a - add second line to simple.log (12 minutes ago) <iamsujun>
* 0c6c743 - add first line to simple.log (16 minutes ago) <iamsujun>
* 9917b2a - (origin/master, origin/HEAD, master) create README (7 weeks ago) <iamsujun>
```

## git reset
git reset常用的有3种方式：mixed、soft、hard。

### git reset --mixed
默认方式，回退到某个版本，保留代码，提交、索引回退。
```bash
$ git reset 80b7d6a
Unstaged changes after reset:
M       simple.log

$ git status
On branch simple-1
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   simple.log

no changes added to commit (use "git add" and/or "git commit -a")

$ git lg
* 80b7d6a - (HEAD, simple-1) add second line to simple.log (14 minutes ago) <iamsujun>
* 0c6c743 - add first line to simple.log (17 minutes ago) <iamsujun>
* 9917b2a - (origin/master, origin/HEAD, master) create README (7 weeks ago) <iamsujun>
```

### git reset --hard
回退到某个版本，包括代码、提交、索引。
```bash
$ git reflog
80b7d6a HEAD@{0}: reset: moving to 80b7d6a
e05910d HEAD@{1}: commit: add third line to simple.log
80b7d6a HEAD@{2}: commit: add second line to simple.log
0c6c743 HEAD@{3}: commit: add first line to simple.log
9917b2a HEAD@{4}: checkout: moving from master to simple-1
9917b2a HEAD@{5}: clone: from d:/gitTest/git_test_1.git/

$ git reset --hard e05910d
HEAD is now at e05910d add third line to simple.log

$ git status
On branch simple-1
nothing to commit, working directory clean

$ git lg
* e05910d - (HEAD, simple-1) add third line to simple.log (2 hours ago) <iamsujun>
* 80b7d6a - add second line to simple.log (2 hours ago) <iamsujun>
* 0c6c743 - add first line to simple.log (2 hours ago) <iamsujun>
* 9917b2a - (origin/master, origin/HEAD, master) create README (7 weeks ago) <iamsujun>
```

### git reset --soft
回退到某个版本，只回退提交。
```bash
$ git reset --soft 80b7d6a

$ git status
On branch simple-1
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   simple.log

$ git lg
* 80b7d6a - (HEAD, simple-1) add second line to simple.log (2 hours ago) <iamsujun>
* 0c6c743 - add first line to simple.log (2 hours ago) <iamsujun>
* 9917b2a - (origin/master, origin/HEAD, master) create README (7 weeks ago) <iamsujun>
```
为了后面演示`git revert`用，我们对代码进行回退。
```bash
$ git reflog
80b7d6a HEAD@{0}: reset: moving to 80b7d6a
e05910d HEAD@{1}: reset: moving to e05910d
80b7d6a HEAD@{2}: reset: moving to 80b7d6a
e05910d HEAD@{3}: commit: add third line to simple.log
80b7d6a HEAD@{4}: commit: add second line to simple.log
0c6c743 HEAD@{5}: commit: add first line to simple.log
9917b2a HEAD@{6}: checkout: moving from master to simple-1
9917b2a HEAD@{7}: clone: from d:/gitTest/git_test_1.git/

$ git reset --hard e05910d
HEAD is now at e05910d add third line to simple.log

$ git lg
* e05910d - (HEAD, simple-1) add third line to simple.log (3 hours ago) <iamsujun>
* 80b7d6a - add second line to simple.log (3 hours ago) <iamsujun>
* 0c6c743 - add first line to simple.log (3 hours ago) <iamsujun>
* 9917b2a - (origin/master, origin/HEAD, master) create README (7 weeks ago) <iamsujun>
```

## git revert
用新的提交对某次提交进行“中和”。
```bash
$ cat simple.log
first
second
third

$ git revert e05910d
[simple-1 d2cd98f] Revert "add third line to simple.log"
 1 file changed, 1 deletion(-)

$ git lg
* d2cd98f - (HEAD, simple-1) Revert "add third line to simple.log" (17 seconds ago) <iamsujun>
* e05910d - add third line to simple.log (3 hours ago) <iamsujun>
* 80b7d6a - add second line to simple.log (3 hours ago) <iamsujun>
* 0c6c743 - add first line to simple.log (3 hours ago) <iamsujun>
* 9917b2a - (origin/master, origin/HEAD, master) create README (7 weeks ago) <iamsujun>

$ cat simple.log
first
second
```
我们发现，一个新的提交“d2cd98f”将“e05910d”中的修改覆盖掉。
