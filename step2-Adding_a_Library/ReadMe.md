# Step 2: Adding a Library  

## todo1~6  

要在CMake中添加一个库，请使用 `add_library()` 命令并指定哪些源文件应构成该库。  

我们可以用一个或多个子目录来组织项目，而不是将所有源文件放在一个目录中。在本例中，我们将专门为我们的库创建一个目录。在这里，我们可以**添加一个新的 CMakeLists.txt 文件和一个或多个源文件**。在顶层的 CMakeLists.txt 文件中，我们将使用 `add_subdirectory()` 命令来将编译器添加到构建中。  

一旦库被创建，它就通过 `target_include_directories()` 和 `target_link_libraries()` 连接到我们的可执行目标。  

```cmake
add_library(<lib_name> [STATIC|SHARED|MODULE] <src>)
add_subdirectory()
target_include_directories()
target_link_libraries(<name> [PUBLIC|PRIVATE|INTERFACE] <lib_name>)
PROJECT_SOURCE_DIR
```

## todo7~14  

我们在前面将标准库里面的sqrt函数替换成了自己写的函数，在大型项目中我们可能会根据需要使用不同的函数，这时候可以利用cmake中的 `option()` 设置变量用以控制使用自定义的函数还是标准库的函数  

```cmake
if()
option(<variable> "<help_text>" [value])
target_compile_definitions()
endif()
```

`if()` 用于检查设置的变量是 `ON` 还是 `OFF`，`target_compile_definitions()` 用于添加一个编译定义  

[todo12~13](./MathFunctions/CMakeLists.txt#L13-L25)做的事，是为了不使用自定义的函数时就不编译mysqrt.cxx，[todo1](./MathFunctions/CMakeLists.txt#L3)原本是这样的  

```cmake
add_library(MathFunctions MathFunctions.cxx mysqrt.cxx)
```

这样有个问题，无论是否使用自定义的函数都会编译库文件mysqrt.cxx  

设置的变量都在cache中，我们能在使用cmake构建后生成的 `CMakeCache.txt` 文件中找到它，外部改变变量可以使用下面的命令重新构建  

```cmake
# in build directory
cmake .. -DUSE_MYMATH=OFF
# use MinGW
cmake .. -G "MinGW Makefiles" -DUSE_MYMATH=OFF
```

## 补充  

在CMake中，接口属性是一种特殊类型的属性，它们不直接应用于目标本身，而是传递给依赖于该目标的其他目标。这使得你可以在项目中定义一些属性，并通过依赖关系将它们传递给其他目标，而不必直接链接这些目标。

接口属性常常用于库的头文件路径、编译定义、编译选项等。这使得你可以轻松地共享这些设置，而不必在每个目标中都重新设置一遍。

下面是一个简单的例子，说明了如何在CMake中使用接口属性：

```cmake
# CMakeLists.txt for LibraryA  
add_library(LibraryA INTERFACE)
target_include_directories(LibraryA INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_compile_definitions(LibraryA INTERFACE USE_FEATURE_X)

# CMakeLists.txt for ExecutableB
add_executable(ExecutableB main.cpp)
target_link_libraries(ExecutableB PRIVATE LibraryA)
```

在这个例子中：

LibraryA 是一个接口库，它声明了接口属性：

通过 target_include_directories 设置了头文件路径。
通过 target_compile_definitions 定义了编译期间使用的宏（宏 USE_FEATURE_X）。
ExecutableB 是一个可执行文件，通过 target_link_libraries 命令将 LibraryA 添加为其依赖。因为 LibraryA 是一个接口库，所以 ExecutableB 可以访问到 LibraryA 的接口属性，包括头文件路径和编译定义。

通过这种方式，你可以将一些共享的设置、宏定义等信息定义在接口库中，而不必在每个可执行文件或库中重复设置。这使得项目的维护和管理更为方便。  

## INTERFACE实例  

当使用 CMake 和 C++ 时，你可能会有一个项目结构如下：

```lua
project_root/
|-- CMakeLists.txt
|-- include/
|   |-- LibraryA/
|       |-- feature.h
|-- src/
|   |-- LibraryA/
|       |-- feature.cpp
|-- main.cpp
```

假设 feature.h 包含以下内容：

```cpp
// feature.h
#pragma once
```

void featureFunction();
而 feature.cpp 包含以下内容：

```cpp
// feature.cpp
#include <iostream>

void featureFunction() {
    std::cout << "Feature function implementation." << std::endl;
}
```

然后，你的 main.cpp 文件可能如下：

```cpp
// main.cpp
#include "LibraryA/feature.h"

int main() {
    featureFunction();
    return 0;
}
```

接下来，你的 CMakeLists.txt 文件可以看起来像这样：

```cmake
# CMakeLists.txt

cmake_minimum_required(VERSION 3.12)
project(MyProject)

# LibraryA
add_library(LibraryA INTERFACE)
target_include_directories(LibraryA INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)

# Add source files to LibraryA (assuming feature.cpp is in src/LibraryA)
target_sources(LibraryA INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/src/LibraryA/feature.cpp)

# Executable
add_executable(MyApp main.cpp)

# Link the executable with LibraryA
target_link_libraries(MyApp PRIVATE LibraryA)
```

在这个例子中：

LibraryA 是一个 INTERFACE 库，它的目的是传递接口属性，不包含实际的二进制文件。  

feature.cpp 文件被添加到 LibraryA 的源文件中，以便其他目标可以访问 featureFunction() 的实现。  

MyApp 是一个可执行文件，通过 target_link_libraries() 将 LibraryA 添加为依赖。这使得 MyApp 可以使用 featureFunction() 函数而无需直接链接到 LibraryA 的二进制文件。  

请注意，这个示例中的项目结构和 CMake 配置是一种常见的组织方式，但实际的项目可能有不同的需求和组织结构。  

所以，实际的编译过程仍然会涉及到 feature.cpp 的编译，但是 LibraryA 本身并不会生成库文件。INTERFACE 库的主要作用是传递接口信息，以便在整个项目中共享和继承。  

使用 INTERFACE 库的方式主要是为了提供一种在 CMake 项目中共享接口属性的机制，以及更好地组织项目的结构。虽然这样做可能会显得繁琐，但它有一些优点：

- 接口共享： INTERFACE 库允许你定义一组接口属性，如头文件路径、编译定义等，而不需要生成实际的库文件。这使得这些接口属性能够被依赖于该库的其他目标继承，从而实现接口的共享。

- 组织结构： 使用 INTERFACE 库有助于更好地组织项目结构。通过将接口属性定义在独立的库中，你可以更清晰地区分实现细节和接口定义。

- 可维护性： 这种方式提高了项目的可维护性。当你需要共享一组属性时，只需将其添加到 INTERFACE 库，而不必在每个目标中都重复设置。

- 依赖管理： 如果你的项目是一个库，可能会被其他项目使用，INTERFACE 库是一种良好的方式来定义公共接口，确保其他项目可以正确地使用你的库。

虽然在小型项目中可能感觉不必要，但在大型项目或多模块项目中，这种组织结构和接口共享的机制通常是非常有用的。当项目规模扩大时，这种方式可以帮助你更好地管理和维护代码。  
