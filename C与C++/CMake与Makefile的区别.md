## CMake与makefile的区别
CMakeLists.txt 与 Makefile 都用于管理和自动化项目的构建过程，但它们之间有一些重要区别：

语法和语言：

CMakeLists.txt 使用 CMake 的自定义语言，其语法更加清晰和可读，具有更高的抽象层次，使得配置和生成过程更加简单。  
Makefile 使用 Make 构建系统的自定义语言，其语法较为低级，需要手动编写规则和依赖关系。  

跨平台性：  
CMake 是一个跨平台的构建工具，可以生成适用于不同操作系统和编译器的构建文件。这意味着 CMakeLists.txt 可以用于多种不同的平台，并支持跨平台的项目构建。  
Makefile 通常是为特定平台和编译器编写的，因此在不同平台上可能需要不同的 Makefile。  

生成器：  
CMake 允许生成不同类型的构建文件，如 Makefile、Visual Studio 项目文件、Ninja 构建文件等。这使得项目可以使用不同的构建系统来构建，而不必更改 CMakeLists.txt。  
Makefile 通常用于生成 Makefile，因此它在不同的构建系统上的可移植性较差。  

可扩展性：  
CMake 提供了许多内置模块和函数，可以用于搜索依赖项、配置项目、生成文档等。它还支持自定义函数和宏的创建，以便进行更高级的构建操作。  
Makefile 的可扩展性较差，需要手动编写复杂的规则和脚本。  

维护和可读性：  
CMakeLists.txt 通常比较容易维护，因为它的语法更加清晰，模块化，允许更容易地组织项目结构。  
Makefile 的语法相对较低级，可能需要更多的维护工作，并且更容易出现错误。  
CMakeLists.txt 更适合管理大型、复杂和跨平台的项目，而 Makefile 通常更适合小型项目或特定于某个平台的项目。  
选择使用哪种工具通常取决于项目的需求和团队的偏好。在实际项目中，有时会将 CMake 与 Makefile 结合使用，以实现更高级的构建和依赖关系管理。  
## 基本使用  
编写 CMake 配置文件 CMakeLists.txt 。  
执行命令 cmake PATH 或者 ccmake PATH 生成 Makefile（ccmake 和 cmake 的区别在于前者提供了一个交互式的界面）。其中， PATH 是 CMakeLists.txt 所在的目录。  
使用 make 命令进行编译。  

### 指定cmake最小版本  
cmake_minimum_required(VERSION 3.4.1)  
在有些情况下，如果 CMakeLists.txt 文件中使用了一些高版本 cmake 特有的一些命令的时候，就需要加上这样一行，提醒用户升级到该版本之后再执行 cmake。  

### 设置项目名称  
project(demo)  
最好写上，它会引入两个变量 demo_BINARY_DIR 和 demo_SOURCE_DIR，同时，cmake 自动定义了两个等价的变量 PROJECT_BINARY_DIR 和 PROJECT_SOURCE_DIR。  
### 设置编译类型  
add_executable(demo demo.cpp) # 生成可执行文件  
add_library(common STATIC util.cpp) # 生成静态库  
add_library(common SHARED util.cpp) # 生成动态库或共享库  
add_library 默认生成是静态库，通过以上命令生成文件名字，  

在 Linux 下是：demo libcommon.a libcommon.so  

在 Windows 下是：demo.exe common.lib common.dll  
### 指定编译包含的源文件  
1、明确指出包含哪些源文件  
add_library(demo demo.cpp test.cpp util.cpp)  
2. 搜索所有的cpp文件  
aux_source_directory(dir VAR) 发现一个目录(dir)下所有的源代码文件并将列表存储在一个变量(VAR)中。  
aux_source_directory(. SRC_LIST) # 搜索当前目录下的所有.cpp文件  
add_library(demo ${SRC_LIST})  
### 设置target需要链接的库  
target_link_libraries( # 目标库 demo # 目标库需要链接的库 ${log-lib} )  
在 Windows 下，系统会根据链接库目录，搜索xxx.lib 文件，Linux 下会搜索 xxx.so 或者 xxx.a 文件，如果都存在会优先链接动态库（so 后缀）。  
指定链接动态库或静态库  
target_link_libraries(demo libface.a) # 链接libface.a  
target_link_libraries(demo libface.so) # 链接libface.so  
2. 指定全路径  
target_link_libraries(demo ${CMAKE_CURRENT_SOURCE_DIR}/libs/libface.a)  
target_link_libraries(demo ${CMAKE_CURRENT_SOURCE_DIR}/libs/libface.so)  
3. 指定链接多个库  
target_link_libraries(demo  
    ${CMAKE_CURRENT_SOURCE_DIR}/libs/libface.a  
    boost_system.a  
    boost_thread  
    pthread)  
### 设置包含的目录  
include_directories(  
    ${CMAKE_CURRENT_SOURCE_DIR}  
    ${CMAKE_CURRENT_BINARY_DIR}  
    ${CMAKE_CURRENT_SOURCE_DIR}/include  
)  
### 设置变量  
1、set 直接设置变量的值  
set(SRC_LIST main.cpp test.cpp)  
add_executable(demo ${SRC_LIST})  
2、 set 追加设置变量的值  

set(SRC_LIST main.cpp)  
set(SRC_LIST ${SRC_LIST} test.cpp)  
add_executable(demo ${SRC_LIST})  
3、常用变量  
PROJECT_SOURCE_DIR：工程的根目录  
PROJECT_BINARY_DIR：运行cmake命令的目录，通常为${PROJECT_SOURCE_DIR}/build  
PROJECT_NAME：返回通过 project 命令定义的项目名称  
CMAKE_CURRENT_SOURCE_DIR：当前处理的 CMakeLists.txt 所在的路径  
CMAKE_CURRENT_BINARY_DIR：target 编译目录  
CMAKE_CURRENT_LIST_DIR：CMakeLists.txt 的完整路径  
EXECUTABLE_OUTPUT_PATH：重新定义目标二进制可执行文件的存放位置  
LIBRARY_OUTPUT_PATH：重新定义目标链接库文件的存放位置  
