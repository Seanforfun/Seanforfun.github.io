---
layout: post
title:  "LLVM | LLVM Installation On Ubuntu"
date:   2019-07-31 13:17
author: Botao Xiao
categories: llvm
comment: true
description: Installation process on Ubuntu with script file.
---

### Prerequest
1. Git, to download source code and related materials.
2. LLVM source code, from its [github mirror](https://github.com/llvm-mirror/llvm).
3. cmake, tool to compile and install LLVM.
4. Clang, compiler created with LLVM. We can have this source code for clang from its [github mirror](https://github.com/llvm-mirror/clang).

### Download source code for LLVM realated projects
1. Download source code from github.
    ```bash
    git clone https://github.com/llvm/llvm-project.git
    ```

2. Add Clang to LLVM tools
    ```bash
    cp -rf clang ./llvm/tools
    ```

3. Compile and install LLVM, need to be attention that LLVM cannot be compiled in root of program, so we need to create a directory build.
    * -G: generator
    * DCMAKE_INSTALL_PREFIX: final path
    * DCMAKE_BUILD_TYPE: could be debug or Release etc.
    * [Options for compiling LLVM](https://llvm.org/docs/CMake.html)
    
    ```bash
    mkdir build
    cd build
    cmake -G "Unix Makefiles" -DLLVM_TARGETS_TO_BUILD=host -DCMAKE_INSTALL_PREFIX=/usr/local/llvm -DCMAKE_BUILD_TYPE=MinSizeRel -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DLLVM_INCLUDE_EXAMPLES=OFF -DLLVM_INCLUDE_TESTS=OFF -DCLANG_INCLUDE_TESTS=OFF ..
    make -j10 # modify accourding to your cpu lcore number.
    make install
    ```

### My Installation Script, run in administrator
```bash
# !/bin/bash
# This is a bash file to install LLVM on unbuntu. Run in administrator mode(root).

echo --------------------------------------------------------------------------
echo STEP 1: Preparation. Update apt and install Cmake and git
echo --------------------------------------------------------------------------
apt update
apt install cmake
apt install git
echo --------------------------------------------------------------------------
echo STEP 2: Download source code of LLVM.
echo --------------------------------------------------------------------------
# Please change to your desired directory from root.
LLVM_ROOT='/home/xiaob6/test'
if [ ! -d '$LLVM_ROOT' ];then
    mkdir $LLVM_ROOT
else
	echo [$LLVM_ROOT] has already existed.
	exit 1
fi
cd $LLVM_ROOT
git clone https://github.com/llvm/llvm-project.git
cd llvm-project
cp -rf clang ./llvm/tools
echo --------------------------------------------------------------------------
echo STEP 3: Compile and install LLVM
echo --------------------------------------------------------------------------
cd ./llvm
mkdir build
cd build
cmake -G "Unix Makefiles" -DLLVM_TARGETS_TO_BUILD=host -DCMAKE_INSTALL_PREFIX=/usr/local/llvm -DCMAKE_BUILD_TYPE=MinSizeRel -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DLLVM_INCLUDE_EXAMPLES=OFF -DLLVM_INCLUDE_TESTS=OFF -DCLANG_INCLUDE_TESTS=OFF ..
make -j10 # modify accourding to your cpu lcore number.
make install
```
