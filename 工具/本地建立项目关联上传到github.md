# 所需指令

```
   1 git init
   2 git add .
   3 git commit -m 'first commit'
   4 git remote add origin git@github.com:PeipengWang/SpringBoot.git
   5 git remot -v
   6 git remote -v
   7 git push -u origin master
```

# 流程

1、先在远程仓库(如github)创建项目,为了避免错误,不要初始化 README, license, 或者gitignore文件 .

2、打开Terminal终端

3、切换到你的本地项目目录

4、初始化本地仓库

```
git init
```

5、添加文件到本地仓库

```
git add .
```

6、提交文件

```
git commit -m "First commit"
```

7、到远程仓库的页面上,复制仓库地址 (笔者以配置好ssh,故复制ssh形式的仓库地址)

```
git@github.com:PeipengWang/SpringBoot.git
```

8、添加远程仓库地址到本地仓库

```
git remote add origin {远程仓库地址}
git remote add origin git@github.com:PeipengWang/SpringBoot.git
```

检查是否关联上了

```
git remote -v
origin  git@github.com:PeipengWang/SpringBoot.git (fetch)
origin  git@github.com:PeipengWang/SpringBoot.git (push)

```


9、push到远程仓库

```
 git push -u origin master
```

到远程仓库找到master分支就能看到代码