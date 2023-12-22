# Selecting Static or Shared Libraries  

```cmake
option(BUILD_SHARED_LIBS "Build using shared libraries" ON)
```

控制 `add_library()` 的生成库的默认形式是 `SHARED` 还是 `STATIC`  

在windows上设置/不设置位置无关的代码都能成功编译，它的位置在[这里](MathFunctions/CMakeLists.txt#L29-L31)  

```cmake
set_target_properties(SqrtLibrary PROPERTIES
                      POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS}
                      )
```
