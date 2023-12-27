# Packing Debug and Release  

在顶层 CMakeLists.txt 文件的开头附近[设置`CMAKE_DEBUG_POSTFIX`](./CMakeLists.txt#L6)，以及教程可执行文件上的[`DEBUG_POSTFIX`属性](./CMakeLists.txt#L42)，这样，生成的调试版本的可执行文件会加上设置的后缀字母'd'  

为 MathFunctions 库添加版本编号。在 MathFunctions/CMakeLists.txt 中，[设置`VERSION`和`SOVERSION`属性](./MathFunctions/CMakeLists.txt#L3-L4)  

VERSION属性用于指定目标的完整版本号。在上述命令中，版本号被设置为1.0.0，可以根据需要将其更改为其他版本号。  

SOVERSION属性用于指定目标的API兼容版本号。在上述命令中，API兼容版本号被设置为1，表示目标具有与主要版本号为1的其他版本兼容的API。这通常用于动态链接库（shared library）的版本管理。  

我们需要安装、调试和发布构建。我们可以使用 `CMAKE_BUILD_TYPE` 来设置配置类型：  

```bash
cd debug
cmake -DCMAKE_BUILD_TYPE=Debug ..
cmake --build .
cd ../release
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
```

现在调试和发布版本都已经完成了，我们可以使用自定义配置文件将这两个版本打包到一个发布版本中。在 Step12 目录中，创建一个名为 `MultiCPackConfig.cmake` 的文件。在此文件中，首先包含由 cmake 可执行文件创建的默认配置文件。  

接下来，使用 `CPACK_INSTALL_CMAKE_PROJECTS` 变量指定要安装的项目。在本例中，我们希望同时安装debug和release。  

```cmake
# MultiCPackConfig.cmake
include("release/CPackConfig.cmake")

set(CPACK_INSTALL_CMAKE_PROJECTS
    "debug;Tutorial;ALL;/"
    "release;Tutorial;ALL;/"
    )
```

从 Step12 根目录，运行 cpack ，指定我们的自定义配置文件和 config 选项：  

```bash
cpack --config MultiCPackConfig.cmake
```
