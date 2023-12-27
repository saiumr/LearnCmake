# Adding Export Configuration  

这一节没有实际的项目演示挺难理解的，可以参阅知乎博文：[CMake教程系列-10-导出配置 By 余朔钰](https://zhuanlan.zhihu.com/p/488700798)  

之前的教程有本地安装以及打包CMake工程，这次我们要添加必要的信息，以便其他CMake工程能使用我们的项目，而且无论是从构建目录、本地安装还是打包的应用程序都可以做到  

本节将导出MathFunctions库供其他使用cmake的项目使用，导出以及显示配置安装 `MathFunctionsTargets.cmake`  

```cmake
# install libs
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
install(TARGETS ${installable_libs} 
        EXPORT MathFunctionsTargets  # 使用EXPORT关键字生成 MathFunctionsTargets.cmake
        DESTINATION lib)
# install include headers
install(FILES MathFunctions.h DESTINATION include)

# 安装导出的 MathFunctionsTargets
install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)

# 下面这步是必须的（Windows上试了没有也能通过编译，并没有警告，不知道为啥）
# 外部使用了MathFunctions库的项目要包含的头文件（编译时和安装时）
target_include_directories(MathFunctions
                           INTERFACE
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                           $<INSTALL_INTERFACE:include>
                           )
```

生成 `MathFunctionsConfig.cmake` （来自于`Config.cmake.in`）以便其他CMake项目使用 `find_package()` 命令搜索到导出的库  

```cmake
# Config.cmake.in的内容
# 此宏将在 configure_file 时自动替换为对路径及内容进行检查的预设cmake宏，用于检查各种预设路径
@PACKAGE_INIT@
include("${CMAKE_CURRENT_LIST_DIR}/MathFunctionsTargets.cmake")
```

```cmake
# 包含cmake模块，以使用 configure_package_config_file() 命令将配置提供的文件
include(CMakePackageConfigHelpers)

# generate the config file that includes the exports
configure_package_config_file(
  # 输入文件
  ${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  # 输出路径/文件
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
  # 安装路径
  INSTALL_DESTINATION "lib/cmake/example"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  )
```

紧接着，`write_basic_package_version_file()` 是下一个此命令写入一个文件，由 `find_package()` 使用，记录所需软件包的版本和兼容性。在这里，我们使用 `Tutorial_VERSION_*` 变量并说它与 `AnyNewerVersion` 兼容，这表示该版本或任何更高版本与请求的版本兼容。  

```cmake
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
  VERSION "${Tutorial_VERSION_MAJOR}.${Tutorial_VERSION_MINOR}"
  COMPATIBILITY AnyNewerVersion
)
```

最后，将上面生成的 `MathFunctionsConfig.cmake` 和 `MathFunctionsConfigVersion.cmake` 都设置为要安装  

```cmake
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake
  DESTINATION lib/cmake/MathFunctions
  )
```

至此，我们已经为我们的项目生成了一个可重定位的CMake配置，可以在项目安装或打包后使用。如果我们希望我们的项目也可以从构建目录中使用，我们只需要在顶层 CMakeLists.txt 的底部添加以下内容：  

```cmake
export(EXPORT MathFunctionsTargets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsTargets.cmake"
)
```

## 补充  

在一个实际的 CMake 项目中，PACKAGE_PREFIX_DIR 的值通常是为了表示包的根目录，这个值会被用于设置包的安装路径、包含目录、库目录等信息，以便在使用这个包的项目中正确地定位包的文件。  

假设你有以下的项目结构：  

```lua
MyLib
|-- CMakeLists.txt
|-- include
|   `-- mylib
|       `-- mylib.h
|-- src
|   `-- mylib.cpp
`-- build
```

其中，`MyLib` 是你的 CMake 项目，包含了一个库 `libmylib.so`，以及头文件 `mylib.h`。现在你希望将这个库和头文件安装到系统，以便其他项目可以轻松使用。  

在 `CMakeLists.txt` 文件中，你可能会使用 `configure_package_config_file` 来生成配置文件 `MyLibConfig.cmake`，并定义 `PACKAGE_PREFIX_DIR`：  

```cmake
# CMakeLists.txt

include(CMakePackageConfigHelpers)

configure_package_config_file(
  "Config.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/MyLibConfig.cmake"
  INSTALL_DESTINATION "lib/cmake/MyLib"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
)

install(
  FILES "${CMAKE_CURRENT_BINARY_DIR}/MyLibConfig.cmake"
  DESTINATION "lib/cmake/MyLib"
)

# ... 其他设置和安装步骤
```

在 Config.cmake.in 文件中，`@PACKAGE_INIT@` 被扩展为：  

```cmake
get_filename_component(PACKAGE_PREFIX_DIR "${CMAKE_CURRENT_LIST_DIR}/../../../" ABSOLUTE)
```

这里，`PACKAGE_PREFIX_DIR` 被设置为 MyLib 项目的上两级目录的绝对路径，即 MyLib 项目的根目录。这个值会在生成的 `MyLibConfig.cmake` 文件中使用，以便在其他项目中正确地设置包含目录、库目录等路径。

假设你将 MyLib 安装到系统，那么在其他项目中使用这个包时，`PACKAGE_PREFIX_DIR` 的值就会指向 MyLib 的安装目录。这样，其他项目就可以使用这个值来正确地引用 MyLib 的头文件和库文件。
