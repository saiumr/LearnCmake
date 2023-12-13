# Step 1: A Basic Starting Point  

## todo 1~3  

任何项目的最顶部CMakeLists.txt文件必须首先使用 `cmake_minimum_required()` 命令指定最低CMake版本。这将建立策略设置并确保以下CMake函数在兼容的CMake版本中运行。这将建立策略设置并确保以下CMake函数在兼容的CMake版本中运行。  

要启动一个项目，我们使用 `project()` 命令来设置项目名称。每个项目都需要这个调用，并且应该在 cmake_minimum_required() 之后立即调用。正如我们稍后将看到的，这个命令还可以用来指定其他项目级别的信息，比如语言或版本号。  

最后， `add_executable()` 命令告诉CMake使用指定的源代码文件创建一个可执行文件。  

```cmake
cmake_minimum_required(VERSION 3.10)  
project(Tutorial)  
add_executable(Tutorial main.cpp)  
```

## todo 4~6  

CMake有一些特殊的变量，它们要么是在后台创建的，要么是在项目代码中设置的。这些变量中的许多都以 `CMAKE_` 开头。  

在CMake中启用对特定C++标准的支持的一种方法是使用 `CMAKE_CXX_STANDARD` 变量。对于本教程，将 CMakeLists.txt 文件中的 CMAKE_CXX_STANDARD 变量设置为 11 ，将 `CMAKE_CXX_STANDARD_REQUIRED` 设置为 True 。确保在调用 add_executable() 的上面添加 CMAKE_CXX_STANDARD 声明。

```cmake
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

## todo 7~12

让一个在 CMakelists.txt 文件中定义的变量在源代码中也可用。实现此目的的一种方法是使用已配置的头文件。我们创建一个输入文件，其中包含一个或多个要替换的变量。这些变量有特殊的语法，看起来像 `@VAR@` 。然后，我们使用 `configure_file()` 命令将输入文件复制到给定的输出文件，并将这些变量替换为 CMakelists.txt 文件中的当前值 VAR 。  

`VERSION`是用project()命令设定的，这部分要打印发行软件版本号，输入文件`TutorialConfig.h.in`，输出文件`TutorialConfig.h`，输出文件不存在时会自动创建  

由于配置的文件将被写入到项目二进制目录中，因此我们必须将该目录添加到路径列表中以搜索包含文件。这时就要用到target_include_directories()命令。  

```cmake
<PROJECT-NAME>_VERSION_MAJOR
<PROJECT-NAME>_VERSION_MINOR
configure_file(<input_file> <output_file>)
target_include_directories()
```

cmake 构建命令  

```cmake
# 使用MinGW
cmake -S . -B build -G "MinGW Makefiles"
cd build
cmake --build .
# 或者 make
```
