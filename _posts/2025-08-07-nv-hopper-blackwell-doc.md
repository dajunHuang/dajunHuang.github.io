---
title: NVIDIA Hopper 和 Blackwell 系列的产品们
description: 整理了一下 Hopper 和 Blackwell 系列产品的资料
date: 2025-07-08 17:00:00
author: djhuang
category: [HPC]
tags: [NVIDIA, GPU, Hopper, Blackwell]
published: true
# image: /assets/pic/posts/HPC/2025-08-07-nv-hopper-blackwell-doc/image.png
---

在网上搜了一下英伟达 Hopper 和 Blackwell 系列的GPU产品，结果搜到的资料很碎片，没有清晰的结构，产品介绍里全都是广告词汇，故意不让人一眼看出产品在技术和性能指标上的关系和不同，于是整理了一下搜索的结果。括号里是实际硬件配置，最后的数字是文档链接。


| GPU                                                          | DGX(GPU 服务器)[1](https://www.nvidia.com/en-us/data-center/dgx-platform/),[2](https://docs.nvidia.com/dgx-systems/index.html) | DGX SupperPOD（多节点GPU服务器）[1](https://www.nvidia.com/en-us/data-center/dgx-superpod/),[2](https://resources.nvidia.com/en-us-dgx-systems/dgx-ai-4?ncid=no-ncid),[3](https://docs.nvidia.com/dgx-superpod/#deployment-guides) | HGX(GPU 服务器基础平台)[1](https://www.nvidia.com/en-us/data-center/hgx/) |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| H100[1](https://www.nvidia.com/en-us/data-center/h100/)      | DGX H100 (8*H100)[1](https://www.nvidia.com/en-gb/data-center/dgx-h100/),[2](https://resources.nvidia.com/en-us-dgx-systems/ai-enterprise-dgx?ncid=no-ncid&xs=489753),[3](https://docs.nvidia.com/dgx/dgxh100-user-guide/introduction-to-dgxh100.html) | DGX H100 SuperPOD (32*DGX H100)[1](https://docs.nvidia.com/dgx-superpod/reference-architecture-scalable-infrastructure-h100/latest/index.html) | HGX H100 (8*H100)[1](https://nvdam.widen.net/s/5kgbjq2v2t/hpc-hgx-h100-datasheet-nvidia-web) |
| H200[1](https://www.nvidia.com/en-us/data-center/h200/)      | DGX H200 (8*H200) [1](https://www.nvidia.com/en-us/data-center/dgx-h200/) | DGX H200 SuperPOD (32*DGX H200)[1](https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-h200/latest/index.html) | HGX H200 (8*H200)[1](https://nvdam.widen.net/s/5kgbjq2v2t/hpc-hgx-h100-datasheet-nvidia-web) |
| GH200 (1\*Grace CPU+1\*H200)[1](https://www.nvidia.com/en-us/data-center/grace-hopper-superchip/),[2](https://resources.nvidia.com/en-us-grace-cpu/nvidia-grace-hopper) | DGX GH200(8*GH200)[1](https://www.nvidia.com/en-in/data-center/dgx-gh200/),[2](https://www.zhaocs.info/wp-content/uploads/2024/04/dgx-gh200-whitepaper.pdf),[3](https://download.deltacomputer.com/nvidia-dgx-gh200-datasheet-us-web.pdf) | /                                                            | /                                                            |
| /                                                            | DGX B200 (8*B200)[1](https://www.nvidia.com/en-us/data-center/dgx-b200/),[2](https://docs.nvidia.com/dgx/dgxb200-user-guide/index.html) | DGX B200 SuperPOD (32*DGX B200)[1](https://docs.nvidia.com/dgx-superpod/reference-architecture-scalable-infrastructure-b200/latest/index.html) | HGX B200 (8*B200)[1](https://nvdam.widen.net/s/wwnsxrhm2w/blackwell-datasheet-3384703) |
| /                                                            | DGX GB200, GB200 NVL72 (36\*Grace CPU+72\*B200)[1](https://www.nvidia.com/en-us/data-center/dgx-gb200/?ncid=no-ncid),[2](https://resources.nvidia.com/en-us-dgx-systems/dgx-superpod-gb200-datasheet?ncid=no-ncid),[3](https://www.nvidia.com/en-us/data-center/gb200-nvl72/) | DGX GB200 SuperPOD (8*DGX GB200)[1](https://docs.nvidia.com/dgx-superpod/reference-architecture-scalable-infrastructure-gb200/latest/index.html) | /                                                            |
| /                                                            | DGX B300 (2\*Intel Xeon CPU + 8\*NVIDIA Blackwell Ultra GPU)[1](https://www.nvidia.com/en-us/data-center/dgx-b300/),[2](https://resources.nvidia.com/en-us-dgx-systems/dgx-b300-datasheet?ncid=no-ncid) | DGX B300 SuperPOD (64*DGX B300)[1](https://docs.nvidia.com/dgx-superpod/reference-architecture/scalable-infrastructure-b300/latest/abstract.html) | HGX B300(8\*NVIDIA Blackwell Ultra GPU)[1](https://nvdam.widen.net/s/wwnsxrhm2w/blackwell-datasheet-3384703) |
| /                                                            | DGX GB300, GB200 NVL72 (36\*Grace CPU+72\*NVIDIA Blackwell Ultra GPU)[1](https://resources.nvidia.com/en-us-dgx-systems/dgx-gb300-datasheet?ncid=no-ncid),[2](https://www.nvidia.com/en-us/data-center/gb300-nvl72/) | /                                                            | /                                                            |

