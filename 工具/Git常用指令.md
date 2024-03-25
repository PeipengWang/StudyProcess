# Git常用的指令
OMMP提交代码的流程
0、配置：
git config --list 查看当前配置
git congig --global user.name user
这个会显示你的提交到git的名字
格式：git config [–local|–global|–system] –unset section.key
格式：git config [–local|–global|–system] -l
查看仓库级的config，命令：git config –local -l
查看全局级的config，命令：git config –global -l
查看系统级的config，命令：git config –system -l
查看当前生效的配置，命令：git config -l，这个时候会显示最终三个配置文件计算后的配置信息

```sql
git config --global uesr.name 张三00111111
git config --global user.email "aaa@163.com.cn"
```

配置秘钥：

```sql
cd ~/.ssh
ssh-keygen -t rsa -C “aaa@164.com.cn”
```

生成的公钥给git工具这里不详细说
1、拉取最新代码

```sql
git pull
```

2、修改代码
确认修改后的文件
git status 
3、将修改的代码提交到暂存区

```sql
git add .
git restore --staged . 
```

4、提交到远程仓库

```sql
 git commit -m '提交内容注解'
```

回滚：

```sql
git reset
git reset --soft HEAD^
^也可以~数字表示
HEAD~0 表示当前版本
HEAD~1 上一个版本
HEAD^2 上上一个版本
HEAD^3 上上上一个版本
```

5、提交代码

```sql
git push origin HEAD:refs/for/master
```

origin HEAD:refs/for/master是这个主分支的意思，也可以不写则默认
6、没有changId
执行git log查看自己的这笔提交中是否有"Change-Id:"这样的字符，
如果没有，则在当前代码的根目录下执行：

```sql
scp -p -P 29418 aaa@163.com.cn:hooks/commit-msg  .git/hooks/
```

然后再git commit --amend 可以不修改任何东西，保存退出。
这时git log就可以看到Change Id了
7、删除未追踪的文件

```sql
git clean [-d] [-f] [-i] [-n] [-q] [-e ] [-x | -X] [--] 
```

-d 删除未跟踪目录以及目录下的文件，如果目录下包含其他git仓库文件，并不会删除（-dff可以删除）。
-f 如果 git cofig 下的 clean.requireForce 为true，那么clean操作需要-f(--force)来强制执行。
-i 进入交互模式
-n查看将要被删除的文件，并不实际删除文件
8、隐藏恢复修改代码

```sql
git stash隐藏修改的代码
git stash pop隐藏的文本修复
```

git stash #把所有没有提交的修改暂存到stash里面。可用git stash pop回复。
9、比较当前代码与上一次代码的修改文件差异
git diff --name-only HEAD~ HEAD > changes.txt
10、 放弃本地修改 强制更新

```sql
git fetch --all
git reset --hard OMM
```

11、获取完整commit id（如：bb4f92a7d4cbafb67d259edea5a1fa2dd6b4cc7a）

```sql
git rev-parse HEAD
```

获取short commit id（如：bb4f92a）

```sql
git rev-parse --short HEAD
```

获取最新一次已经提交但是未合入的分支
shortCommit=$(git ls-remote | awk '{print $2}' | sed 's/\// /g' | sort -n -k4 | tail -n 1 | sed 's/ /\//g')
git pull "代码的ssh连接" ${shortCommit}
12、 回退版本
确认时间点

```sql
git log --oneline --before '12-23-2020'
```

查看id

```sql
git reset --hard id
```

git fetch是将远程主机的最新内容拉到本地，用户在检查了以后决定是否合并到工作本机分支中。
其他：

```sql
git show --pretty=format: --name-only $GERRIT_PATCHSET_REVISION>codealldiff;
git show --diff-filter=d --pretty=format: --name-only $GERRIT_PATCHSET_REVISION>codediff;
git show --diff-filter=cdmr --pretty=format: --name-only $GERRIT_PATCHSET_REVISION>codeadd;
git diff --name-only 
```
