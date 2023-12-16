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

[todo12~13](./MathFunctions/CMakeLists.txt#L17-L24)做的事，是为了不使用自定义的函数时就不编译mysqrt.cxx，[todo1](./MathFunctions/CMakeLists.txt#L3)原本是这样的  

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