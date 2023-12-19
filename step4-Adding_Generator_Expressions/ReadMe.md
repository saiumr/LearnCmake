# Adding Generator Expressions  

```cmake
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>"
```

这行CMake代码是用于设置一个变量 gcc_like_cxx，其值根据当前的编译器类型来确定。让我们逐步解释这段代码：  
  
1. `set(gcc_like_cxx ...)`：这一部分使用CMake的set命令来设置一个变量。该变量的名称是 gcc_like_cxx。  
  
2. `"$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>"`：这是一个CMake的生成器表达式（generator expression）。生成器表达式允许在CMake配置和生成阶段执行一些逻辑。  
  
- `COMPILE_LANG_AND_ID` 是一个生成器表达式的函数，用于检查当前编译器是否属于指定的一组编译器。  
  
- `CXX` 表示 C++ 编译器。  
  
- `ARMClang`,`AppleClang`,`Clang`,`GNU`,`LCC` 是一个逗号分隔的编译器标识列表，表示ARMClang、AppleClang、Clang、GNU（例如GCC）和LCC编译器。

  所以，`"$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>"` 的含义是，如果当前编译器是C++编译器，并且是ARMClang、AppleClang、Clang、GNU或LCC之一，那么生成器表达式的值为真；否则，它为假。  
  
3. 最终，set(gcc_like_cxx ...) 将 gcc_like_cxx 设置为生成器表达式的值。这意味着 gcc_like_cxx 将为真（true）或假（false），具体取决于当前使用的C++编译器是否属于上述列出的编译器之一。  

```cmake
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
```

1. `target_compile_options(tutorial_compiler_flags INTERFACE ...)`：这是 CMake 的命令，用于为目标（target）设置编译选项。在这里，目标名称是 `tutorial_compiler_flags`。  
  
2. `INTERFACE`：这表示这些选项是接口选项，意味着它们将被传递给依赖于 `tutorial_compiler_flags` 的目标。  
  
3. `"$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"`：这是一个生成器表达式，根据前面设置的 gcc_like_cxx 变量的值来确定编译选项。具体来说：  
  
- `$<${gcc_like_cxx}: ...>`：如果 `gcc_like_cxx` 为真，那么表达式 `...` 将被执行；否则，将被忽略。

- `$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>`：这是编译选项列表，表示当在构建接口时，要传递给编译器的选项。在这里，它包括一系列用于启用警告的选项，如 `-Wall`、`-Wextra` 等。

  所以，整个表达式的含义是，如果当前编译器是类似于 `GCC` 的 `C++` 编译器（即 `gcc_like_cxx` 为真），则传递一系列警告选项给编译器。

4. `"$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"`：类似地，这一部分是为了 MSVC 编译器设置编译选项。

- `$<${msvc_cxx}: ...>`：如果 `msvc_cxx` 为真，那么表达式 `...` 将被执行。

- `$<BUILD_INTERFACE:-W3>`：在构建接口时，传递 -W3 选项给 MSVC 编译器，这是启用警告的一种方式。  
