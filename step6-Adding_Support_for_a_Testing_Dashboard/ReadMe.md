# Adding Support for a Testing Dashboard  

CDash 是一个由 Kitware 公司开发的免费、开源的在线服务，专为 CMake 项目提供集成测试和构建结果的可视化和管理。  

完成练习需要做：  

todo1下将`enable_testing()`替换为  

```cmake
include(CTest)
```

启用测试

同时包含CTestConfig.cmake  

然后构建、安装、启动测试  

```bash
mkdir build
cd build
cmake .. -G "MinGW Makefiles"
cmake --build .
cmake --install .
ctest -D CMakeTutorial  # use CDash
```

checking results: [CDash.org](https://my.cdash.org/index.php?project=CMakeTutorial)
