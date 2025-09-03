# Reduced AE Plugin Template

[中文](#中文文档) | [English](#english-documentation)

---

## English Documentation

A minimal After Effects plugin development template that includes plugin entry points, configuration files, AE headers, and CMakeLists, designed to replace Adobe's official Skeleton template for AE plugin development.

### Features

1. Support CMake, can be used to build plugins on Windows and macOS
2. Minimal template, implements a simple pixel-copying plugin with all unnecessary code removed from the template
3. Uses patched PiPLtool.exe, which allows spaces in your project path

### Usage

#### For Windows Users

You can download and use the following commands to generate the project:

```bash
mkdir build
cd build
cmake .. -G "Visual Studio 17 2022"
cmake --build . --config Release
```

#### For macOS Users

Sorry, I have not tested this on the macOS platform.

### Related Projects

- CMakeLists.txt modified from: https://github.com/mobile-bungalow/after_effects_cmake
- PiPLtool.exe obtained from: https://github.com/dolag233/NewPiPLtool - This project's patch enables PiPLtool.exe to support file path parameters with spaces

---

## 中文文档

这是一个最小的AE插件开发模板，包括了插件入口、插件配置文件、AE头文件、CMakeList等，旨在替换AE官方的Skeleton，作为AE插件开发的模板。

### 特点

1. 支持CMake，可用于在Windows和MacOS构建插件
2. 最小模板，实现了一个单纯拷贝像素的插件，将所有无用代码从模板中移除
3. 使用了patch过的PiPLtool.exe，这让你的工程路径中可以附带空格

### 使用方法

#### 对于Windows用户

你可以直接下载后使用以下命令生成工程：

```bash
mkdir build
cd build
cmake .. -G "Visual Studio 17 2022"
cmake --build . --config Release
```

#### 对于macOS用户

对不起我并没有在macOS平台上进行测试过。

### 相关项目

- CMakeList.txt从此项目中修改而来：https://github.com/mobile-bungalow/after_effects_cmake 
- PiPLtool.exe从此项目中获取：https://github.com/dolag233/NewPiPLtool 这个项目的patch使得PiPLtool.exe支持了带空格的文件路径参数