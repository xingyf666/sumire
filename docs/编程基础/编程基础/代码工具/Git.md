# Git

## 基本介绍

Git 是一种分布式管理工具，用于进行版本控制，以便将来查询特定版本修订情况的系统。

![[Screenshot_20230412_195240.jpg|300]]

代码托管中心是基于网络服务器的远程代码仓库，简称为**远程库**

* 局域网
    * GitLab
* 互联网
    * GitHub
    * Gitee

由于 Cmder 在安装时已经有 Git 了，不再额外下载 Git 。



### 全局配置

#### 用户信息

首先要配置全局的用户信息

```shell
git config --global user.name name		# 设置用户签名
git config --global user.email email	# 设置用户邮箱
```

这里设置的用户签名和之后 github 或 gitee 账号没有关系。我们可以用

```shell
git config
```

来查询相关指令，找到 --get 参数，使用

```shell
git config --get user.name
git config --get user.email
```

来查看设置信息；也可以在 `C:\Users\Admin` 目录下，用

```shell
cat .gitconfig
```

显示 git 的配置文件进行查看。或者指令查看

```shell
git config --global --list
```

还可以移除配置

```shell
git config --global --unset user.name
```



#### 面密提交

新建 `.git-credentials` 文件，按照格式

```shell
https://{username}:{passwd}@gitee.com
```

在该文件中输入账号和密码如下

```shell
https://xingyf666:www.xing@126.com@gitee.com
```

然后使用全局指令 --global 配置账号密码信息，实现免密提交

```shell
git config --global credential.helper store
```

此时打开 `.gitconfig` 就可以看到如下信息

```shell
[credential]
	helper = store
```

注意由于我使用的密码中有 `@` 字符，可能破坏了格式，所以配置不成功。



#### 生成公钥

Gitee 提供了基于 SSH 协议的 Git 服务，在使用 SSH 协议访问仓库之前，需要先配置好账户/仓库的 SSH 公钥

```shell
ssh-keygen -t ed25519 -C "xxxxx@xxxxx.com"
```

这里的 `xxxxx@xxxxx.com` 只是生成的 sshkey 的名称，并不约束或要求具体命名为某个邮箱。



照提示完成三次回车，即可生成 `ssh key` 。在目录 `C:\Users\Admin\.ssh` 下可以找到 `id_ed25519.pub` 文件，其中包含公钥

```shell
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIkiGC8TdhfoXXP+kz1qHgS7SgPq4eVWh9jqHjsjF5zO 814083398@qq.com
```

注意这些全部都是公钥。复制生成后的 ssh key，通过仓库主页 **「管理」->「部署公钥管理」->「添加部署公钥」**，添加生成的 public key 添加到仓库中。GitHub 可以在 **Settings -> SSH and GPG keys** 中添加公钥。



#### 凭据管理

在 git clone 时，会要求输入账户和密码。如果输错了就会一直报错，无法 clone 。这时候就需要在控制面板搜索凭据管理器，删除网站的凭据。

![[image-20231214214730353.png]]



### 远程库连接

建立一个文件夹，在其中使用

```shell
git init
```

Git 就会新建一个 `.git` 文件夹，它是隐藏文件夹。然后添加远程库并拉取

```sh
git remote add name addr
git pull name master
```



#### Remote

远程库常用操作

```shell
git remote -v 					# 查看当前所有远程地址别名
git remote add <name> <addr> 	# 添加远程库别名
git remote show <name>			# 显示远程库信息
git remote rm <name>			# 移除远程库
git remote rename <name>		# 重命名远程库
```



#### Clone

可以使用 clone 命令将 gitee 上的代码下载到本地

```shell
git clone https:// ... .git
```

任何开源代码都可以直接 clone，它会执行如下操作：

* 拉取代码
* 初始化本地仓库
* 创建别名

因此克隆操作相当于直接建立了本地连接。



如果不想为库做贡献，可以设置克隆深度

```shell
git clone ... --depth=1
```

这样只会克隆最新的版本，不包含历史提交信息，因此下载速度更快。



### 代码提交

编辑本地内容后，就可以将代码提交到本地缓存区，然后推送到远端。



#### Status

通过 status 命令查看 git 状态

```shell
git status
```

此时会显示当前所在的分支以及代码提交的信息。例如

```shell
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   src/main.cpp

no changes added to commit (use "git add" and/or "git commit -a")
```

增加 -s 参数显示简略信息

```she
git status -s
```



#### Add

需要使用 add 添加文件到暂存区

```shell
git add file	# 添加指定的文件
git add .		# 添加所有变更的文件
```



#### Commit

通过此命令提交暂存区的文件到本地库

```shell
git commit -m "log commit" file
```

然后可以使用 push 进行提交。



#### Stash

通过此命令将当前工作目录和暂存区的修改（尚未提交的更改）保存到一个临时存储区域中。

```shell
git stash        # 暂存当前的修改
git stash list   # 查看所有暂存的工作进度
git stash apply  # 恢复最近一次的暂存工作进度
```

它和 commit 的区别在于不会提交到本地库，从而避免创建新的提交记录。当执行

```shell
git stash
git pull
```

此时之前暂存的部分修改会被覆盖，如果希望恢复之前暂存的内容，使用

```shell
git stash pop
```

这样就可以解决冲突合并问题。



#### Restore

通过此命令取消对本地文件的更改

```cpp
git restore file
```

它的作用是当本地文件与远程分支冲突时，可以回滚本地文件，避免冲突。



#### Pull

如果**本地内容修改之前与远程库内容不一致，则本地内容修改后提交的操作会被拒绝**。这时就需要拉取远程代码与本地合并

```shell
git pull <name> <branch>
```

此时将会用远程库更新的内容覆盖掉本地的部分代码。只要没有产生冲突，就可以直接 push 推送；否则**要解决冲突后重新提交推送**。



使用 --rebase 操作可以使分支合并更加优美

```shell
git pull --rebase <name> <branch>
```

这是因为 pull 会在这里合并分支产生“多余的” commit 信息。注意此时**本地文件的修改必须都已经通过 commit 提交**，否则会报错。



#### Push

推送本地库到远程库

```shell
git push <name> <branch>
```

注意，当远程库与本地库不一致，应当先拉取远程库到本地。当然也可以强制推送

```shell
git push --force <name> <branch>
```

它会忽略版本不一致的情况，用本地内容强制覆盖远程库，通常不建议使用。



#### Diff

比较文件的不同，并且显示出改动的位置。有几种用法

* `git diff` 本地文件的改动
* `git diff --cached` 通过 add 提交的改动
* `git diff HEAD` 查看上述两种改动
* `git diff origin/main` 比较远程库

可以比较两次提交之间的差异

```shell
git diff [branch-1] [brach-2]
```



#### Rm

使用 rm 指令删除指定的文件。有几种形式

* `git rm <file>` 删除本地和暂存区的文件
* `git rm -f <file>` 强制删除修改后的本地和暂存区的文件
* `git rm --cached <file>` 删除添加到暂存区的文件
* `git rm -r *` 递归删除目录下所有文件和子目录



#### Reflog

可以查看提交历史记录

```shell
git reflog
```

例如 git reflog 可能得到

```shell
52037cb (HEAD -> master) HEAD@{0}: commit: version 0.0.2
2df5fe8 HEAD@{1}: commit: version 0.0.1
1690a91 HEAD@{2}: initial pull
```

其中每行都是每次提交的版本及其描述信息。



#### Log

使用功能更强的 log 命令查看具体的信息

```shell
git log
```

可以添加参数

* `--oneline` 查看简洁信息（比 reflog 更简洁，只显示不同版本）
* `--graph` 查看图形化历史信息
* `--reverse` 反向显示日志
* `--author=name` 指定查看用户提交日志



#### Blame

查看指定文件的修改记录

```shell
git blame <file>
```



### 版本穿梭

使用 git 进行版本穿梭可以快速在不同的代码版本之间切换。



#### Reset

用于回退版本，可以指定退回某次提交的版本。语法为

```shell
git reset [--soft | --mixed | --hard] [HEAD]
```

默认为 --mixed 选项

* mixed 移动 HEAD 指针，**重置缓存区**文件为之前提交 (commit) 时的情况，**工作区内容不变**
* soft 移动 HEAD 指针，但是不修改缓存区和工作区，也就是**只回退了版本号**
* hard 移动 HEAD 指针，同时重置缓存区和工作区为之前的版本。不建议使用，因为非常危险，容易丢失工作区进度

使用 HEAD 选项指定回退的版本

* `HEAD` 当前版本
* `HEAD~1` 上一个版本
* `HEAD~2` 上上一个版本



如果回退过头，需要先通过 reflog 获得之前的版本信息。例如在前面得到的版本信息下

```shell
52037cb (HEAD -> master) HEAD@{0}: commit: version 0.0.2
2df5fe8 HEAD@{1}: commit: version 0.0.1
1690a91 HEAD@{2}: initial pull
```

执行 hard 参数命令

```shell
git reset --hard 1690a91
```

就可以回到初始化时的状态。

>[!note]
>利用 `--soft` 选项可以撤销之前的 commit，进而实现 commit 合并。



#### 远程库提交

如果需要回退远程库，首先就要查看远程库的提交记录

```shell
git log origin/main
```

然后指定 hash 值进行回退

```shell
 git reset --hard fd09bdfd9a8c2fa4bd6ea7dcfe5825bccf411b54
```



#### 合并 commit

当 PR 提交时，为了简化多余的历史记录，需要将分支中的 commit 合并为同一个。依次执行

```shell
git log	# 查看远程库历史记录
git reset --soft commitID
git add .
git commit --amend -m "merge"
```

然后推送代码，即可完成 commit 合并。如果合并时，当前分支存在落后于目标分支的 commit，只能在合并 commit 之后再 rebase/merge 目标分支，如果顺序倒过来，reset 导致的回退将会取消掉 rebase/merge 的效果。



### 分支操作

分支的好处在于可以同时并行推进多个功能开发，提高开发效率。各个分支在开发过程中，如果某一个分支开发失败，不会对其他分支有任何影响，失败的分支删除重新开始即可。



#### Branch

分支创建命令

```shell
git branch			# 查看本地分支
git branch name		# 创建分支
git branch -v		# 查看分支，显示对应版本
git branch -d		# 删除分支
```

当 master 中有代码时，才能进行分支操作。其中不带有 * 号的分支才可以删除（实际上带有 * 号的分支是当前操作的分支）。



当远程分支被删除，需要对应删除本地的分支，并更新本地记录的远程分支

```sh
git branch -r			# 查看远程分支情况
git branch -d rebuild	# 删除本地分支 rebuild
git fetch --prune		# 更新远程分支记录
```

如果本地分支未合并，可能会报错。可以使用 `-D` 参数强制删除

```sh
git branch -D rebuild
```



#### Checkout

用于切换分支

```shell
git checkout name		# 切换到指定分支
git checkout -b name	# 创建分支并切换到该分支
```

我们在不同分支下编辑的文件就是当前分支下的文件，切换分支就可以查看和编辑所在分支的同一文件。



具体流程为：

* 创建新的分支，并切换到该分支
* 修改代码，执行 commit 提交
* 切换回原分支

每次修改完后，都需要 commit 提交才可以切换到其它分支。



不仅如此，还可以用于回滚，主要目的是将指定文件恢复到之前的版本

```shell
git checkout <commit-hash> -- <file-path>
```

如果使用

```shell
git checkout <commit-hash>
```

会将所有文件回滚到指定提交，此时头指针将会分离，需要重新切换回分支

```shell
git checkout master
```

>[!note]
>适用于恢复文件以及临时查看历史版本。



#### Merge

将任何分支合并到当前分支

```shell
git merge name	# 将指定分支合并到当前分支
```

注意合并操作不是合并分支本身，而是合并代码，分支仍然存在，只是两部分代码相互覆盖了。



当两个分支中的内容相互冲突时，就会合并失败，此时必须先处理分支的冲突，否则无法切换分支。例如 master 分支下的文件

```cpp
#include "hello.h"
#include <iostream>

void Hello()
{
    std::cout << "World";
}
```

通过 commit 提交完成，然后创建 testing 分支，修改为

```cpp
#include "hello.h"
#include <iostream>

void Hello()
{
    std::cout << "Hello World";
}
```

然后 commit 提交；执行合并

```shell
git merge master	# 将 master 合并到 testing 上
```

由于出现代码冲突，文件中会显示

```cpp
#include "hello.h"
#include <iostream>

void Hello()
{
<<<<<<< HEAD
    std::cout << "Hello World";
=======
    std::cout << "World";
>>>>>>> master
}
```

解决冲突的代码

```cpp
#include "hello.h"
#include <iostream>

void Hello()
{
    std::cout << "Hello World";
}
```

然后 commit 提交，注意不能加文件名，因为 commit 要对两个分支都进行操作。最后再执行合并操作

```shell
git merge master --continue
```

合并将会修改当前分支，不会影响 master 分支。



#### Rebase

可以使用 rebase 来合并分支

```shell
git rebase name
```

下图给出了 merge 和 rebase 的区别：

* merge 将当前分支的 C 版本与指定分支的 C' 版本共同合并到当前分支上，得到 D 版本；
* rebase 将当前分支的 C 版本作为基底，把指定分支的 B', C' 两个版本接到 C 版本的后面；

使用 merge 可以保留分支分离与合并的时间与关系，但是每次合并会产生多余的 commit 信息；使用 rebase 只会保留一条分支历史，因此没有之前的分离与合并的关系，但是能够使分支推进过程更加清晰。

![[Screenshot_20230513_221156.jpg]]

常用的变基参数有

```shell
git rebase --continue 	# 继续下一个冲突
git rebase --skip 		# 跳过当前冲突
git rebase --abort 		# 退出 rebase 模式
```

使用 rebase 合并冲突的情况与 merge 类似，执行

```shell
git rebase master
```

如果出现冲突提示，就需要修改冲突文件，然后执行

```shell
git add .
git commit -m "rebase"
```

现在冲突已经解决，可以执行

```shell
git rebase --continue
```



#### Fetch

将远程仓库的最新版本拉取到本地

```shell
git fetch name	# 拉取最新版本到本地
```

与 pull 不同，fetch 将最新代码拉取后创建一个新的分支 `FETCH_HEAD`，此时本地的代码不会更改。还需要执行

```shell
git merge FETCH_HEAD
```

将该分支与本地分支合并。因此

```shell
git pull origin master
```

等价于

```shell
git fetch origin master
git merge FETCH_HEAD
```



### 版本标签

在某一个版本的开发完成后，可以对代码打一个标签

```shell
git tag -a v1.0.0 -m "完成v1.0.0版本"
```

这样可以在仓库中看到对应的 Tag 版本。创建 Release 发行版本时，就是选择要发行的 Tag 版本。



### .gitignore

由于代码编译后产生的中间文件和结果文件都不是必要的，全部上传会让代码变得臃肿，因此 gitignore 文件用于取消上传某些文件。



#### 基本语法

基本语法规则如下：

* `#` 表示注释；

```makefile
# 注释
```

* 斜杠结尾表示文件夹；

```makefile
src/build/
```

* `!` 表示取反，不会忽略指定的文件或目录；

```makefile
!main.exe
```

* glob 模式匹配；正则表达式的简化版本

    * `*` 表示任意数量的任意字符；
    * `**` 表示匹配任何中间目录；
    * `[abc]` 匹配方括号中的任意字符；
    * `[0-9]` 匹配 0-9 之间的任意一个数字；
    * `?` 表示任意一个字符；

```makefile
# 匹配所有 txt 文件
*.txt

# 匹配中间目录
src/**/build/

# 匹配任意数量的 cdb
[cdb]*

# 匹配任意数量的数字
[0-9]*
[1-9]*

# 匹配 build 目录下的 .json 文件
build/*.json
```



#### 取消追踪

对于尚未提交的内容，可以直接将其添加到 `.gitignore` 中；但如果已经提交的内容，即使添加到 `.gitignore` 中也不会停止追踪。这就需要将其从 Git 索引中删除

```shell
git rm --cached example.txt
git commit -m "stop tracking example.txt"
```



#### 大小写

可以启用大小写检查

```shell
git config core.ignorecase false
```

不过似乎没什么用，还是先删除所有要修改的内容，提交一次之后再恢复内容，再提交一次。



#### 文件模板

官方提供了常用的 [gitignore](https://github.com/github/gitignore) 文件模板，对不同的编程语言提供了对应的文件。在**创建仓库**时，可以指定 `.gitignore` 的模板

![[image-20230515153115723.png]]



### .gitkeep

正常情况下，Git 只会跟踪文件的变化，而不会跟踪空目录。当需要保留一个空目录时，可以在目录下添加一个名为 `.gitkeep` 的文件，这样 Git 就会将该目录以及其中的 `.gitkeep` 文件一起跟踪和保存。



### pull-request

PR 可以通过 fork 仓库或者其它分支向主分支提交合并请求。如果当前分支同时存在领先和落后目标分支的 commit，就会产生代码冲突。首先需要将目标分支合并到当前分支

```sh
git merge master	# 将 master 合并到当前分支
```

然后通过 VS 或者 VSCode 解决冲突的代码，最后再进行提交。



### 工作流

经典的 GitFlow 工作流如图所示，其中涉及到的主要分支类型有

* master 分支，即**主分支**。项目发布版本所在的分支；
* develop 分支，即**开发分支**，从 master 分支中分出。一般不会直接更改该分支，而是从开发分支中再分离出 feature 分支。开发完成后将 feature 分支的改动合并到 develop 分支。Release 分支从 develop 分支中分出；
* release 分支，即**发布分支**，从 develop 分支中分出。该分支用于发布前的测试，进行简单的 bug 修复。如果 bug 修复比较复杂，需要合并回 develop 分支，由其它分支修复。此分支测试完成后，需要同时合并回 develop 和 master 分支；
* feature 分支，即**功能分支**，从 develop 分支中分出。团队中每个人维护一个 feature 分支，开发完成后合并回 develop 分支；
* fix 分支，即**补丁分支**，从 develop 分支中分出。用于 bug 修复，修复完成后合并回 develop 分支，然后将其**删除**；
* hotfix 分支，**热补丁分支**，从 master 分支中分出。进行线上版本的 bug 修复，修复完成后合并回 develop 和 master 分支；

![[Notepad_202305131557_22661.png]]



## GitHub

下面介绍 GitHub 的基本使用方法。



### 基本概念

| 概念                  | 作用                                                         |
| --------------------- | ------------------------------------------------------------ |
| 仓库 Repository       | 用来储存项目代码                                             |
| 分支 Fork             | 将他人的代码分支到自己的项目，然后对方的主页上就多了一个项目 |
| 发起请求 Pull Request | 基于 Fork，如果别人 Fork 你的代码，并进行了改进，他可以发起请求 PR |
| 事务卡片 Issue        | 发现代码 BUG，但是还没有成型代码，需要讨论时使用             |



### 个人令牌

从 21 年 8 月 13 以后就不再支持用户名密码的验证方式，需要创建个人访问令牌。在 Settings 中找到 Developer settings，点击 Personal access tokens 中的 Tokens 即个人令牌，选择创建新的令牌。为了避免麻烦，设置有效期 Expiration 为永久。

![[image-20230801132850147.png]]

将下面的选项全部选中，然后生成。在 push 代码时将下面的 Tokens 复制到密码的位置即可成功。

![[image-20230801133059603.png]]

为了方便起见，可以将令牌加入远程仓库链接。修改现有项目的 url

```shell
git remote set-url origin https://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@github.com/xingyf666/<repo>.git
```

其中 repo 是仓库名。如果克隆仓库，可以使用这一 url 之间获得正确格式的仓库链接。



### 端口管理

#### ssh

有时候使用 ssh 向 GitHub 提交和拉取代码时，会出现连接错误

```sh
$ git push origin cover_wire
ssh: connect to host github.com port 22: Connection timed out
fatal: Could not read from remote repository.
```

这时候可以先测试 443 端口

```sh
$ ssh -T -p 443 git@ssh.github.com
Hi xxxx! You've successfully authenticated, but GitHub does not provide shell access.
```

出现上述结果说明此端口可用。然后在存放公钥和私钥的目录 `.ssh` 下增加 config 文件

```shell
Host github.com
User hellojue@foxmail.com
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_ed25519
Port 443
```

然后就可以正常推送。



#### http

如果按照上面的方式修改了端口，可能导致 http 方式拉取提交代码出现问题，这时候就修改回原先的端口

```shell
Host github.com
User hellojue@foxmail.com
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_ed25519
Port 22
```



### Issues

Issues 用于提出 bug 等评论功能，每个 Issues 会有对应的标号 `#1,#2,...`，在根据 Issue 修改代码后，新的提交可以命名为

```shell
git commit -m "Solve #1"
```

打开页面后，可以在 commit 中直接跳转到对应的 Issue，非常方便。

![[image-20240317144545593.png]]



## Termux

### 基本介绍

可以运行在手机端的命令行工具。如果在执行命令时出现莫名其秒的文件复制、创建等错误，应该注意检查应用权限，授予其访问所有文件的权限。



### Git

命令行安装程序

```shell
apt update
apt install git
git --version
```



### Obsidian

手机端使用 Obsidian Git 插件必须指定提交名、邮箱，必须使用 HTTP 协议，提供用户名和密码。
