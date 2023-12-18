# Adding Usage Requirements for a Library  

## todo1~3  

一个目标参数的`使用要求`可以更好地控制库或可执行文件的链接和包含行，同时还能更好地控制 CMake 中目标的传递属性。主要利用使用要求的命令有：  

```cmake
target_compile_definitions()
target_compile_options()
target_include_directories()
target_link_directories()
target_link_options()
target_precompile_headers()
target_sources()
```

这一节的内容就是重构step2的内容  

我们想要声明的是，任何链接到 `MathFunctions` 的人都需要包含当前的源目录，而 MathFunctions 本身则不需要。这可以用 `INTERFACE` 使用要求来表示。记住 INTERFACE 是指消费者需要而生产者不需要的东西。见[todo1](./MathFunctions/CMakeLists.txt#L3)  

```cmake
target_include_directories(MathFunctions
                           INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
                           )
```

这样所有链接 MathFunctions 库文件的目标就会自动将该文件夹加入寻找头文件的目录中，todo2~3就是删除原来的引用方式  

使用这种技术，我们的可执行目标使用我们的库所做的唯一事情就是用库目标的名称调用 target_link_libraries()。在大型项目中，手动指定库依赖关系的经典方法很快就会变得非常复杂，所以使用这种技术很有必要的  

## todo4~7

现在我们已经将代码切换到一种更现代的方法，让我们演示一种将为多个目标设置属性的现代技术。  

让我们重构现有代码以使用 INTERFACE 库。我们将在下一步中使用该库来演示 `generator expressions` 的常见用法。  

```cmake
add_library()
target_compile_features()
target_link_libraries()
```


