# pthread
1、新建一个demoC  
CMakeLists.txt加入依赖  
target_link_libraries(demoC -pthread)  
2、依赖  
#include <pthread.h>  
#include <stdio.h>  
3、定义头文件  
pthread_test.h  
 头文件中定义函数  
```  
#include <pthread.h>  
#include <stdio.h>  
void *thread_function(void *arg);  
/**
 * 创建新的线程
 * @return
 */
int create_new_thread(void);
```
 4、实现头文件中定义的函数  
 pthread_test.c
```
#include "pthread_test.h"
void *thread_function(void *arg) {
    printf("Hello from the new thread!\n");
    return NULL;
}
/**
 * 测试创建一个线程  
 * @return
 */
int create_new_thread(void){
    pthread_t thread_id;
    int result = pthread_create(&thread_id, NULL, thread_function, NULL);
    if (result != 0) {
        perror("Thread creation failed");
        return 1;
    }
    printf("Main thread: Created a new thread with ID %lu\n", thread_id);
    pthread_join(thread_id, NULL); // Wait for the new thread to finish
    printf("Main thread: New thread has terminated\n");
    return 0;
}

```
5、运行  
引入自定义头文件  
实现main  
```
#include "pthread_test.h"
int main() {
    create_new_thread();
    return 0;
}
```
运行结果
```
Main thread: Created a new thread with ID 139967580378880
Hello from the new thread!
Main thread: New thread has terminated
```


