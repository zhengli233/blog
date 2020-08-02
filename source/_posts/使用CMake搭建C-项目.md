---
title: 使用CMake搭建C++项目
top: false
cover: false
toc: true
mathjax: true
date: 2020-08-02 16:12:26
password:
summary:
tags:
categories: C++
thumbnail:
---

### CMake是什么
官网[cmake.org](https://cmake.org)是这么描述的：
CMake is an open-source, cross-platform family of tools designed to build, test and package software. CMake is used to control the software compilation process using simple platform and compiler independent configuration files, and generate native makefiles and workspaces that can be used in the compiler environment of your choice. The suite of CMake tools were created by Kitware in response to the need for a powerful, cross-platform build environment for open-source projects such as ITK and VTK.

<!--more-->

翻译过来就是：
CMake是开源、跨平台工具家族的一员，用来生成、测试和打包软件。CMake通过使用简易的平台和独立于编译器的配置文件，根据你的选择来生成编译环境需要的原生的makefile和workspace，并以此控制软件的编译过程。CMake套件由KitWare打造，满足了开源项目如ITK和VTK对一款强大的、跨平台的构建环境的需求。

简单来说，CMake就是一款开源且跨平台的编译环境构造工具，我们可以用它为自己的C++项目编写项目配置文件，并在不同的平台，如Windows、Mac和Linux，生成所需的编译配置和环境。

### 为什么要用CMake
C/C++项目的编译过程大体分为2步，编译和链接。编译就是将源文件生成目标文件，链接就是将源文件中的引用部分转换成目标文件中实际实现的地址（这里实际简略了一些步骤，我没有展开）。CMake的配置文件CMakeLists.txt里面就规定了源文件（对应第一步）和所需的第三方头文件和库文件（对应第二步）。
这里大家可能就看出来了，CMake实际上做了Visual Studio的项目文件（Windows下）和makefile（unix下）的工作。没错，CMake会根据你的平台生成对应的文件，这样我们就只需要维护一份项目配置文件就可以做到多平台通用了。

在我个人经历的团队合作项目中，Visual Studio的项目配置文件总会在分支合并时遇到冲突，这通常是因为不同开发者本地路径不同导致的。即使使用了相对路径，也不能完全避免冲突，这就很令人抓狂。
而CMakeLists作为通用的项目配置文件，使用`find_package`来配置第三方库可以有效避免路径带来的冲突问题，并且环境配置一目了然，赏心悦目。

浏览一下Github上的开源项目，只要是跨平台应用，绝大多数都提供了CMakeLists作为配置文件，方便不同平台的用户使用，可以说是主流的配置工具，学会使用是很有必要的。

### 如何使用CMake
先上一个CMakeLists例子，我们来逐句解释
```
cmake_minimum_required(VERSION 3.8)

message(STATUS "Working in Solution Dir")

find_package(Qt5Core)

add_definitions(-Dxxx)

project(project_name)
include_directories(include)

file(GLOB lib_header /lib/*.h)
file(GLOB head_files /include/*.h)
list(APPEND head_files lib_header)

add_subdirectory(sub_directory)

qt5_add_resources(QRC_FILE ${PROJECT_DIR}/Resources/hawkeye_pro.qrc)

set(EXECUTABLE_OUTPUT_PATH ${SOLUTION_DIR}/build/bin)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

add_library(lib_name SHARED ${source_files})
target_link_libraries(lib_name ${dependent_libs} Qt5::Core)
add_executable(exe ${source_files})
```

`cmake_minimum_required`
CMake版本最低要求，根据实际情况来定。
`message`
打印信息，`STATUS`加不加都行，输出格式不同而已。当生成出错时，可以根据打印信息的位置定位问题。
`find_package`
查找打包的软件，这要求该软件在系统路径下，并且它发布时带有后缀名`.cmake`的配置文件。`.cmake`文件记录了本地该软件的安装情况，`find_package`会将这些信息告诉编译器，这可比写makefile和在vs中设置include和库文件简洁多了。
`add_definitions`
添加宏定义
`project`
项目名
`include_directories`
头文件路径，通过`find_package`配置的头文件不需要在这里重复了
`file`
将众多文件定义为一个变量
`list`
链表操作
`add_subdirectory`
加入一个子项目路径，要求该路径下含有一个CMakeLists文件。子项目会继承父项目的所有变量，但不会影响父项目。执行方式是递归的。
`qt5_add_resources`
Qt项目的特殊操作，编译资源文件，项目不含Qt可以忽略。
`set`
定义变量，这里需要说明的是，CMake有很多预先定义好的变量，这个需要查阅文档。`CMAKE_AUTOMOC`和`CMAKE_AUTORCC`是Qt相关的变量，这里就不详细展开了。
`add_library`
定义生成的库文件，包括名字、静态库或动态库、源文件是哪些等。
`target_link_libraries`
需要链接的第三方库
`add_executable`
定义生成的可执行文件，包括名字、源文件是哪些等。

这里所列出的是一些十分常用的命令，还有一些比较冷门的骚操作没有写进去，这些命令配置基本可以满足一般的项目需求了。

写好CMakeLists之后，新建一个`build`文件夹（不新建也行，看的不闹心就行），在`build`路径下执行
```sh
cmake ${your_cmakelists_path}
```
这样你就会得到一个makefile（unix下）或vs的sln文件（Windows）下。我强烈建议在Windows下使用CMake的Gui程序来生成sln文件，这样可以更清楚的控制生成的vs项目的版本和其他一些配置。

### 总结
CMake是主流的C++跨平台编译环境配置工具，简单易用，即使自己的项目不用，使用别人开源的项目也会用到，学习一下基本用法肯定没错。
