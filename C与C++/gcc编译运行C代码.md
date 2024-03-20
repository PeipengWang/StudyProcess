# gcc编译c语言并运行流程
查看gcc编译器是否存在  
gcc -v

第一个Hello，World  
```
#include <stdio.h>
 
int main()
{
   /* 我的第一个 C 程序 */
   printf("Hello, World! \n");
   
   return 0;
}
```
编译并执行  
$ gcc hello.c  
$ ./a.out  
Hello, World!  

请确保您的路径中已包含 gcc 编译器，并确保在包含源文件 hello.c 的目录中运行它。  
如果是多个 c 代码的源码文件，编译方法如下：  
$ gcc test1.c test2.c -o main.out  
$ ./main.out  