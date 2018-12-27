---
title: ubuntu上用MinGW-w64 交叉编译LLVM+Clang
date: 2018-12-27 16:13:40
mathjax: true
tags: [LLVM, 交叉编译, MinGW, ubuntu]
---


# 设置

Ubuntu18.04, MinGW-w64,LLVM6.0+Clang 按照官网配置编译目录

安装MinGW-w64

    sudo apt install mingw-w64


# 方法1

按照[官网](https://llvm.org/docs/HowToCrossCompileLLVM.html)交叉编译的方式，先本地编译再指定MinGW编译

```
-DCMAKE_CROSSCOMPILING=True
-DCMAKE_INSTALL_PREFIX=<install-dir>
-DLLVM_TABLEGEN=<path-to-host-bin>/llvm-tblgen
-DCLANG_TABLEGEN=<path-to-host-bin>/clang-tblgen
-DLLVM_DEFAULT_TARGET_TRIPLE=arm-linux-gnueabihf
-DLLVM_TARGET_ARCH=ARM
-DLLVM_TARGETS_TO_BUILD=ARM

If you’re compiling with GCC, you can use architecture options for your target, and the compiler driver will detect everything that it needs:

-DCMAKE_CXX_FLAGS='-march=armv7-a -mcpu=cortex-a9 -mfloat-abi=hard'
```

# 方法2

在`cmake/platform`目录下建立`.cmake`空文件，详见`cmake/modules/CrossCompile.cmake`

附`CMAKE_TOOLCHAIN_FILE`设置


```
if (CMAKE_SYSTEM_NAME MATCHES "Windows")
# the name of the target operating system
# SET(CMAKE_SYSTEM_NAME Windows)

# which compilers to use for C and C++
SET(CMAKE_C_COMPILER x86_64-w64-mingw32-gcc-posix)
SET(CMAKE_CXX_COMPILER x86_64-w64-mingw32-g++-posix)
# SET(CMAKE_RC_COMPILER i586-mingw32msvc-windres)

# here is the target environment located
SET(CMAKE_FIND_ROOT_PATH  /usr/x86_64-w64-mingw32)

# adjust the default behaviour of the FIND_XXX() commands:
# search headers and libraries in the target environment, search
# programs in the host environment
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)

endif (CMAKE_SYSTEM_NAME MATCHES "Windows")

# cmake -G "Unix Makefiles" -DCMAKE_SYSTEM_NAME=Windows -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=../ToolChain.cmake -DCMAKE_INSTALL_PREFIX=/Program/llvm ../..
```

