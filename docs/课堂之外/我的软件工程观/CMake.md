# 现代 CMake 高级教程

## 关于 CMake

## 第 0 章：命令行小技巧



## 第 1 章：添加源文件

**单个源文件**

法一：添加一个可执行文件作为构建目标

```cmake
add_executable(main main.cpp)
```

法二：先创建目标，稍后再添加源文件

```cmake
add_executable(main)
target_sources(main PUBLIC main.cpp)
```



**多个源文件**

逐个添加即可

```cmake
add_executable(main)
target_sources(main PUBLIC main.cpp other.cpp)
```

也可以使用变量来存储

```cmake
add_executable(main)
set(sources main.cpp other.cpp)
target_sources(main PUBLIC ${sources})
```

建议把头文件也加上，这样在 VS 里可以出现在“Header Files”一栏

```cmake
add_executable(main)
set(sources main.cpp other.cpp other.h)
target_sources(main PUBLIC ${sources})
```

使用 `GLOB` 自动查找当前目录下指定扩展名的文件，实现批量添加源文件，但这样添加新的文件的时候 `CMake` 可能不会被更新

```cmake
add_executable(main)
set(GLOB sources *.cpp *.h)
target_sources(main PUBLIC ${sources})
```

启用 `CONFIGURE_DEPENDS` 选项，当添加新文件时，自动更新变量

```cmake
add_executable(main)
set(GLOB sources CONFIGURE_DEPENDS *.cpp *.h)
target_sources(main PUBLIC ${sources})
```



**如果源码放在子文件夹里怎么办**？

必须把路径名和后缀名的排列组合全部写出来吗？感觉好麻烦

```cmake
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h mylib/*.cpp mylib/*.h)
target_sources(main PUBLIC ${sources})
```

大可不必！用 `aux_source_directory`，自动搜集需要的文件后缀名

```cmake
add_executable(main)
aux_sources_directory(. sources)
aux_sources_directory(mylib sources)
target_sources(main PUBLIC ${sources})
```

进一步：`GLOB_RECURSE` 了解一下！能自动包含所有子文件夹下的文件

```cmake
add_executable(main)
file(GLOB_RECURSE sources CONFIGURE_DEPENDS *.cpp *.h)
target_sources(main PUBLIC ${sources})
```

`GLOB_RECURSE` 的问题：会把 build 目录里生成的临时 .cpp 文件也加进来

解决方案：要么把源码统一放到 src 目录下，要么要求使用者不要把 build 放到和源码同一个目录里，我个人的建议是把源码放到 src 目录下。



## 第 2 章：项目配置变量

**`CMAKE_BUILD_TYPE`构建的类型，调试模式还是发布模式**

* `CMAKE_BUILD_TYPE` 是 `CMake` 中一个特殊的变量，用于控制构建类型，他的值可以是：
* `Debug` 调试模式，完全不优化，生成调试信息，方便调试程序
* `Release` 发布模式，优化程度最高，性能最佳，但是编译比 `Debug` 慢
* `MinSizeRel` 最小体积发布，生成的文件比 `Release` 更小，不完全优化，减少二进制体积
* `RelWithDebInfo` 带调试信息发布，生成的文件比 `Release` 更大，因为带有调试的符号信息
* 默认情况下 `CMAKE_BUILD_TYPE` 为空字符串，这时相当于 `Debug`。

**各种构建模式在编译器选项上的区别**

在 Release 模式下，追求的是程序的最佳性能表现，在此情况下，编译器会对程序做最大的代码优化以达到最快运行速度。另一方面，由于代码优化后不与源代码一致，此模式下一般会丢失大量的调试信息。
1. `Debug`: `-O0 -g`
2. `Release`: `-O3 -DNDEBUG`
3. `MinSizeRel`: `-Os -DNDEBUG`
4. `RelWithDebInfo`: `-O2 -g -DNDEBUG`
此外，注意定义了 `NDEBUG` 宏会使 `assert` 被去除掉。

**小技巧：设定一个变量的默认值**

如何让 `CMAKE_BUILD_TYPE` 在用户没有指定的时候为 `Release`，指定的时候保持用户指定的值不变呢。
就是说 `CMake` 默认情况下` CMAKE_BUILD_TYPE `是一个空字符串。
因此这里通过 `if (NOT CMAKE_BUILD_TYPE) `判断是否为空，如果空则自动设为` Release `模式。
大多数 `CMakeLists.txt` 的开头都会有这样三行，为的是让默认的构建类型为发布模式（高度优化）而不是默认的调试模式（不会优化）。
我们稍后会详细捋一遍类似于 `CMAKE_BUILD_TYPE` 这样的东西。绝大多数 `CMakeLists.txt `开头都会有的部分，可以说是“标准模板”了。

```cmake
if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE Release)
endif()
```

**project：初始化项目信息，并把当前 `CMakeLists.txt` 所在位置作为根目录**

这里初始化了一个名称为 `hellocmake` 的项目，为什么需要项目名？
对于 `MSVC`，他会在 build 里生成 `hellocmake.sln` 作为“IDE 眼中的项目”。

`CMAKE_CURRENT_SOURCE_DIR` 表示当前源码目录的位置，例如 `~/hellocmake`。
`CMAKE_CURRENT_BINARY_DIR` 表示当前输出目录的位置，例如 `~/hellocmake/build`。

```cmake
cmake_minimum_required(VERSION 3.15)
project(hellocmake)

message("PROJECT_NAME: ${PROJECT_NAME}")
message("PROJECT_SOURCE_DIR: ${PROJECT_SOURCE_DIR}")
message("PROJECT_BINARY_DIR: ${PROJECT_BINARY_DIR}")
message("CMAKE_CURRENT_SOURCE_DIR: ${CMAKE_CURRENT_SOURCE_DIR}")
message("CMAKE_CURRENT_BINARY_DIR: ${CMAKE_CURRENT_BINARY_DIR}")
add_executable(main main.cpp)

add_subdirectory(mylib)
```

和子模块的关系：`PROJECT_x_DIR` 和 `CMAKE_CURRENT_x_DIR` 的区别

* `PROJECT_SOURCE_DIR` 表示最近一次调用 project 的 `CMakeLists.txt `所在的源码目录。
* `CMAKE_CURRENT_SOURCE_DIR` 表示当前 `CMakeLists.txt` 所在的源码目录。
* `CMAKE_SOURCE_DIR 表示最为外层` `CMakeLists.txt `的源码根目录。
* 利用 `PROJECT_SOURCE_DIR` 可以实现从子模块里直接获得项目最外层目录的路径。
* 不建议用 `CMAKE_SOURCE_DIR`，那样会让你的项目无法被人作为子模块使用。

其他

* `PROJECT_SOURCE_DIR`：当前项目源码路径（存放 `main.cpp` 的地方）
* `PROJECT_BINARY_DIR`：当前项目输出路径（存放 `main.exe` 的地方）
* `CMAKE_SOURCE_DIR`：根项目源码路径（存放 `main.cpp` 的地方）
* `CMAKE_BINARY_DIR`：根项目输出路径（存放 `main.exe` 的地方）
* `PROJECT_IS_TOP_LEVEL：BOOL`类型，表示当前项目是否是（最顶层的）根项目
* `PROJECT_NAME`：当前项目名
* `CMAKE_PROJECT_NAME`：根项目的项目名
	详见：https://cmake.org/cmake/help/latest/command/project.html



**project 的初始化：LANGUAGES 字段**

project(项目名 LANGUAGES 使用的语言列表...)  指定了该项目使用了哪些编程语言。
目前支持的语言包括：

* C：C 言
* CXX：C++ 语言
* ASM：汇编语言
* Fortran：老年人的编程语言
* CUDA：英伟达的 CUDA（3.8 版本新增）
* OBJC：苹果的 Objective-C（3.16 版本新增）
* OBJCXX：苹果的 Objective-C++（3.16 版本新增）
* ISPC：一种因特尔的自动 SIMD 编程语言（3.18 版本新增）
* **如果不指定 LANGUAGES，默认为 C 和 CXX。**

常见问题：LANGUAGES 中没有启用 C 语言，但是却用到了 C 语言

```cmake
cmake_mininum_required(VERSION 3.15)
project(hellocmake LANGUAGES CXX)

add_executable(main main.c)
```

也可以先设置 LANGUAGES NONE，之后再调用 enable_language(CXX) 这样可以把 enable_language 放到 if 语句里，从而只有某些选项开启才启用某语言之类的

```cmake
cmake_mininum_required(VERSION 3.15)
project(hellocmake LANGUAGES NONE)
enable_language(CXX)

add_executable(main main.cpp)
```

**设置 C++ 标准：`CMAKE_CXX_STANDARD` 变量**

* `CMAKE_CXX_STANDARD` 是一个整数，表示要用的 C++ 标准。
* 比如需要 C++17 那就设为 17，需要 C++23 就设为 23。
* `CMAKE_CXX_STANDARD_REQUIRED` 是 BOOL 类型，可以为 ON 或 OFF，默认 OFF。
* 他表示是否一定要支持你指定的 C++ 标准：如果为 OFF 则 CMake 检测到编译器不支持 C++17 时不报错，而是默默调低到 C++14 给你用；为 ON 则发现不支持报错，更安全。
* `CMAKE_CXX_EXTENSIONS` 也是 BOOL 类型，默认为 ON。设为 ON 表示启用 GCC 特有的一些扩展功能；OFF 则关闭 GCC 的扩展功能，只使用标准的 C++。
* 要兼容其他编译器（如 MSVC）的项目，**都会设为 OFF 防止不小心用了 GCC 才有的特性**。
* 此外，最好是在 project 指令前设置` CMAKE_CXX_STANDARD` 这一系列变量，这样 CMake 可以在 project 函数里对编译器进行一些检测，看看他能不能支持 C++17 的特性。
* **请勿直接修改 `CMAKE_CXX_FLAGS` 来添加 -std=c++17（你在百度 CSDN 学到的用法）。**
* **请使用 `CMake` 帮你封装好的 `CMAKE_CXX_STANDARD`（从业人员告诉你的正确用法）**。
* 为什么百度不对：你 GCC 用户手动指定了 -std=c++17，让 MSVC 的用户怎么办？此外 CMake 已经自动根据 `CMAKE_CXX_STANDARD` 的默认值 11 添加 -std=c++11 选项了，你再添加个 -std=c++17 选项不就冲突了吗？所以请用 `CMAKE_CXX_STANDARD`。

```cmake
cmake_minimum_required(VERSION 3.15)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

project(hellocmake LANGUAGES CXX)

add_executable(main main.cpp)
```



**project 的初始化：VERSION 字段**

* project(项目名 VERSION x.y.z) 可以把当前项目的版本号设定为 x.y.z。
* 之后可以通过 PROJECT_VERSION 来获取当前项目的版本号。
* PROJECT_VERSION_MAJOR 获取 x（主版本号）。
* PROJECT_VERSION_MINOR 获取 y（次版本号）。
* PROJECT_VERSION_PATCH 获取 z（补丁版本号）。

```cmake
cmake_minimum_required(VERSION 3.15)
project(hellocmake VERSION 0.2.3)

message("PROJECT_NAME: ${PROJECT_NAME}")
message("PROJECT_VERSION: ${PROJECT_VERSION}")
message("PROJECT_VERSION_MAJOR: ${PROJECT_VERSION_MAJOR}")
message("PROJECT_VERSION_MINOR: ${PROJECT_VERSION_MINOR}")
message("PROJECT_VERSION_PATCH: ${PROJECT_VERSION_PATCH}")
```

**一些没什么用，但 CMake 官方不知为何就是提供了的项目字段……**

```cmake
cmake_minimum_required(VERSION 3.15)
project(hellocmake
    DESCRIPTION "A free, open-source, online modern C++ course"
    HOMEPAGE_URL https://github.com/parallel101/course
    )

message("PROJECT_NAME: ${PROJECT_NAME}")
message("PROJECT_DESCRIPTION: ${PROJECT_DESCRIPTION}")
message("PROJECT_HOMEPAGE_URL: ${PROJECT_HOMEPAGE_URL}")
add_executable(main main.cpp)
```

**项目名的另一大作用：会设置另外 <项目名>_SOURCE_DIR 等变量**