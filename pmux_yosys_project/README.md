多路选择器 Yosys 逻辑优化实验工程
实验环境：Yosys 0.9，目标 FPGA：iCE40 系列。
本工程完成多种多路选择器 RTL 设计的逻辑综合、逻辑优化对比以及 FPGA 工艺映射资源统计。
1  工程目录结构
工程采用如下目录结构，将 Verilog 源码、Yosys 综合脚本、运行日志与说明文档分类存放：
pmux_yosys_project/
├── rtl/                    # Verilog 原始源码
│   ├── pmux_case01_balanced.v
│   ├── pmux_case02_pipeline.v
│   ├── pmux_case03_crossbar.v
│   ├── pmux_case04_feedback.v
│   ├── pmux_case05_nested.v
│   ├── test1.v
│   ├── test2.v
│   ├── test3.v
│   └── test4.v
├── scripts/                # Yosys 综合脚本
│   ├── pmux_all.ys         # 完整 synth_ice40 工艺映射综合脚本
│   ├── pmux_unopt_rtl.ys  # RTL 未优化脚本：仅 proc 展开
│   └── pmux_opt_rtl.ys    # RTL 逻辑优化脚本：proc+opt/fsm/memory
├── logs/                   # 运行输出日志文件
│   ├── log_opt.txt         # 完整 iCE40 综合日志
│   ├── log_rtl_unopt.txt   # RTL 未优化统计日志
│   └── log_rtl_opt.txt    # RTL 逻辑优化后统计日志
└── README.md               # 工程说明文档
其中各脚本作用如下：
pmux_all.ys：执行完整 synth_ice40 工艺映射综合，输出 FPGA 硬件资源、综合后网表、blif 文件及原理图 svg。
pmux_unopt_rtl.ys：仅对 Verilog 执行 proc 展开，不执行任何逻辑优化，用于获取未优化的 RTL 单元统计。
pmux_opt_rtl.ys：执行 proc 及 opt/fsm/memory 逻辑优化，但不做 FPGA 工艺映射，用于获取逻辑优化后的 RTL 单元统计。
2  运行环境要求
1. 使用 OSS-CAD-Suite，内置 Yosys 0.9 版本。
2. 综合脚本默认依赖共享目录路径 /mnt/hgfs/yosys_share/，若实际路径不一致，需要修改 ys 脚本内文件的读取路径。
3  复现实验步骤
切换到工程所在目录，依次执行以下命令复现实验结果。
3.1  RTL 未优化（仅行为展开，无逻辑优化）
yosys scripts/pmux_unopt_rtl.ys > logs/log_rtl_unopt.txt
3.2  RTL 逻辑优化（仅逻辑化简，不做 FPGA 工艺映射）
yosys scripts/pmux_opt_rtl.ys > logs/log_rtl_opt.txt
3.3  完整 iCE40 FPGA 工艺映射综合
输出综合后 Verilog 网表、blif 文件、svg 原理图及硬件资源统计：
yosys scripts/pmux_all.ys > logs/log_opt.txt
3.4  提取单元统计数据
# 提取 RTL 未优化单元
grep "Number of cells:" logs/log_rtl_unopt.txt
# 提取 RTL 优化后单元
grep "Number of cells:" logs/log_rtl_opt.txt
# 提取 FPGA 综合后硬件单元
grep "Number of cells:" logs/log_opt.txt
4  实验指标说明
4.1  逻辑面积缩减率（RTL 抽象层面）
计算公式为：逻辑面积缩减率 =（RTL 未优化 cells − RTL 优化后 cells）÷ RTL 未优化 cells × 100%。
该指标仅在 RTL 抽象单元粒度下计算，反映 Yosys 对 Verilog 源码的逻辑化简与冗余裁剪能力。
4.2  FPGA 硬件资源消耗
由 synth_ice40 工艺映射得到，对应 iCE40 硬件原语 SB_LUT4、SB_DFF、SB_CARRY 等硬件单元总数，代表实际 FPGA 芯片资源开销。
4.3  重要说明
1. RTL 抽象单元（$mux、$add、$eq）与 FPGA 硬件原语粒度不同，二者不可直接相减计算缩减率。
2. Yosys 0.9 版本不支持 stat-nodes 参数，本实验未统计逻辑节点数量。
3. Yosys 0.9 无法输出每个模块独立的优化耗时与内存峰值，仅可获取整套脚本的全局统计信息。
5  实验简要结论
1. 结构规整的多路选择器（balanced、pipeline、crossbar）源码冗余少，逻辑层面可优化空间很小，逻辑面积缩减率接近 0；工艺映射后算术、多路器逻辑会被拆分为大量 LUT 与进位链硬件单元，硬件资源占用明显上升。
2. 嵌套、反馈型多路选择器存在可合并的中间逻辑，优化器可消除部分冗余逻辑，获得中等程度的逻辑缩减。
3. test1-test4 测试用例含有大量冗余分支与重复表达式，Yosys 通过常量传播、死代码消除、等价单元合并实现较高的逻辑面积缩减率。
6  注意事项
1. 若虚拟机共享挂载路径改变，需要修改所有 ys 脚本中的读写文件路径。
2. svg 原理图生成较慢，若不需要原理图，可注释脚本中全部 show -prefix 行以加快运行速度。
3. 本仓库不包含 OSS-CAD-Suite 工具软件，使用者需自行准备 Yosys 0.9 运行环境。