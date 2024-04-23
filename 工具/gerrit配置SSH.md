（1）   本地创建SSH key
          ssh-keygen -t rsa -C "your_email@youremail.com"
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/b3d0cbaaf3d84022a085ce8d3be5f307.png)
(2) 成功的话会在~/下生成.ssh文件夹，进去，复制id_rsa.pub文件内容
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/191226b8f781405bbc0477667ac891f9.png)

 (3)添加SSH   Key单击“Add SSH key

![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/2b33c82a8bb849fba57c0f71d40434dd.png)
    ![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/cc386ab402b3451f92fa816dc1bb3f57.png)

   (3)为了验证是否成功，在git bash下输入：

          $ ssh -T git@github.com


​    
 ![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/efbaf49bfa704c91b804120a905ab561.png)

如果是第一次的会提示是否continue，输入yes就会看到：You've successfully authenticated, but GitHub does not provide shell access 。这就表示已成功连上github。

