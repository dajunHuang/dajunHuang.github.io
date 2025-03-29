---
layout: posts
title: CUDA Binary Utilities 使用
description: 如何使用 nvcc 和 CUDA Binary Utilities 得到 cuda 代码的 PTX 和 SASS信息
date: 2025-03-28 19:05:00
category:
- Code
tags:
- CUDA
- PTX
- SASS
permalink: 2025-03-29-cubinutils
published: true
image: /assets/pic/posts/Code/2025-03-28-cubinutils/image-20250329174614899.png
---

一个CUDA程序中既有主机端(Host)代码，也有设备端(Device)代码，设备端代码就是`__global__`打头的核函数，主机端代码是程序中除核函数外的所有部分。用`nvcc -arch=native hello.cu -o hello`命令编译得到elf可执行文件，再用`readelf -h hello`检查文件的段表可以得到：

<img src="/assets/pic/posts/Code/2025-03-28-cubinutils/image-20250329180550711.png" alt="image-20250329180550711" style="zoom:50%;" />

可执行文件包含的内容可以简略地表示成：

<img src="/assets/pic/posts/Code/2025-03-28-cubinutils/image-20250329174614899.png" alt="image-20250329174614899" style="zoom: 40%;" />

nvcc编译得到的ELF文件中，`.init`，`.text`等段是主机程序代码，`.nv_fatbin`等段放着cuda设备端代码相关的fatbin。当cuda程序主机代码启动时，cuda runtime会处理嵌入在可执行文件中的fatbinary，获得合适的GPU镜像，然后执行cuda设备端的程序。

fatbin包含PTX和cubin(SASS)，是否以及如何生成ELF文件中的PTX和SASS代码又nvcc编译器的`-compute`和`-arch`参数控制【1】。当程序需要执行设备端代码时，会首先寻找fatbin中有没有可以在当前硬件环境下可以执行的cubin，如果有，会执行这部分SASS代码，如果没有，cuda runtime寻找有没有当前环境支持的PTX代码，如果有，会用JIT的方式动态地把PTX代码编译为SASS运行，如果还没有，程序执行会出错。

如果在nvcc编译时使用`-fatbin`或`-cubin`或`-ptx`选项，仅会输出设备代码相应的部分。nvcc输出的这三样中，只有ptx是可阅读的文本，cubin和fatbin都是二进制文件。如果`-compute`和`-arch`的组合导致本来就生成不了cubin，无法用此方法生成cubin，但就算fatbin中没有PTX，也可以用此方法生成PTX，尚不清楚怎么得到的这个PTX。

cuobjdump【2】可以从编译得到的可执行文件中提取信息。`-sass`选项反编译得到fatbin中的sass代码，`-ptx`得到fatbin中的ptx代码，和nvcc得到的一样，`-fatbin`得到一个fatbin包含内容的列表，如下图，`-elf`得到可执行文件或nvcc产出的cubin文件的二进制信息。

<img src="/assets/pic/posts/Code/2025-03-28-cubinutils/image-20250329190232712.png" alt="image-20250329190232712" style="zoom:50%;" />

【1】[Code Yarns – How to specify architecture to compile CUDA code](https://codeyarns.com/tech/2014-03-03-how-to-specify-architecture-to-compile-cuda-code.html)

【2】[1. Overview — CUDA Binary Utilities 12.8 documentation](https://docs.nvidia.com/cuda/cuda-binary-utilities/index.html#cuobjdump)