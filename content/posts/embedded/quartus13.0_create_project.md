---
title: 'Quartus II 13.0 创建一个项目'
date: '2026-10-24T21:29:25+08:00'
draft: true
lastmod: '2026-10-24T21:29:25+08:00'

categories: ['electronics']
tags: []

cover: "/images/posts/cover.jpg"

showtoc: true
comments: true

keywords: []
---

记录用 Quartus II 13.0 创建一个项目的过程。

<!--more-->
`File->New`：

![](/images/posts/Quartus_create_project_01.png)

设置项目路径&项目名&top-level：

![](/images/posts/Quartus_create_project_02.png)

添加设计文件：

![](/images/posts/Quartus_create_project_03.png)

选择芯片型号：

我的芯片型号为：`EP4CE10F17C8`

![](/images/posts/Quartus_create_project_04.png)


EDA 工具设置：

![](/images/posts/Quartus_create_project_05.png)

完成页面：

![](/images/posts/Quartus_create_project_06.png)

## 项目结构推荐

```bash
My_FPGA_Project/
│
├── My_FPGA_Project.qpf        # Quartus 工程文件（Quartus Prime Project File）
├── My_FPGA_Project.qsf        # Quartus 设置文件（包含引脚分配、编译选项等）
│
├── rtl/                       # 【重点】存放所有的 .v 或 .sv 源代码（可综合设计）
│   ├── top.v                  # 顶层模块
│   ├── uart_tx.v              # 子模块
│   └── uart_rx.v
│
├── tb/ (或 sim/)              # 存放 Testbench（仿真测试文件，不可综合）
│   ├── tb_top.v
│   └── wave.do                # ModelSim 仿真波形脚本
│
├── ip/                        # 存放 Quartus IP 核 (如 .qip, .ip, .v 等生成文件)
│   └── pll_50m/
│
├── constr/ (或 constraints/)  # 存放物理约束和时序约束文件
│   ├── pin_assign.qsf         # 引脚分配（也可以通过 Quartus 界面直接配置在顶层 qsf 中）
│   └── timing.sdc             # 时序约束文件 (SDC)
│
├── output_files/              # Quartus 编译生成的最终输出文件（自动生成）
│   ├── My_FPGA_Project.sof    # 下载到 FPGA 的配置文件
│   ├── My_FPGA_Project.pof
│   └── 各种编译报告 .rpt
│
├── db/ 和 incremental_db/     # Quartus 编译产生的数据库文件（自动生成，极度占用空间）
│
└── doc/                       # 存放设计文档、数据手册、说明等
```

## 编译操作

- 全编译 Start Compilation
- 分析与综合 Analysis & Synthesis
- 布局布线 Fitter (Place & Route)
- 汇编器（生成程序文件）Assembler (Generate programming files)
- TimeQuest 时序分析 TimeQuest Timing Analysis
- EDA 网表生成器 EDA Netlist Writer

## 在 Quartus 中启动 ModelSim 仿真

菜单：Assignmments -> Settings

![](/images/posts/Quartus_create_project_07.png)


1. **保存并添加文件**：把 `key4_led4_press_tb.v` 保存好，然后像添加源码一样，把它添加到 Quartus 工程中（**注意：不要把它设为 Top-Level Entity**）。
2. **关联 Testbench**：
    - 点击菜单栏 `Assignments` -> `Settings...`
    - 左侧展开 `EDA Tool Settings`，点击 `Simulation`。
    - 确保 `Tool Name` 选的是 `ModelSim-Altera`。
    - 在下方 `NativeLink settings` 区域，勾选 **`Compile test bench`**，点击右侧的 **`Test Benches...`**。
    - 点击 `New...`，在 `Test bench name` 填入 `key4_led4_press_tb`，在 `Top level module` 填入 `key4_led4_press_tb`。
    - 点击 `File name` 右侧的 `...`，选中你的 `key4_led4_press_tb.v` 文件，点击 `Add`。
    - 一路点击 `OK` 保存。
3. **启动仿真**：
    - 先在 Quartus 里点击一下 `Analysis & Synthesis`（确保你的设计代码没问题）。
    - 点击菜单栏 **`Tools`** -> **`Run Simulation Tool`** -> **`RTL Simulation`**。
4. **在 ModelSim 中查看波形**：
    - ModelSim 会自动打开。
    - 在左侧的 `sim` 窗口找到 `key4_led4_press_tb`，右键点击里面的 `uut`，选择 **`Add Wave`**（或者直接在 `Objects` 窗口选中 `key` 和 `led` 拖进去）。
