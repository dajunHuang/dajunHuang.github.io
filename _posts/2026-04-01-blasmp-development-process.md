---
title: BLASMp 开发流程笔记（AI Generated）
description: BLASMp 是我博士期间完成的首个完整项目，因为分布式程序的调试难度和能接触到的硬件平台的使用难度，开发它伴随着极大的磨难
date: 2026-04-20 20:42:00
author: djhuang
category: [Experience, HPC]
tags: [AI Generated, BLASMp, Multi-GPU, BLAS3]
published: true
toc: true
published: false
image:
  path: /assets/img/2026-04-01-blasmp-development-process/image.png
  alt: GEMM 通信与计算的完美重叠
---

> 十分感谢实验室另外两位同学愿意帮我分担部分代码工作
{: .prompt-tip }

> 因为 Code Agent 不仅能迅速总结仓库的提交历史，还能把历史和我开发过程中留下的笔记（private）相对应，所以原谅我选择用它来总结生成这篇文章。
{: .prompt-info }

> Code Agent 拥有独立的神经网络，它肯定是无从得知开发过程中我自己这个神经网络是有多么受挫并总结出来的。BLASMp 导致的磨难提醒我更好地选择进行科研的方式。
{: .prompt-danger }

本文把 BLASMp 的开发过程整理为一条可复用的工程路径，信息来源包括：

- 主分支提交历史[^1]（截至 2026-04-17）
- 实验笔记（2025-09 到 2026-04）

---

## 1. 项目定位与总体思路

BLASMp 的目标不是单纯“实现 BLAS3 接口”，而是实现一套面向多 GPU、可跨节点、通信与计算可重叠的分布式 BLAS3 执行框架。整体技术路线可以概括为：

1. 数据采用 2D block-cyclic 分布（接近 PBLAS/ScaLAPACK 思路）。
2. 通信主通道选择 NVSHMEM，进程管理和全局同步使用 MPI。
3. 本地计算优先复用 cuBLAS（后续逐步剥离 CUTLASS 依赖，聚焦稳定主路径）。
4. 每个算子遵循统一模板：Arguments + workspace + preload/compute pipeline + recv/ack 信号协议。

这个路线与笔记中的结论一致：多 GPU BLAS3 的决定因素是通信结构、分块策略、以及同步粒度，而不仅是单核算力。

---

## 2. 时间线总览（阶段化）

### 阶段 A：预研与基线摸底（2025-09 到 2025-10，主要在笔记中）

核心工作：

- 对比了 magma、SLATE、cuBLASMp、BLASX、XKBLAS、CUTLASS 等路线。
- 系统记录不同库在多机多卡、不同 block size、不同布局下的性能。
- 总结 BLAS/LAPACK 单块布局与 PBLAS 分布式布局的差异。
- 明确了后续方向：
  - 先以 cuBLAS/cuBLASMp 为正确性和性能锚点。
  - 重点攻克通信聚合、流水重叠、信号同步正确性。

这个阶段的价值：把“优化目标”从单点 kernel 性能，转成“端到端流水效率”。

---

### 阶段 B：仓库冷启动与原型成型（2025-11）

提交特征：

- 初始脚手架和测试提交（test1/test2/test3/test4）。
- 早期尝试 CUTLASS 路径（e2dbe39）。
- 建立 cuBLASMp 基线 profile（a91354e）。
- SYRK 从“仅通信框架”走向“可计算可校验”：
  - a2f4c88：加入 blasmp syrk 框架（无完整计算）
  - c03247c：用 cublas gemm 完成 syrk 主计算并加 check

阶段产出：

- GEMM/SYRK 初代实现框架。
- 首批 check/profile 测试闭环。

---

### 阶段 C：通信协议打磨与流水稳定化（2025-12）

这是第一轮“工程硬化期”，提交密度高（12 月 32 次）。

核心问题：

- 信号协议容易死锁或覆盖（put/wait/ack 顺序、quiet 语义、stream 同步点）。
- 阶段复用时必须清楚哪些信号需要 ACK，哪些只需 recv。
- NVSHMEM 与 MPI 层同步职责边界需要明确。

关键提交与变化：

- d7b1b48：把 gemm/syrk 拆成 preload + compute，形成后续统一范式。
- 91ec7ba、60ba886、c27ec5c、8cdabed：围绕信号、barrier、quiet 的多轮修复。
- 817a496：comp_stream 重命名为 comm_stream，通信/计算语义更清晰。
- f7aced5：gemm 切到 nvshmem + cublas 主模式。

与笔记互证的经验：

- 先收后发（或先发后收）次序错误会导致卡死。
- 某些同步看似冗余，但撤掉后会暴露隐式竞态。

阶段产出：

- 可重复运行的多阶段流水。
- 一套可调试的信号/事件观测手段（debug log、host print、profile 输出增强）。

---

### 阶段 D：TRSM 攻坚 + SYR2K 集成（2026-01）

这是功能扩展与稳定性攻坚叠加的阶段（1 月 38 次提交，为最高峰）。

TRSM 主线：

- 727eaf1：temp trsm 起步。
- eee6617、20ac77b：初版完整 TRSM。
- 3aa8c37：减少到必要通信，优化主路径。
- 2fe616d：nbi 转 non-nbi，规避一致性和完成语义问题。
- 30803d5、a347aaf、5ffe54f：batch-1 版本与 overlap 持续优化。
- 47c9387：新旧 TRSM 分拆为 trsm.hpp 与 trsm_back.hpp，便于回归。

SYR2K 主线：

- 4e3ef7d：补齐 syr2k.hpp 和 check/profile。
- a154b3c：合并 syr2k 分支，形成完整算子链。

与笔记一致的关键教训：

- workspace 初始化是硬约束，尤其在多 stage 复用时。
- ACK 何时需要，本质取决于“缓冲区是否会在下一轮复用”。
- TRSM 正确性不能只看 B 对比，需要用 AB-B_ori 与前向/后向误差联合验证。

阶段产出：

- TRSM 可用并进入性能调参阶段。
- SYR2K 具备基础可用版本。

---

### 阶段 E：算子扩展、结构清理、跨节点修复（2026-02 到 2026-03）

关键动作：

- 9057312：Symmv2，补齐 SYMM。
- 6b9cfb3：Strip cutlass and format code，代码结构统一化。
- dd54fb1：SYMM 优化并修复跨节点 bug。
- a44915e：SYRK 并行发送 AT，针对转置矩阵传输瓶颈优化。
- a440a82：加入 TRMM。
- a071615：修复 SYR2K。

阶段特征：

- 从“单算子可跑”转向“多算子统一框架 + 可维护代码库”。
- 从“单节点性能”转向“跨节点一致性与可扩展行为”。

---

### 阶段 F：聚合通信与规模化收敛（2026-04）

关键动作：

- b6f55c4：SYR2K 改造为更接近 SYRK 的通信/计算风格。
- 0545389、2b8ab26：ag syrk / ag syr2k（聚合相关迭代）。
- c55bcff：SYMM 最新更新收尾。
- README/CMake 持续更新，测试入口更清晰。

与笔记一致的技术方向：

- 对 SYRK/SYR2K/SYMM 这类“转置参与、通信碎片化严重”的算子，通信聚合是提升关键。
- 开始系统推进强扩展/弱扩展实验，准备论文化表达与图表化结论。

---

## 3. 从提交历史看到的工程模式

### 3.1 先基线，后优化

流程上始终先建立可比对象（cuBLASMp profile/check），再做 BLASMp 优化，避免“自证正确”。

### 3.2 先正确，再重叠

几乎每个算子都经历：

1. 先打通最小通信与计算路径。
2. 用 check 程序做功能正确性。
3. 再引入 num_stages、双流、信号复用、通信聚合。
4. 再用 profile 看吞吐和延迟。

### 3.3 多轮小步回滚友好

例如 trsm.hpp / trsm_back.hpp 的并存，说明你在高风险阶段保留了“可回退实现”，这对分布式通信调试非常重要。

### 3.4 测试程序与算子同速演进

不是“最后补测试”，而是每个新算子都尽快跟上 check/profile。提交热度也印证了这一点（如 syrk/trsm 的 test 文件长期高频更新）。

---

## 4. 当前仓库状态（截至 2026-04-17）

- 总提交数：103
- 时间范围：2025-11-06 到 2026-04-17
- 月度节奏：
  - 2025-11：17
  - 2025-12：32
  - 2026-01：38
  - 2026-02：2
  - 2026-03：6
  - 2026-04：8

热点文件（按触达频次）：

- include/blasmp/syrk.hpp（29）
- test/blasmp_syrk_check.cu（21）
- include/blasmp/trsm.hpp（20）
- test/blasmp_gemm_profile.cu（18）
- test/blasmp_gemm_check.cu（18）

这些数据说明：

- 第一主线是 SYRK 通信结构。
- 第二主线是 TRSM 难点攻坚。
- GEMM 则是长期基座与性能标尺。

---

## 5. 可复用的 BLASMp 开发模板（给后续新算子）

建议继续沿用以下模板开发新算子或重构：

1. 先定义数据分布与最小工作流（Arguments、workspace、tile 映射）。
2. 先完成单阶段正确路径，再引入多 stage。
3. 明确每个信号的生命周期：谁写、谁等、何时复用、是否需要 ACK。
4. check 与 profile 同步上线：
   - check 关注数值误差与跨 rank 一致性。
   - profile 关注 warmup/repeat 统计与 rank 维度输出。
5. 与 cuBLASMp/SLATE 持续对比，避免优化方向偏航。
6. 保留一个“保守实现”用于回退，直到新路径稳定。

---

## 6. 总结

BLASMp 的开发过程，本质上是从“算子实现”逐步升级为“分布式流水系统工程”：先用基线锚定，再通过信号协议、阶段复用、通信聚合和测试闭环，把 GEMM/SYRK/TRSM/SYR2K/SYMM/TRMM 逐步做成可验证、可调优、可扩展的一套统一框架。

[^1]: [BLASMp commit history](https://github.com/uestc-ndsl-hpc/blasMp/commits/main/)