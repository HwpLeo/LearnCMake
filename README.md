# LearnCMake
学习使用CMake


## 入门案例：单个源文件

对于简单的项目，只需要写几行代码就可以了。例如，假设现在我们的项目中只有一个源文件`main.cc`，该程序的用途是计算一个数的指数幂。

```c++
#include <stdio.h>
#include <stdlib.h>
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
    
    if (exponent == 0) {
        return 1;
    }
    
    for(i = 1; i < exponent; ++i){
        result = result * base;
    }
    return result;
}
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


### 编写CMakeList.txt

首先编写`CMakeLists.txt`文件，并保存在`main.cc`源文件同个目录下：

```
# CMake 最低版本号要求
cmake_minimum_required(VERSION 2.8)

# 项目信息
project(Demo1)

# 指定生成目标
add_executable(Demo main.cc)
```

CMakeLists.txt的语法比较简单，由命令、注释和空格组成，其中命令是不区分大小写的。符号`#`后面的内容被认为是注释。命令由命令名称、小括号和参数组成，参数之间使用空格进行间隔。

对于上面的CMakeLists.txt文件，依次出现了几个命令：

1. `cmake_minimum_required`：指定运行此配置文件所需的CMake的最低版本；
2. `project`：参数是`Demo1`，该命令表示项目的名称是`Demo1`。
3. `add_executable`：将名为`main.cc`的源文件编译成一个名称为Demo的可执行文件。

## 多个源文件

### 同一目录，多个源文件

上面的例子只有单个源文件。现在假如把`power`函数单独写进一个名为`MathFunctions.c`的源文件里，使得这个工程变成如下的形式：

```
./Demo2
    |
    +--- main.cc
    |
    +--- MathFunctions.cc
    |
    +--- MathFunctions.h
```

这个时候，CMakeLists.txt文件可以改成如下的形式：

```
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo2)

# 指定生成目标
add_executable(Demo main.cc MathFunctions.cc)
```

唯一的改动只是在`add_executable`命令中增加了一个`MathFunctions.cc`源文件。这样写当然没什么问题，但是如果源文件很多，把所有源文件的名字都加进去僵尸一件烦人的工作。更省事的方法是使用`aux_source_directory`命令，该命令会查找指定目录下的所有源文件，然后将结果存进指定变量名。其语法如下：

```
aux_source_directory(<dir> <variable>)
```

因此，可以修改CMakeLists.txt如下：

```
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)
# 项目信息
project (Demo2)
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)
# 指定生成目标
add_executable(Demo ${DIR_SRCS})
```

这样，CMake会将当前目录所有源文件的文件名赋值给变量`DIR_SRCS`，再指示变量`DIR_SRCS`中的源文件需要编译成一个名称为Demo的可执行文件。

### 多个目录，多个源文件

现在进一步将MathFunctions.h和MathFunctions.cc文件移动到math目录下。

```
./Demo3
    |
    +--- main.cc
    |
    +--- math/
          |
          +--- MathFunctions.cc
          |
          +--- MathFunctions.h
```

对于这种情况，需要分别在项目根目录Demo3和math目录里各编写一个CMakeLists.txt文件。为了方便，我们可以先将math目录里的文件编译成静态库再由main函数调用。
根目录中的CMakeLists.txt：
```
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)
# 项目信息
project (Demo3)
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)
# 添加 math 子目录
add_subdirectory(math)
# 指定生成目标 
add_executable(Demo main.cc)
# 添加链接库
target_link_libraries(Demo MathFunctions)
```

该文件添加了下面的内容：第3行，使用命令`add_subdirectory`指明本项目包含一个子目录math，这样math目录下的CMakeLists.txt文件和源代码也会被处理。第6行，使用命令`target_link_libraries`指明可执行文件main需要连接一个名为MathFunctions的链接库。

子目录中的CMakeLists.txt：

```
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_LIB_SRCS 变量
aux_source_directory(. DIR_LIB_SRCS)
# 生成链接库
add_library (MathFunctions ${DIR_LIB_SRCS})
```

在该文件中使用命令`add_library`将src目录中的源文件编译为静态链接库。

## 自定义编译选项

CMake允许为项目增加编译选项，从而可以根据用户的环境和需求选择最合适的编译方案。

例如，可以将MathFunctions库设为一个可选的库，如果该选项为`ON`，就使用该库定义的数学函数来进行运算。否则就调用标准库中的数学函数库。

### 修改CMakeLists文件

我们要做的第一步是在顶层的CMakeLists.txt文件中添加该选项：

```
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)
# 项目信息
project (Demo4)
# 加入一个配置头文件，用于处理 CMake 对源码的设置
configure_file (
  "${PROJECT_SOURCE_DIR}/config.h.in"
  "${PROJECT_BINARY_DIR}/config.h"
  )
# 是否使用自己的 MathFunctions 库
option (USE_MYMATH
       "Use provided math implementation" ON)
# 是否加入 MathFunctions 库
if (USE_MYMATH)
  include_directories ("${PROJECT_SOURCE_DIR}/math")
  add_subdirectory (math)  
  set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)
# 指定生成目标
add_executable(Demo ${DIR_SRCS})
target_link_libraries (Demo  ${EXTRA_LIBS})
```

// TODO


[CMake入门实战](https://www.hahack.com/codes/cmake/)