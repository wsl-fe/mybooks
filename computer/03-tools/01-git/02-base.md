# Git 基础

## 获取Git仓库
1、`git init` ： 初始化，对项目开始用Git管理

2、`git clone [url]` ： 从现有仓库克隆



## 记录每次更新到仓库

1、`git status` ： 检查当前文件状态



2、`git add` ： 对新文件进行跟踪；把已跟踪的文件放到暂存区；合并时把有冲突的文件标记为已解决状态

`.gitignore`文件 ： 该文件列出要忽略的文件模式

```shell
$ cat .gitignore
*.[oa]
*~
```

> 第一行告诉 Git 忽略所有以 .o 或 .a 结尾的文件。一般这类对象文件和存档文件都是编译过程中出现的，我们用不着跟踪它们的版本。第二行告诉 Git 忽略所有以波浪符（~）结尾的文件，许多文本编辑软件（比如 Emacs）都用这样的文件名保存副本。此外，你可能还需要忽略 log，tmp 或者 pid 目录，以及自动生成的文档等等。要养成一开始就设置好 `.gitignore` 文件的习惯，以免将来误提交这类无用的文件。
> 文件 `.gitignore` 的格式规范如下：
>   ○ 所有空行或者以注释符号 ＃ 开头的行都会被 Git 忽略。
>   ○ 可以使用标准的 glob 模式匹配。
>   ○ 匹配模式最后跟反斜杠（/）说明要忽略的是目录。
>   ○ 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。

所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。星号（*）匹配零个或多个任意字符；[abc] 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；问号（?）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。

3、`git diff` ： 显示具体添加和删除的行
`git diff -cached` ： 看已经暂存起来的文件和上次提交时的快照之间的差异



4、`git commit` ： 提交更新。注：需要先git add。
`git commit -a` ： 自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 git add 步骤



5、`git rm` ： 移除文件
`git rm -f` ： 强制删除文件，如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 -f，以防误删除文件后丢失修改的内容。
`git -rm --cached` ： 移除跟踪但不删除文件



6、`git mv` ： 移动文件，也可以用于改名、



## 查看提交历史
`git log` ： 查看提交历史
```
选项		说明
-p		按补丁格式显示每个更新之间的差异。
-n 	 	显示最近n次的更新
-Un		修改上下文的行数为n行
--word-diff		按 word diff 格式显示差异。
--stat			显示每次更新的文件修改统计信息。
--shortstat		只显示 --stat 中最后的行数修改添加移除统计。
--name-only		仅在提交信息后显示已修改的文件清单。
--name-status	显示新增、修改、删除的文件清单。
--abbrev-commit		仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。
--relative-date		使用较短的相对时间显示（比如，“2 weeks ago”）。
--graph			显示 ASCII 图形表示的分支合并历史。
--pretty		使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。
--oneline		`--pretty=oneline --abbrev-commit` 的简化用法
```



## 撤销操作

1、`git commit --amend` ： 撤销刚才的提交操作，重新提交。此命令可以重新编辑提交说明，但将要提交的文件快照和之前的一样。

2、`git reset HEAD <file>...` ： 取消已经暂存的文件

3、`git checkout -- <file>...` ： 取消对文件的修改



## 远程仓库的使用

1、`git remote`： 查看当前配置有哪些远程仓库
  `-v` ： 显示对应的克隆地址

2、`git remote add [shortname] [url]` ： 添加一个新的远程仓库



**抓取数据**

3、`git fetch [remote-name]` ： 从远程仓库抓取数据到本地
  `fetch` 命令只是将远端的数据拉到本地仓库，并不自动合并到当前工作分支，需要手工合并。

4、`git pull` ： 自动抓取数据下来，然后将远端分支自动合并到本地仓库中当前分支。（常用）



**推送数据**

5、`git push [remote-name] [branch-name]` ： 推送数据到远程仓库。
只有在所克隆的服务器上有写权限，或者同一时刻没有其他人在推数据，这条命令才会如期完成任务。如果在你推数据前，已经有其他人推送了若干更新，那你的推送操作就会被驳回。你必须先把他们的更新抓取到本地，合并到自己的项目中，然后才可以再次推送。



6、`git remote show [remote-name]` ： 查看某个远程仓库的详细信息

7、`git remote rename` ： 修改某个远程仓库在本地的简称

8、`git remote rm` ： 删除对应的远程仓库
	
	
## 打标签

Git 使用的标签有两种类型：轻量级的（lightweight）和含附注的（annotated）。

- 轻量级标签就像是个不会变化的分支，实际上它就是个指向特定提交对象的引用。
- 含附注标签，实际上是存储在仓库中的一个独立对象，它有自身的校验和信息，包含着标签的名字，电子邮件地址和日期，以及标签说明，标签本身也允许使用 GNU Privacy Guard (GPG) 来签署或验证。



1、`git tag`：列出现有标签



**新建标签**

2、`git tag [tag-name]`：创建一个轻量级标签

3、`git tag -a [tag-name]`：创建一个含附注类型的标签

`git tag -a v1.4 -m 'my version 1.4'`：-m 选项指定对应的标签说明

4、`git tag -s`：如果有自己的私钥，还可以用 GPG 来签署标签，只需要把之前的 -a 改为 -s 即可



5、`git show`：查看相应标签的版本信息，并连同显示打标签时的提交对象



6、`git tag -v [tag-name]`：验证已经签署的标签。此命令会调用 GPG 来验证签名，所以你需要有签署者的公钥，存放在 keyring 中，才能验证



7、`git tag -a [tag-name] verify`：在后期对早先的某次提交加注标签。只要在打标签的时候跟上对应提交对象的校验和（或前几位字符）即可



8、`git push origin [tagname]`：分享标签到远端仓库。如果要一次推送所有本地新增的标签上去，可以使用 --tags 选项
