---
title: 纯组合逻辑实战：三八译码器 (3-to-8 Decoder) 与 Vivado 仿真入门
date: {{date}}
tags:
  - FPGA
  - Verilog
  - Vivado
  - 数字逻辑
---

# 纯组合逻辑实战：三八译码器 (3-to-8 Decoder) 与 Vivado 仿真入门

> [!abstract] 导语 本笔记记录了使用 Vivado 从零编写纯组合逻辑电路（三八译码器）并完成行为级波形仿真的全过程。核心收获是从类似 MATLAB/C 的“时间顺序执行”软件思维，正式跨越到“空间并行结构”的底层硬件思维。

## 一、 核心概念与思维转换

### 1. 硬件思维的本质

- **并行执行：** Verilog 描述的是物理电路图。代码中的各个模块和并行赋值语句是同时发生的，而不是自上而下逐行运行。
    
- **黑盒思维：** 写代码相当于在造一个具有输入输出引脚的黑盒（Module），在脑海中需要有清晰的**端口连接拓扑图**。
    

### 2. 独热码 (One-Hot Code)

三八译码器的核心输出逻辑。8 根输出线中，**永远只有 1 根是高电平（1），其余 7 根全为低电平（0）**。随着输入的增加，唯一的 `1` 像对角线一样移位，而不是按普通二进制计数。

## 二、 设计文件 (Design Source)

> [!info] 模块功能 带有使能端 (`en`) 的 3位输入 (`in`) 转 8位输出 (`out`) 译码器。

### 1. 核心代码 (`decoder_3to8.v`)

Verilog

```
`timescale 1ns / 1ps

module decoder_3to8 (
    input  wire       en,   // 1位使能信号
    input  wire [2:0] in,   // 3位输入信号 (位宽规范：[MSB:LSB])
    output reg  [7:0] out   // 8位输出信号
);

    // 组合逻辑：敏感列表包含所有输入信号，任何输入变化都会触发重新计算
    always @(*) begin
        if (en == 1'b0) begin
            out = 8'b0000_0000; // 未使能时全灭
        end else begin
            case(in)
                3'b000: out = 8'b0000_0001; // 十进制 0
                3'b001: out = 8'b0000_0010; // 十进制 1
                3'b010: out = 8'b0000_0100; // 十进制 2
                3'b011: out = 8'b0000_1000; // 十进制 3
                3'b100: out = 8'b0001_0000; // 十进制 4
                3'b101: out = 8'b0010_0000; // 十进制 5
                3'b110: out = 8'b0100_0000; // 十进制 6
                3'b111: out = 8'b1000_0000; // 十进制 7
                default: out = 8'b0000_0000; // 硬件好习惯：必须加 default 兜底，防止生成锁存器(Latch)
            endcase
        end
    end
endmodule
```

### 2. 核心语法解析

- **位宽定义 `[2:0]`：** 遵循“大端模式 (MSB First)”。`[2:0]` 代表 `in[2]` 是最高位权重，`in[0]` 是最低位。坚决避免使用 `[0:2]`，防止逻辑反转错乱。
    
- **输出端口为何是 `reg`？** 在 Verilog 语法规定中，**任何在 `always` 块内被赋值的变量必须声明为 `reg`**。但这在纯组合逻辑（无时钟边沿触发）中，综合工具依然会将其映射为物理导线/逻辑门，而不会生成带有记忆功能的触发器（Flip-Flop）。
    

## 三、 仿真文件 / 测试台 (Testbench)

> [!info] Testbench 本质 Testbench (`tb_decoder_3to8.v`) 是一个封闭的物理实验室。它没有对外的输入输出端口（模块括号为空），我们在内部用信号发生器（`reg`）产生激励，喂给被测芯片（`uut`），并用示波器探头（`wire`）观察输出。

### 1. 核心代码 (`tb_decoder_3to8.v`)

Verilog

```
`timescale 1ns / 1ps

module tb_decoder_3to8(); // 实验室没有对外引脚

    // 激励信号：主动给 UUT 喂信号，在 initial 中赋值，必须是 reg
    reg       tb_en;
    reg [2:0] tb_in;
    
    // 观测信号：被动接收 UUT 的输出，必须是 wire
    wire [7:0] tb_out;

    // 实例化被测模块 (Unit Under Test)
    decoder_3to8 uut (
        .en  (tb_en),
        .in  (tb_in),
        .out (tb_out)
    );

    // 编写测试剧本（激励产生）
    initial begin
        // 初始状态
        tb_en = 1'b0;
        tb_in = 3'b000;
        #10; // 延时 10ns

        // 遍历所有状态
        tb_en = 1'b1;
        tb_in = 3'b000; #10;
        tb_in = 3'b001; #10;
        tb_in = 3'b010; #10;
        tb_in = 3'b011; #10;
        tb_in = 3'b100; #10;
        tb_in = 3'b101; #10;
        tb_in = 3'b110; #10;
        tb_in = 3'b111; #10;
        
        // 测试使能关闭
        tb_en = 1'b0;
        #10;
        
        $finish; // 结束仿真（注意此处的分号）
    end
endmodule
```

### 2. 接口类型反转原理

- **输入口：** 在 Design 中是 `wire`（被动接收），在 Testbench 中是 `reg`（主动驱动）。
    
- **输出口：** 在 Design 中是 `reg`（被 `always` 主动驱动），在 Testbench 中是 `wire`（被动观测）。
    

## 四、 Vivado 仿真与 Debug 避坑指南

### 1. 常见报错与解决（Debug 记录）

> [!bug] 报错 1: 缺少分号导致语法错误
> 
> - **现象：** `launch_simulation failed`，Messages 提示 `earlier errors`。
>     
> - **原因：** Verilog 行为语句末尾漏写分号（如 `$finish` 后）。
>     
> - **解决：** 补全分号并保存。
>     

> [!bug] 报错 2: 模块不嵌套（层级识别失败）
> 
> - **现象：** Simulation Sources 中，`tb` 文件与设计文件呈平级并列状态，无法进行测试验证。
>     
> - **原因：** Testbench 中例化的模块名与设计文件不一致，或 Vivado 未自动识别 Top 模块。
>     
> - **解决：** 检查拼写。右键点击 `tb` 文件 -> 选择 **"Set as Top"**，强制设为顶层。成功标志是 `tb` 文件名加粗，且展开后包含设计模块 `uut`。
>     

> [!bug] 报错 3: 灵异的 "cannot find port" (找不到端口)
> 
> - **现象：** Tcl Console 报错：`[VRFC 10-3180] cannot find port 'en' / 'in' / 'out'`。
>     
> - **排查 1：未保存文件。** 检查文件名标签卡上是否有 `*` 号，仿真器只读取硬盘上最后一次保存的内容（按 `Ctrl+S`）。
>     
> - **排查 2：中文符号干扰。** 检查是否有全角单引号或分号。
>     
> - **排查 3：同名空文件。** 检查是否误建了同名测试文件覆盖了设计文件。
>     
> - **终极杀招（清理缓存）：** 如果代码全对但依然报错，右键点击左侧导航栏的 `Run Simulation` -> 选择 **`Reset Simulation Run`** 清理“智障缓存”，然后再左键点击运行。
>     

### 2. 波形读取技巧

> [!tip] 完美观察波形的三板斧
> 
> 1. **全景适应 (Zoom Fit)：** 点击波形窗口上方“四个箭头向外”的图标，或选中波形区直接按键盘 **`F`** 键，看清全局时间轴。
>     
> 2. **展开总线：** 点击总线信号（如 `tb_out[7:0]`）左侧的 **`>`** 箭头，将其展开为 8 根物理连线，观察阶梯状独热码移位。
>     
> 3. **进制切换 (Radix)：** Vivado 默认显示多位总线为 **十六进制 (Hex)**（如 01, 02, 04, 08, 10, 20... 注意 10 即为 16）。可通过右键信号名 -> `Radix` -> 改为 `Binary` (二进制) 或 `Unsigned Decimal` (无符号十进制) 以直观核对。
>     

## 五、 进阶：纯 Wire 级实现

若强制要求输出口为 `wire` 类型，可摒弃 `always` 块，直接使用**连续赋值语句 (`assign`)** 实现数据流描述，底层综合出的门级电路与 `reg`+`case` 完全一致：

Verilog

```
module decoder_3to8_wire (
    input  wire       en,
    input  wire [2:0] in,
    output wire [7:0] out  // 此处可直接为 wire
);

    assign out = (en == 1'b0) ? 8'b0000_0000 :
                 (in == 3'b000) ? 8'b0000_0001 :
                 (in == 3'b001) ? 8'b0000_0010 :
                 /* ... 中间省略 ... */
                 (in == 3'b111) ? 8'b1000_0000 : 8'b0000_0000;

endmodule
```