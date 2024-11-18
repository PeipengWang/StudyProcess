

# Linux系统调用

```
open() :打开文件
close() :关闭文件
read():从文件读数据
write():向文件写数据
seek():定位文件指针
```

## 扩展

### open 函数

作用：打开文件，返回一个文件描述符（int 类型），用于后续文件操作。
常用模式：
O_RDONLY: 只读打开。
O_WRONLY: 只写打开。
O_RDWR: 读写模式。
O_CREAT: 如果文件不存在则创建。
O_TRUNC: 打开时清空文件内容。

### read 函数扩展

作用：从文件描述符中读取数据。
参数：
src: 文件描述符。
buffer: 数据存储缓冲区。
BUFFER_SIZE: 每次读取的最大字节数。
返回值：
正值：实际读取的字节数。
0：文件末尾。
负值：出错。

### write 函数扩展

作用：向文件描述符写入数据。
参数：
tgt: 目标文件描述符。
buffer: 写入的数据缓冲区。
bytesRead: 要写入的字节数（通常为上次 read 的返回值）。
返回值：
正值：实际写入的字节数。
负值：出错。

### **`close` 函数扩展**

- **作用**：关闭文件描述符，释放资源。
- **注意**：未关闭文件描述符可能导致资源泄露。

## 代码实例

```
#include <fcntl.h>    // 文件控制操作，如 open()、O_RDONLY 等
#include <unistd.h>   // 提供系统调用接口，如 read()、write()、close() 等
#include <stdio.h>    // 标准输入输出库，用于 perror() 和 printf() 等函数
#include <stdlib.h>   // 标准库函数，如 exit()

#define BUFFER_SIZE 1024  // 定义缓冲区大小，用于一次读取/写入的字节数


#define BUFFER_SIZE 1024  // 定义缓冲区大小

void backup_file_with_syscall(const char *source, const char *target) {
    int src = open(source, O_RDONLY);  // 以只读方式打开源文件
    //如果返回值小于 0，说明文件打开失败，可能原因包括文件不存在或权限不足。
    if (src < 0) {
        perror("Failed to open source file");  // 输出错误信息
        exit(EXIT_FAILURE);  // 终止程序，返回失败状态
    }

     // 打开目标文件，以写入方式 (O_WRONLY)，并使用 O_CREAT 和 O_TRUNC
    int tgt = open(target, O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (tgt < 0) {
        perror("Failed to open target file");
        close(src);  // 关闭已打开的源文件，避免资源泄露
        exit(EXIT_FAILURE);
    }

    char buffer[BUFFER_SIZE];  // 定义缓冲区，用于存放读取的数据
    ssize_t bytesRead;         // 记录每次读取的字节数

    // 循环读取源文件内容，直到文件末尾
    while ((bytesRead = read(src, buffer, BUFFER_SIZE)) > 0) {
        write(tgt, buffer, bytesRead);  // 将读取的数据写入目标文件
    }

    close(src);  // 关闭源文件
    close(tgt);  // 关闭目标文件
}
/**
 main 函数
定义源文件路径和目标文件路径。
调用 backup_file_with_syscall 实现备份操作。
使用 printf 提示备份完成。
 */
int main() {
    const char *source = "文件路径\\source.dat";
    const char *target = "文件路径\\target.dat";

    backup_file_with_syscall(source, target);
    printf("File backup completed using system calls.\n");

    return 0;
}

```



## 运行代码

新建一个文件source.dat



![image-20241118214046052](https://raw.githubusercontent.com/PeipengWang/picture/master/homePictimage-20241118214046052.png)

![image-20241118214408502](https://raw.githubusercontent.com/PeipengWang/picture/master/homePictimage-20241118214408502.png)

修改代码中的source.dat的文件路径

例如

修改为：

```
    const char *source = "E:\\FileCopyTest\\source.dat";
    const char *target = "E:\\FileCopyTest\\target.dat";
```

运行代码

会发现多了target.dat文件

![image-20241118214559294](https://raw.githubusercontent.com/PeipengWang/picture/master/homePictimage-20241118214559294.png)

![image-20241118214616363](https://raw.githubusercontent.com/PeipengWang/picture/master/homePictimage-20241118214616363.png)

# C语言库函数

```
#include <stdio.h>   // 标准输入输出库，提供 FILE、fopen、fread、fwrite 等函数
#include <stdlib.h>  // 标准库函数，提供 exit() 和 perror()

#define BUFFER_SIZE 1024  // 定义缓冲区大小，用于一次读取/写入的字节数

void backup_file_with_stdio(const char *source, const char *target) {
    // 使用 fopen() 打开源文件，以二进制只读模式 ("rb") 打开文件
    FILE *src = fopen(source, "rb");
    //fopen 返回 NULL 表示文件打开失败。
    //常见错误原因包括文件不存在或权限不足。
    if (src == NULL) {
        perror("Failed to open source file");  // 输出系统错误信息
        exit(EXIT_FAILURE);  // 程序终止，返回失败状态
    }
    // 使用 fopen() 打开目标文件，以二进制写入模式 ("wb") 打开文件
    FILE *tgt = fopen(target, "wb");
    if (tgt == NULL) {
        perror("Failed to open target file");
        fclose(src);
        exit(EXIT_FAILURE);
    }

    char buffer[BUFFER_SIZE];  // 缓冲区
    size_t bytesRead;

    // 循环读取源文件并写入目标文件
    while ((bytesRead = fread(buffer, 1, BUFFER_SIZE, src)) > 0) {
        fwrite(buffer, 1, bytesRead, tgt);
    }

    fclose(src);  // 关闭源文件
    fclose(tgt);  // 关闭目标文件
}

int main() {
    const char *source = "source.dat";
    const char *target = "target.dat";

    backup_file_with_stdio(source, target);
    printf("File backup completed using C standard I/O.\n");

    return 0;
}

```

