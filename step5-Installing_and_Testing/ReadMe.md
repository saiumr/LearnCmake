# Installing and Testing  

通常，仅仅构建一个可执行文件是不够的，它还应该是可安装的。使用CMake，我们可以使用 `install()` 命令指定安装规则。在CMake中支持构建的本地安装通常很简单，只需指定**安装位置**以及要**安装的目标和文件**。  

安装 Tutorial 可执行文件和 MathFunctions 库到 `bin` 目录和 `lib` 目录，它们的头文件安装到 `include` 目录。  

## todo1~4 install()  

```cmake
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)  # if 存在名称为 SqrtLibrary 的 TARGET
  list(APPEND installable_libs SqrtLibrary)
endif()

install(TARGETS ${installable_libs} DESTINATION lib)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
```

## 编译和安装

build完成后，在build目录下执行（在cmake 3.15 之前要用老命令 `make install` ）  

```bash
cmake --install .
```

多配置工具使用 `--config` 指定配置  

```bash
cmake --install . --config Debug
cmake --install . --config Release
```

我们使用IDE编译项目，一般会生成一个名为 `install` 的目标，使用下面的命令可以做到与IDE一样的事  

```bash
cmake --build . --target install --config Debug
```

cmake根目录变量 `CMAKE_INSTALL_PREFIX` 控制安装文件所在的根目录，也可以使用 `--prefix` 选项可以在命令中设置根目录  

```bash
cmake --install . --prefix "/home/myuser/installdir"
```

默认会从系统根目录创建install命令指定的目录，除非override path prefix  
作为示例，将本次学习目录step5..作为安装目录，设置如下  

```cmake
set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_SOURCE_DIR})
```

## todo5~9 add_test()  

下面为我们的项目添加测试，使用 `ctest` 很方便  

```cmake
enable_testing()                         # 启用测试
add_test(NAME Runs COMMAND Tutorial 25)  # 添加一个顶层测试而不验证，确保程序能正常运行
function()                               # 定义一个函数，写测试用例更方便
set_tests_properties()                   # 设定测试属性
ctest
```

## 运行测试用例  

在build目录构建完成后  

```bash
# 列出所有可用的测试，但并不执行它们
ctest -N
# 单独运行某个测试用例
ctest -R [test_name]
# 在详细模式下运行测试，并显示更多的输出信息
ctest -VV
# 两个选项常常一块使用
# 这会显示测试的名称并提供详细的输出，但并不实际运行测试
# 对于诊断测试失败或者查看测试执行的详细信息非常有帮助
ctest -N -VV
# 多配置生成器要指定配置（如Visual Studio会生成 Debug 和 Release 版本）
ctest -C Debig -VV
```
