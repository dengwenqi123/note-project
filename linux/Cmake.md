## Cmake

### 语法特性介绍

- ### 基本语法格式： 指令(参数1 参数2 ...)

  - 参数使用**括弧**括起
  - 参数之间使用**空格**或分号分开

- 指令是大小写无关的，参数和变量是大小写相关的

```bash
set(HELLO hello.cpp)
add_executable(hello main.cpp hello.cpp)
ADD_EXECUTABLE(hello main.cpp hello.cpp)
```

- 变量使用\${}方式取值，但是在IF控制语句中是直接使用变量名

### 重要指令

- Make_minimum_required - 指定CMake 的最小版本要求
- 语法： cmake_minimum_required(VERSION versionNumber)

```bash
 # CMake最小版本要求为2.8.3
cmake_minimum_required(VERSION2.8.3)
```

Project - 定义工程名称，并可指定工程支持的语言

```bash
# 指定工程名为 HELLOWORLD
project(HELLOWORLD)
```

set - 显示的定义变量

```bash
# 定义SRC 变量，其值为sayhello.cpp hello.cpp
set(SRC sayhello.cpp hello.cpp)

```

Include_directory - 向工程添加多个特定的头文件搜索路径 --> 相当于指定 g++编译器的 -I 参数

语法： **include_directories([AFTER|BEFORE] [SYSTEM] dir1 dir2 ...)**

```bash
# 将 /usr/include/myincludefolder 和 ./include 添加到头文件搜索路径
include_directories(/usr/include/myincludefolder ./include)
```

Link_directories - 向工程添加多个特定的库文件搜索路径 ---> 相当于指定 g++ 编译器-L 参数

语法： link_directories(dir1 dir2)

```bash
# 将 /usr/lib/mylibfolder 和 ./lib 添加到库文件搜索路径
link_directories(/usr/lib/mylibfolder ./lib)
```

Add_library - 生成库文件

语法： add_library(lib name [SHARED|STATIC|MODULE] [EXCLUDE_FROM_ALL]source1 source2 ... sourceN)

```bash
# 通过变量 SRC 生成 libhello.so 共享库
add_library(hello SHARED ${SRC})
```

add_compile_options: 添加编译参数

语法： add_compile_options()

```bash
#添加编译参数 -Wall -std=c++11 -O2
add_compile_options(-Wall -std=c++11 -O2)
```

add_executable: 生成可执行文件

语法: add_executable(exename source1 source2 .... sourceN)

```bash
#编译 main.cpp 生成可执行文件main
add_executable(main main.cpp)
```

Target_link_libraries - 为 target 添加需要链接的共享库 ---> 相同于指定 g++ 编译器 -I 参数

语法L target_link_libraries(target library1<debug>)

```bash
target_link_libraries(main hello)
```

**add_subdirectory** - 向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置

语法：add_subdirectory(source_dir [binary_dir])

```bash
# 添加src 子目录，src中需有一个CMakeLists.txt
ad_subdirectory(src)
```

Aux_source_directory: 发现一个目录下所有的源代码文件并将列表存储在一个变量中，这个指令临时被用来自动构建源文件列表

语法：aux_source_directory(dir VARIABLE)

```bash
#定义SRC变量，其值为当前目录下所有的源代码文件
aux_source_directory(. SRC)
#编译SRC变量所代表的源代码文件，生成main可执行文件
add_executable(main ${SRC})
```

#### INSTALL

- 用于定义安装规则，安装的内容可以包括目标二进制、动态库、静态库以及文件、目录、脚本等。
- 常用的如OpenCV一般情况下安装到系统目录，即 /usr/lib、/usr/bin、/usr/include[Ubuntu系统]

```bash
INSTALL(
	TARGET myrun mylib mystaticlib
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION libstatic
)
```

**参数含义**

1. FILES: 为普通的文本文件
2. PROGRAMS:指的是非目标文件的可执行程序(如脚本文件)
3. DIRECTORY: 为目录





## Reference

[CMakeLists文件install的使用](https://www.it610.com/article/1293833403598184448.htm)

