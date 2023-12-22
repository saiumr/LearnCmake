# Packing and Installer  

使用 `cpack` 打包项目  

编译好之后，在build下直接执行  

```bash
cpack
```

使用 `-G` 选项，指定生成，使用 `-C` 选项，指定生成不同发布版  

```bash
cpack -G ZIP -C Debug
```

生成器是用于定义生成安装包的方式的工具，可以将项目构建的输出打包为不同的格式，如ZIP、DEB、RPM等  

通过运行 `cpack --config CPackSourceConfig.cmake` 命令，你可以使用 CPack 工具来生成一个包含完整源代码树的归档文件。这通常用于创建源代码发布的归档文件。  

如果使用windows打包应用程序需要安装NSIS以支持cpack  
