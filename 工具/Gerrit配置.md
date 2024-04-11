Gerrit 代码审核工具，丰富的权限管理，具有广泛通用性。这里简单搭建了一套gerrit服务器，用于学习gerrit工具，可以更好的为开源社区贡献代码。
git 客户端下载[链接](https://gitforwindows.org/)
1.已经创建好的帐号有admin dev1 dev2 verify1 verify2 review1 review2

```bash
    admin管理员账户，代码合并
    dev1 dev2开发人员提交代码
    verify1 verify2 用于核查提交的代码能否编译成功（通常去其他集成工具配合）
    review1 review2 审核人
```
2.先登录admin，设置SSH信息
1）点击右侧帐号名称，点击Settings
![在这里插入图片描述](https://img-blog.csdnimg.cn/d70216da83b946ca9fcd642ad85e4b68.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
2）将自己的ssh公钥粘贴进去（ssh 可以通过ssh-genkey生成）
![在这里插入图片描述](https://img-blog.csdnimg.cn/32c877f588c544ed83ca75e3a2721c61.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
这个可以参考这个[链接](https://blog.csdn.net/Artisan_w/article/details/119154234)
3. admin创建工程
点击Project－> Create New Project
![在这里插入图片描述](https://img-blog.csdnimg.cn/d09457d83f1c47e18df98fe208fd7c01.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)

设置工程名称：Test1
![在这里插入图片描述](https://img-blog.csdnimg.cn/801015a3687442c29a2e97fa7e8dd601.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)

在Project－>List中可以看到自己的创建工程

4.创建用户组 dev verify review
People->Create New Group，创建三个组，dev组用于提交代码／verify用于验证代码／review组用于code review

![在这里插入图片描述](https://img-blog.csdnimg.cn/5a4db0a827e6460281ba3f947eb82d2c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
添加用户组结果如下：

![Paste_Image.png](https://img-blog.csdnimg.cn/863b0155e74641f6b437e7f1fc5d8884.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)

5.权限设置
Gerrit权限分配功能很强大，可以自己在Access多多尝试。在Project->access中设置权限，点击edit

![在这里插入图片描述](https://img-blog.csdnimg.cn/1371b20a8d7747de86d933ca8e617fcf.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)


具体的Access配置自己可以尝试，其中Verify CodeReview的配置比较重要。一次完整的流程，需要Verifier CodreReviewer的共同作用才能最终submit代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/c4b994bcc55a491fa754fd510aedf262.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)

6.开发着提交代码
xxx.xxx.xxx.xxx云服务平台地址

    1）git clone ssh://dev1@xxx.xxx.xxx.xxx:29418/Test1
    2）gitdir=$(git rev-parse --git-dir); scp -p -P 29418 dev1@xxx.xxx.xxx.xxx:hooks/commit-msg ${gitdir}/hooks/
    3）touch haha.txt
    4)   git add haha.txt
    5)  git commit -m "first"
    6)  git push origin HEAD:refs/for/master
此时在all->open能够看到一次修改的提交，all->代表已经合并的提交，Abandon代表拒绝的提交

其中状态标志CR代表code review情况，V代表verify情况

Open
review1用户 的Test1界面显示

![在这里插入图片描述](https://img-blog.csdnimg.cn/4cee7dd90e20479895913e89b242a3ba.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)

点击蓝色按钮后，code review通过

![Paste_Image.png](https://img-blog.csdnimg.cn/67e73682a89a45cc951a3011aeae98ba.png)

verify1界面

![在这里插入图片描述](https://img-blog.csdnimg.cn/c8d37a29a3b8445ba23fc7ad432cb97f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)


点击蓝色按钮，V通过

![Paste_Image.png](https://img-blog.csdnimg.cn/c81e99f3180544f58b8a6a09ae949475.png)

当两者通过后，admin的submit功能出现

![在这里插入图片描述](https://img-blog.csdnimg.cn/2a51754113d54d1596e9d4a75acea05b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)

点击完成后，all->open看不到了，all->merged出现本次提交纪录

![Paste_Image.png](https://img-blog.csdnimg.cn/343621f0ed374a0588ef5235922d8b51.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
