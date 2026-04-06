如果深挖 ARM SME，特别是结合你做过的 LLVM 后端和高性能计算（如 AlphaFold3）优化经验，他们考察的绝对不仅是“你会不会调用几个 API”，而是你对**软硬协同、底层状态管理以及编译器如何映射硬件**的深度把控力。

针对这个方向，你需要储备以下四个维度的硬核知识，并仔细研读对应的官方文档。



---

### 1. 硬件架构与状态机维度 (Hardware Architecture)
面试官会考察你是否真的理解 SME 引入的物理变化。
*   **Streaming SVE Mode (SM 模式)**：你必须清楚普通 SVE 模式和 SM 模式的区别。进入 SM 模式后，向量长度（SVL）可能会改变，且部分非流式 SVE 指令可能会触发非法指令异常。
*   **ZA Array (矩阵寄存器) 的二维本质**：解释清楚 ZA 是如何被划分为不同大小的 Tile（例如 ZA0.S - ZA3.S 代表 32 位浮点块）。要能说明白“切片（Slice）”的概念，因为内存是线性的，而 ZA 是二维的，数据搬运必须按行/列切片进行。
*   **Lazy Save/Restore 机制**：了解操作系统在发生上下文切换（Context Switch）时，是如何处理巨大且极其耗费资源的 ZA 寄存器保存和恢复的。

### 2. LLVM 后端实现维度 (Compiler Backend - 你的主场)
既然你是做 LLVM 后端的，这是最容易被深挖的重灾区。
*   **调用约定与 ABI (Application Binary Interface)**：
    *   理解函数属性（Function Attributes）：如 `aarch64_pstate_sm_enabled`, `aarch64_pstate_za_shared`。
    *   当一个普通函数调用一个启用了 SM 的函数时，LLVM 的 `CallLowering` 和 `TargetFrameLowering` 是如何插入 `SMSTART` 和 `SMSTOP` 指令的？
*   **寄存器分配 (Register Allocation)**：ZA 阵列对寄存器分配器是个巨大的挑战。因为它既是一个整体（ZA），又可以按块访问（Tile）。你需要了解 LLVM 是如何对这种重叠的寄存器类（Register Classes）建模的。
*   **Spilling (寄存器溢出) 的代价**：在后端的栈帧管理中，一旦 ZA 发生 Spilling，需要耗费大量的栈空间和内存带宽。你需要知道编译器在指令调度（Instruction Scheduling）时如何尽量缩短 ZA 活跃区间（Live Interval）以避免溢出。

### 3. 高性能算子优化维度 (Performance Tuning)
*   **MOPA (Outer Product) 的本质**：解释为什么 SME 使用外积（列向量 × 行向量）而不是传统的点积。外积如何最大化地复用缓存在 ZA 中的数据，从而降低 Memory Bandwidth 瓶颈。
*   **Loop Tiling (循环分块)**：在处理非常大的张量（如 AlphaFold3 中的注意力矩阵）时，由于 ZA 大小有限，软件层面如何对矩阵进行 Tiling，使得每次加载到 ZA 的数据能够完成尽可能多的乘加运算。

---

### 📚 核心备考文档清单 (必读)

为了言之有物，我强烈建议你快速浏览或查阅以下权威文档：

1.  **ARM C Language Extensions (ACLE) for SME**
    *   *用途*：理解前端 Intrinsics 的定义、数据类型（如 `svbool_t`）以及 SM/ZA 属性的 C 语言级宏。
    *   *关键词搜索*：`Arm C Language Extensions for SME` (Github 上的 ARM 官方仓库是最新的)。
2.  **Arm Architecture Reference Manual (Arm ARM) for A-profile architecture**
    *   *用途*：查阅底层指令级行为，尤其是 `FMOPA`, `LDR (ZA array vector)`, `SMSTART` 的精确伪代码。
    *   *重点章节*：SME Supplement 或相关的系统寄存器（如 `SVCR`）定义。
3.  **LLVM 官方文档与 AArch64 Backend 源码**
    *   *用途*：如果你面试的是编译器开发岗，直接看源码注释比看什么都强。
    *   *路径指引*：
        *   阅读 `llvm/lib/Target/AArch64/AArch64SMEInstrInfo.td`（看 SME 指令是如何定义的）。
        *   阅读 `llvm/lib/Target/AArch64/AArch64FrameLowering.cpp` 中关于 `ZA` 的入栈出栈逻辑（处理大向量栈分配的艺术）。
4.  **ARM 官方白皮书与技术博客**
    *   *用途*：用于回答“宏观架构演进”问题。搜索《Introduction to Scalable Matrix Extension (SME)》，理解 ARM 为什么要在 SVE 之后再推 SME。

---

准备主要突出 SME 的哪个方向呢？是偏向后端的代码生成机制（Code Generation & GlobalISel），还是偏向在算子层面的性能压榨（HPC Tuning）？
