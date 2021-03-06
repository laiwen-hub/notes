# CMake入门

参考：https://www.hahack.com/codes/cmake/



## 常用命令

- project(projectname)：指定工程名称；
- set(var [value])：定义变量；
- message([SEND_ERROR | STATUS | FATAL_ERROR] "message to display")：向终端输出信息；
- aux_source_directory(dir VAR)：查找dir目录下的所有源文件，并将名称保存到VAR变量；
- add_library(target [shared | static] SRC_LIST)：生成静态库文件或共享库文件；
- target_link_libraries(target libother)：链接需要的库文件；
- add_executable：生成可执行文件；
- add_definitions：为源文件的编译添加由-D定义的标志，可用来添加任何标志（例如添加预编译宏），目前被细分为以下命令：
  - add_compile_definitions()：添加预编译宏；
  - include_directories()：添加头文件目录；
  - add_compile_options()：添加其它options；
  
  



## 同一目录，多个源文件

### 目录结构及源码

Demo
├── CMakeLists.txt
├── main.cpp
├── MathFunctions.cpp
└── MathFunctions.h



```cpp
// main.cpp
#include <stdio.h>
#include <stdlib.h>
#include "MathFunctions.h"

int main(int argc, char *argv[])
{
    if (argc < 3){
        printf("Usage: %s base exponent \n", argv[0]);
        return 1;
    }
    double base = atof(argv[1]);
    int exponent = atoi(argv[2]);
    double result = power(base, exponent);
    printf("%g ^ %d is %g\n", base, exponent, result);
    return 0;
}
```



```cpp
// MathFunctions.cpp
/**
 * power - Calculate the power of number.
 * @param base: Base value.
 * @param exponent: Exponent value.
 *
 * @return base raised to the power exponent.
 */
double power(double base, int exponent)
{
    int result = base;
    int i;

    if (exponent == 0) 
        return 1;

    for(i = 1; i < exponent; ++i)
        result = result * base;

    return result;
}
```



```cpp
// MathFunctions.h
#ifndef POWER_H
#define POWER_H

extern double power(double base, int exponent);

#endif
```



### CMake1：逐个添加源文件

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo)

# 指定生成目标
add_executable(Demo main.cpp MathFunctions.cpp)
```

CMakeLists.txt 的语法比较简单，由命令、注释和空格组成，其中**命令是不区分大小写**的。命令由命令名称、小括号和参数组成，参数之间使用空格进行间隔。

对于上面的 CMakeLists.txt 文件，依次出现了几个命令：

- cmake_minimum_required：指定运行此配置文件所需的 CMake 的最低版本；
- project：参数值是 Demo，该命令表示项目的名称是 Demo ；
- add_executable： 将源文件（main.cpp、MathFunctions.cpp）编译成一个名称为Demo的可执行文件。



### CMake2：自动获取所有源文件

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo)

# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS) # +++

# 指定生成目标
add_executable(Demo ${DIR_SRCS})
```

- aux_source_directory：查找指定目录下的额所有源文件，并将结果保存在变量中。



## 多个目录，多个源文件

### 目录结构

Demo
├── CMakeLists.txt
├── main.cpp
└── math
    ├── MathFunctions.cpp
    └── MathFunctions.h



### CMake1：手动包含子目录中的文件

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo)

# 添加头文件目录
include_directories(./math) # +++

# 查找目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(./math DIR_SRCS) # +++
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(Demo ${DIR_SRCS})
```

- include_directories：添加头文件目录。

对于子目录中文件比较少的情况，直接添加比较方便。



### CMake2：根目录和子目录各添加一个CMakeLists

根目录的CMakeLists.txt：

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo)

# 添加头文件目录
include_directories(math) # +++

# 添加子目录
add_subdirectory(math) # +++

# 查找目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(Demo ${DIR_SRCS})

# 添加链接库
target_link_libraries(Demo MathFunctions) # +++
```



子目录的CMakeLists.txt：

```cmake
# 查找目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_LIB_SRCS)

# 生成链接库
add_library (MathFunctions ${DIR_LIB_SRCS})          # 生成.a
# add_library (MathFunctions SHARED ${DIR_LIB_SRCS}) # 生成.so
```

- include_directories：添加头文件目录；
- add_subdirectory：添加子目录，子目录中的CMakeLists.txt 文件和源代码也会被处理；

- target_link_libraries：指明可执行文件需要连接一个名为MathFunctions的链接库；
- add_library：将源文件编译为静态链接库。



编译之后的build文件夹目录结构如下：

Demo
├── CMakeCache.txt
├── CMakeFiles (文件夹内容省略)
├── cmake_install.cmake
├── Demo
├── Makefile
└── math
    ├── CMakeFiles (文件夹内容省略)
    ├── cmake_install.cmake
    ├── **libMathFunctions.a**
    └── Makefile

可以看到生成了名为MathFunctions的静态链接库。



## 为工程添加测试

### CMake1：手动添加测试

CMake 提供了一个称为 CTest 的测试工具。

只需要在CMakeLists.txt中调用一系列的add_test命令，即可添加测试。

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo)

# 添加头文件目录
include_directories(math)

# 添加子目录
add_subdirectory(math)

# 查找目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(Demo ${DIR_SRCS})

# 添加链接库
target_link_libraries(Demo MathFunctions)

##########################

# 启用测试
enable_testing()

# 测试1：程序是否成功运行，并return 0
add_test (test_run Demo 5 2)

# 测试2：当输入参数不符合规定的格式时（例如仅输入了./Demo，没有输入计算参数），打印的提示是否正确
add_test (test_usage Demo)
set_tests_properties (test_usage PROPERTIES PASS_REGULAR_EXPRESSION "Usage: .* base exponent")

# 测试3：5的平方计算是否正确
add_test (test_5_2 Demo 5 2)
set_tests_properties (test_5_2 PROPERTIES PASS_REGULAR_EXPRESSION "is 25")

# 测试4：2的10次方计算是否正确
add_test (test_2_10 Demo 2 10)
set_tests_properties (test_2_10 PROPERTIES PASS_REGULAR_EXPRESSION "is 1024")
```

- set_tests_properties(test1 PROPERTIES prop1 value1)：测试test输出的property，其中的prop1为`PASS_REGULAR_EXPRESSION`时，表示测试输出是否包含后面跟着的字符串。



执行：

1. cmake ..
2. make
3. make test



输出：

```shell
$ make && make test
-- Configuring done
-- Generating done
-- Build files have been written to: /home/sunny/5programtest/LearningMakefile/build
[ 50%] Built target MathFunctions
[100%] Built target Demo
Running tests...
Test project /home/sunny/5programtest/LearningMakefile/build
    Start 1: test_run
1/4 Test #1: test_run .........................   Passed    0.00 sec
    Start 2: test_usage
2/4 Test #2: test_usage .......................   Passed    0.00 sec
    Start 3: test_5_
3/4 Test #3: test_5_2 .........................   Passed    0.00 sec
    Start 4: test_2_10
4/4 Test #4: test_2_10 ........................   Passed    0.00 sec

100% tests passed, 0 tests failed out of 4

Total Test time (real) =   0.00 sec
```



### CMake2：宏定义简化测试

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo)

# 添加头文件目录
include_directories(math)

# 添加子目录
add_subdirectory(math)

# 查找目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(Demo ${DIR_SRCS})

# 添加链接库
target_link_libraries(Demo MathFunctions)

##########################

# 启用测试
enable_testing()

# 定义一个宏，用来简化测试工作
macro (do_test arg1 arg2 result)
  add_test (test_${arg1}_${arg2} Demo ${arg1} ${arg2})
  set_tests_properties (test_${arg1}_${arg2} PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro (do_test)
 
# 使用该宏进行一系列的数据测试
do_test(5 2 "is 25")
do_test(10 5 "is 100000")
do_test(2 10 "is 1024")
```



## 遇到的问题及解决方法

### 编译依赖

#### 问题背景

在编译Sony ToF SDK的时候，发现由于调用的动态库之间有相互依赖关系，在make clean之后，通常需要多次make才能编译成功。

简化后的工程如下：

```
.
├── CMakeLists.txt
├── lib
│   ├── libPolynomialFunc.so
│   └── libPowerFunc.so
├── main.cpp
└── math
    ├── CMakeLists.txt
    ├── PolynomialFunctions.cpp
    ├── PolynomialFunctions.h
    └── power
        ├── CMakeLists.txt
        ├── MathFunctions.cpp
        └── MathFunctions.h
```



- MathFunctions.cpp编译生成libPowerFunc.so；

- PolynomialFunctions.cpp编译生成libPolynomialFunc.so，依赖libPowerFunc.so；

- main.cpp编译生成可执行文件Demo，依赖libPolynomialFunc.so；
- 编译生成的.so文件，都放在lib目录下。



三个CMakeLists.txt如下：

1. main.cpp的CMakeLists.txt

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
set(PROJECT_NAME Demo)
project(${PROJECT_NAME})

message(STATUS "The project is ${PROJECT_NAME}")
message(STATUS "The project dir is ${PROJECT_SOURCE_DIR}")

# 添加头文件目录
include_directories(math)

# 添加子目录
add_subdirectory(math)

# 查找目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(${PROJECT_NAME} ${DIR_SRCS})

# 添加链接库
target_link_libraries(
    ${PROJECT_NAME} 
    ${PROJECT_SOURCE_DIR}/lib/libPolynomialFunc.so
)
```



2. PolynomialFunctions.cpp的CMakeLists.txt

```cmake
set(PROJECT_NAME PolynomialFunc)
project(${PROJECT_NAME})

message(STATUS "The project is ${PROJECT_NAME}")
message(STATUS "The project dir is ${PROJECT_SOURCE_DIR}")

# 添加头文件目录
include_directories(power)

# 添加子目录
add_subdirectory(power)

set(LIBRARY_OUTPUT_PATH  ${PROJECT_SOURCE_DIR}/../lib)

# 查找目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_LIB_SRCS)

# 生成链接库
add_library(${PROJECT_NAME} SHARED ${DIR_LIB_SRCS})

# 添加链接库
target_link_libraries(
    ${PROJECT_NAME} 
    ${PROJECT_SOURCE_DIR}/../lib/libPowerFunc.so
)
```



3. MathFunctions.cpp的CMakeLists.txt

```cmake
set(PROJECT_NAME PowerFunc)
project(${PROJECT_NAME})

message(STATUS "The project is ${PROJECT_NAME}")
message(STATUS "The project dir is ${PROJECT_SOURCE_DIR}")

set(LIBRARY_OUTPUT_PATH  ${PROJECT_SOURCE_DIR}/../../lib)

# 查找目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_LIB_SRCS)

# 生成链接库
add_library(${PROJECT_NAME} SHARED ${DIR_LIB_SRCS})
```



#### 现象

make的时候，由于没有按照依赖来编译 ，所以需要编译3次，才正常。

```shell
sunny:build$ cmake ..
-- The C compiler identification is GNU 5.4.0
-- The CXX compiler identification is GNU 5.4.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- The project is Demo
-- The project dir is /home/sunny/5programtest/LearningMakefile
-- The project is PolynomialFunc
-- The project dir is /home/sunny/5programtest/LearningMakefile/math
-- The project is PowerFunc
-- The project dir is /home/sunny/5programtest/LearningMakefile/math/power
-- Configuring done
-- Generating done
-- Build files have been written to: /home/sunny/5programtest/LearningMakefile/build


sunny:build$ make -j8
Scanning dependencies of target Demo
Scanning dependencies of target PowerFunc
Scanning dependencies of target PolynomialFunc
make[2]: *** No rule to make target '../lib/libPolynomialFunc.so', needed by 'Demo'。 停止。
make[2]: *** 正在等待未完成的任务....
[ 16%] Building CXX object CMakeFiles/Demo.dir/main.cpp.o
[ 50%] Building CXX object math/power/CMakeFiles/PowerFunc.dir/MathFunctions.cpp.o
[ 50%] Building CXX object math/CMakeFiles/PolynomialFunc.dir/PolynomialFunctions.cpp.o
CMakeFiles/Makefile2:76: recipe for target 'CMakeFiles/Demo.dir/all' failed
make[1]: *** [CMakeFiles/Demo.dir/all] Error 2
make[1]: *** 正在等待未完成的任务....
[ 66%] Linking CXX shared library ../../../lib/libPowerFunc.so
[ 83%] Linking CXX shared library ../../lib/libPolynomialFunc.so
../../math/../lib/libPowerFunc.so:无法识别文件: 文件被截断
[ 83%] Built target PowerFunc
collect2: error: ld returned 1 exit status
math/CMakeFiles/PolynomialFunc.dir/build.make:84: recipe for target '../lib/libPolynomialFunc.so' failed
make[2]: *** [../lib/libPolynomialFunc.so] Error 1
CMakeFiles/Makefile2:128: recipe for target 'math/CMakeFiles/PolynomialFunc.dir/all' failed
make[1]: *** [math/CMakeFiles/PolynomialFunc.dir/all] Error 2
Makefile:83: recipe for target 'all' failed
make: *** [all] Error 2


sunny:build$ make -j8
make[2]: *** No rule to make target '../lib/libPolynomialFunc.so', needed by 'Demo'。 停止。
CMakeFiles/Makefile2:76: recipe for target 'CMakeFiles/Demo.dir/all' failed
make[1]: *** [CMakeFiles/Demo.dir/all] Error 2
make[1]: *** 正在等待未完成的任务....
[ 16%] Linking CXX shared library ../../lib/libPolynomialFunc.so
[ 50%] Built target PowerFunc
[ 66%] Built target PolynomialFunc
Makefile:83: recipe for target 'all' failed
make: *** [all] Error 2


sunny:build$ make -j8
[ 33%] Built target PolynomialFunc
[ 83%] Linking CXX executable Demo
[ 83%] Built target PowerFunc
[100%] Built target Demo

```



#### 解决方法

##### 方法一（add_dependencies）

在CMakeLists.txt中增加对依赖的描述：

1. main.cpp的CMakeLists.txt

```diff
# 省略

# 指定生成目标
add_executable(${PROJECT_NAME} ${DIR_SRCS})

+ # 添加依赖
+ add_dependencies(${PROJECT_NAME} PolynomialFunc)

# 添加链接库
target_link_libraries(
    ${PROJECT_NAME} 
    ${PROJECT_SOURCE_DIR}/lib/libPolynomialFunc.so
)
```



2. PolynomialFunctions.cpp的CMakeLists.txt：

```diff
# 省略

# 生成链接库
add_library(${PROJECT_NAME} SHARED ${DIR_LIB_SRCS})

+ # 添加依赖
+ add_dependencies(${PROJECT_NAME} PowerFunc)

# 添加链接库
target_link_libraries(
    ${PROJECT_NAME} 
    ${PROJECT_SOURCE_DIR}/../lib/libPowerFunc.so
)
```



##### 方法二（链接项目名称）

```
target_link_libraries(
    ${PROJECT_NAME} 
    ${PROJECT_SOURCE_DIR}/../lib/libPowerFunc.so
)
```

在使用`target_link_libraries`的时候，第二个参数使用项目名称，而不是具体的库路径。

1. main.cpp的CMakeLists.txt

```diff
# 省略

# 添加链接库
target_link_libraries(
    ${PROJECT_NAME} 
-   ${PROJECT_SOURCE_DIR}/lib/libPolynomialFunc.so
+   polynomialFunc
)
```



2. PolynomialFunctions.cpp的CMakeLists.txt：

```diff
# 省略

# 添加链接库
target_link_libraries(
    ${PROJECT_NAME} 
-   ${PROJECT_SOURCE_DIR}/../lib/libPowerFunc.so
+   PowerFunc
)
```



最终的工程，参考`./appendix/CMake入门_编译依赖`。



### 调用OpenCV库

```cmake
find_package(OpenCV REQUIRED)

target_link_libraries(
    ${PROJECT_NAME}
    ${OpenCV_LIBS}
)
```



### 指定C++11编译标准

在编译结构光源码的时候，报错：

```
/home/sunny/3GitRepository/sl_StructuredLight/sl_git/src/common/errno.cc:12:1: error: ‘thread_local’ does not name a type
 thread_local static int gErrno = 0;
 ^
/home/sunny/3GitRepository/sl_StructuredLight/sl_git/src/common/errno.cc:12:1: note: C++11 ‘thread_local’ only available with -std=c++11 or -std=gnu++11
```



在CMakeLists.txt中添加以下内容：

```cmake
# 设置指定的C++编译器版本
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# 指定为C++11 版本
set(CMAKE_CXX_STANDARD 11)
```



参考：https://cloud.tencent.com/developer/article/1741243



### 解决SSE优化指令集编译错误

在编译结构光源码的时候，报错：

```
/home/sunny/3GitRepository/sl_StructuredLight/sl_git/include/NEON_2_SSE.h:6531:19: error: ‘__builtin_ia32_pblendw128’ needs isa option -m32 -msse4.1
     __m128i low = _mm_blend_epi16(zero, a, 0x55); // 0b1010101
```

该错误的原因是编译的时候没有指定对应的SSE选项。

在文档（ [Intel官方指令集向导](https://software.intel.com/sites/landingpage/IntrinsicsGuide/)）中找到出错指令对应的SSE指令集，然后添加到CMake的编译选项中。

例如这个报错中的指令为`_mm_blend_epi16`，在网页中搜索：

![CMake_SSE指令集查询](picture/CMake_SSE指令集查询.png)



在CMakeLists.txt中添加如下内容即可：

```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse4.1")
```

