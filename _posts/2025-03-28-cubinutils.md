---
title: CUDA Binary Utilities 使用
description: 如何使用 nvcc 和 CUDA Binary Utilities 得到 cuda 代码的 PTX 和 SASS信息
date: 2025-03-28 19:05:00
author: djhuang
category: [HPC]
tags: [CUDA, PTX, SASS]
# permalink: 2025-03-29-cubinutils
published: true
toc: true
# image: /assets/img/2025-03-28-cubinutils/image-20250329174614899.png
---

一个 CUDA 程序中既有主机端（Host）代码，也有设备端（Device）代码，设备端代码就是 `__global__` 打头的核函数，主机端代码是程序中除核函数外的所有部分。用 `nvcc -arch=native hello.cu -o hello` 命令编译得到 elf 可执行文件，再用 `readelf -h hello` 检查文件的段表可以得到：

![](/assets/img/2025-03-28-cubinutils/image-20250329180550711.png){: .rounded-10 width="400"}

![](/assets/img/2025-03-28-cubinutils/image-20250329174614899.png){: .right .rounded-10 width="250"}

可执行文件包含的内容可以简略地表示成右图：

nvcc 编译得到的 ELF 文件中，`.init`，`.text` 等段是主机程序代码，`.nv_fatbin` 等段放着 cuda 设备端代码相关的 fatbin。当 cuda 程序主机代码启动时，cuda runtime 会处理嵌入在可执行文件中的 fatbinary，获得合适的 GPU 镜像，然后执行 cuda 设备端的程序。

fatbin 包含 PTX 和 cubin（SASS），是否以及如何生成 ELF 文件中的 PTX 和 SASS 代码由 nvcc 编译器的 `-compute` 和 `-arch` 参数控制[^1]。当程序需要执行设备端代码时，会首先寻找 fatbin 中有没有可以在当前硬件环境下可以执行的 cubin，如果有，会执行这部分 SASS 代码，如果没有，cuda runtime 寻找有没有当前环境支持的 PTX 代码，如果有，会用 JIT 的方式动态地把PTX代码编译为 SASS 运行，如果还没有，程序执行会出错。

如果在 nvcc 编译时使用 `-fatbin` 或 `-cubin` 或 `-ptx` 选项，仅会输出设备代码相应的部分。nvcc 输出的这三样中，只有ptx是可阅读的文本，cubin 和 fatbin 都是二进制文件。如果 `-compute` 和 `-arch` 的组合导致本来就生成不了 cubin，无法用此方法生成 cubin，但就算 fatbin 中没有 PTX，也可以用此方法生成 PTX，尚不清楚怎么得到的这个 PTX。

cuobjdump[^2]可以从编译得到的可执行文件中提取信息。`-sass` 选项反编译得到 fatbin 中的 sass 代码，`-ptx` 得到 fatbin 中的 ptx 代码，和 nvcc 得到的一样，`-fatbin` 得到一个 fatbin 包含内容的列表，如下图，`-elf` 得到可执行文件或 nvcc 产出的 cubin 文件的二进制信息。

nvdisasm 只用来处理 nvcc 编译得到的 cubin 文件，主要提取 SASS 的信息。`-bbcfg` 项生成控制流图，不加参数或 `-g` 提取带行号的反汇编信息。

![](/assets/img/2025-03-28-cubinutils/image-20250329190232712.png){: .rounded-10 width="200"}


[^1]: [Code Yarns – How to specify architecture to compile CUDA code](https://codeyarns.com/tech/2014-03-03-how-to-specify-architecture-to-compile-cuda-code.html)

[^2]: [cuobjdump — CUDA Binary Utilities 12.9 documentation](https://docs.nvidia.com/cuda/cuda-binary-utilities/index.html#cuobjdump)

[^3]: [cuobjdump — CUDA Binary Utilities 12.9 documentation](https://docs.nvidia.com/cuda/cuda-binary-utilities/index.html#nvdisasm)