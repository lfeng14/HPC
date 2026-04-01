# ARM HPC 高性能计算学习笔记

本项目是关于 ARM 架构下高性能计算（HPC）的学习笔记，涵盖 NEON、SVE 指令集以及性能优化技巧。

## 目录结构

```
.
├── docs/
│   ├── neon/              # NEON 指令集相关
│   │   ├── introduction.md       # NEON 简介
│   │   └── matrix-multiplication.md  # NEON 矩阵乘法优化
│   ├── sve/               # SVE 指令集相关
│   │   ├── introduction.md       # SVE 简介
│   │   └── optimization-notes.md # SVE 优化笔记
│   ├── optimization/      # 通用优化技巧
│   │   ├── narrow.md             # Narrow 优化 - FP32 转 BF16
│   │   └── parallelization.md    # 并行化优化要点
│   └── cache/             # Cache 相关
│       └── introduction.md       # Cache 优化
└── README.md
```

## 内容概览

### NEON
- NEON 128-bit 向量架构及 lane 概念
- NEON vs SVE 的适用场景对比
- 矩阵乘法的 NEON 优化实现

### SVE (Scalable Vector Extension)
- SVE 可变长向量特性
- SVE 寄存器（Z/P/FFR）详解
- 四种 SVE 开发方式
- SVE 汇编优化要点

### 优化技巧
- Narrow 优化：FP32 到 BF16 的批量转换与打包
- 并行化要点：内存对齐、伪共享、循环优化
- restrict 关键字的使用
- 编译器自动向量化指南

### Cache
- Cache 基本原理
- 加载/存储顺序对缓存的影响
- 预加载（Preload）的正确使用
- ARM 过程调用标准（PCS）

## 扩展阅读

- [Arm NEON to SVE Migration](https://developer.arm.com/documentation/102131/0100/Part-Four---Migrate-your-Neon-code-to-SVE)
- [Arm SVE Architecture](https://developer.arm.com/Architectures/Scalable%20Vector%20Extensions)
- [Optimizing with SVE Intrinsics](https://developer.arm.com/documentation/102699/0100/Optimizing-with-intrinsics?lang=en)
- [Arm SVE Instruction Explorer](https://dougallj.github.io/asil/index.html)
