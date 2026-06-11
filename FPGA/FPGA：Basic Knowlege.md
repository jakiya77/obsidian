https://hdlbits.01xz.net/
# I Basics
## 1. concatenation 
![[pngPasted image 20260605153511.png|661]]
```verilog
module top_module( 
    input a,b,c,
    output w,x,y,z );
    assign {w,x,y,z} = {a,b,b,c};
endmodule
```
## 2. Gate（notgate andgat）
![[png：Pasted image 20260605153725.png|346]]![[png：Pasted image 20260605154307.png|355]]
![[png：Pasted image 20260605154644.png|350]]![[png：Pasted image 20260605155805.png|338]]
Verilog has separate bitwise-NOT (~) and logical-NOT (!) operators
Verilog has separate bitwise-AND (&) and logical-AND (&&) operators
Verilog has separate bitwise-OR (`|`) and logical-OR (`||`) operators
The bitwise-XOR operator is `^`. There is no logical-XOR operator.

| **运算符类型**   | **符号**              | **关注点**      | **输入/输出宽度**                | **常用场景**                    |
| ----------- | ------------------- | ------------ | -------------------------- | --------------------------- |
| **Bitwise** | `~`, `&`, `\|`, `^` | 数据的**每一位**   | 保持原数据的位宽                   | 硬件连线、掩码操作、总线信号按位处理          |
| **Logical** | `!`, `&&`, `\|\|``  | 整个数据的**真/假** | 输出永远只有 **1 位** (`0` 或 `1`) | `if` 语句或 `case` 语句的**条件判断** |
## 3 wire declaration 
![[pngPasted image 20260605162303.png|487]]
```verilog
`default_nettype none
module top_module(
    input a,
    input b,
    input c,
    input d,
    output out,
    output out_n   ); 
    wire and_1;
    wire and_2;

   
    assign and_1 = a & b;
    assign and_2 = c & d;
    assign out =  and_1|and_2;
    assign out_n = ~(and_1|and_2);
    
   
endmodule

```
>[!hint] Be aware of the number of wires.

## 4 7458

![[png：Pasted image 20260605165630.png|235]]

# II、Vectors
## 1、Vectors definition
![[pngPasted image 20260605183126.png]]
```verilog
module top_module ( 
    input wire [2:0] vec,
    output wire [2:0] outv,
    output wire o2,
    output wire o1,
    output wire o0  ); // Module body starts after module declaration
    assign o2 = vec[2];
    assign o1 = vec[1];
    assign o0 = vec[0];
    assign outv = vec;

endmodule
```
>[!hint] + assign out = my_vector[10]; // Part-select one bit out of the vector

## 2、Vectors debug
>[!tldr]+ 
>1. The endianness of a vector is normmally defined as  [upper:lower] `wire [3:0] vec;`
>2. default_nettype none     // Disable implicit nets. Reduces some types of bugs.
>3.  reg [7:0] mem [255:0];   // 256 unpacked elements, each of which is a 8-bit packed vector of reg. just a 8$\times$ 256 matrix  
>4.Part-Select: 

## 3、vector replication and sign-extending
```verilog
assign out = { {4{in[3]}}, in }; //4'b**1**101 (-3) to 8 bits results in 8'b**1111**1101
```
## 4、Pairwise Comparison
![[png：Pasted image 20260605203957.png]]
```verilog
module top_module (
    input a, b, c, d, e,
    output [24:0] out );//

    assign out = ~({5{a,b,c,d,e}})^{{5{a}},{5{b}},{5{c}},{5{d}},{5{e}}};

endmodule

```

# III、Modules
---
## 1. Instantiation
在设计层次化电路时，将子模块（小芯片）引入到顶层模块（大电路板）的操作叫做**例化（Instantiation）**。根据子模块引脚的定义方式，连线分为以下两种情况：

---

### Named Ports

 语法格式
`.子模块引脚名(你大板子的导线名)`
```verilog
// 假设子模块声明为：module mod_a (input in1, input in2, output out);

mod_a uut (
    .in1(a),      // 把大板子的 a 导线，插进芯片的 in1 引脚
    .in2(b),      // 把大板子的 b 导线，插进芯片的 in2 引脚
    .out(out_sig) // 把大板子的 out_sig 导线，插进芯片的 out 引脚
);
```

### Positional Ports
🔹 语法格式
`子模块名 实例名 (导线1, 导线2, 导线3, ...);`
Verilog

```verilog
// 假设子模块声明为：module mod_a (output, output, input, input, input, input);

mod_a uut (out1, out2, a, b, c, d); 
```

## 2. Module shift
![[png：Pasted image 20260605223203.png]]
```verilog
module top_module ( input clk, input d, output q );
    wire w1 ;
    wire w2;
    my_dff uut1 (
        .clk(clk),
        .d(d),
        .q(w1)
    );
    my_dff uut2 (
        .clk(clk),
        .d(w1),
        .q(w2)
    );
    my_dff uut3 (
        .clk(clk),
        .d(w2),
        .q(q)
    );
endmodule
```
![[png：Pasted image 20260606140606.png|717]]
```verilog
module top_module ( 
    input clk, 
    input [7:0] d, 
    input [1:0] sel, 
    output reg [7:0] q 
);


    wire [7:0] w1;
    wire [7:0] w2;
    wire [7:0] w3; 
  
    my_dff8 u_my_dff8_1(
        .clk(clk),
        .d(d),
        .q(w1)
    );
    
    my_dff8 u_my_dff8_2(
        .clk(clk),
        .d(w1),
        .q(w2)
    );
    
    my_dff8 u_my_dff8_3(
        .clk(clk),
        .d(w2),
        .q(w3) // 触发器输出给 w3，不要直接给 q
    );

    always @(*) begin
        case(sel)
            2'b00: q = d;  
            2'b01: q = w1; 
            2'b10: q = w2; 
            2'b11: q = w3; 
        endcase
    end

endmodule

```

## 3. Module add
![[png：Pasted image 20260606142425.png]]
```verilog

module top_module(
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);
    wire w1; wire [15:0] w2; wire [15:0] w3; // the width of the vector is 16
    add16 u_add16_1(
        .a(a[15:0]),
        .b(b[15:0]),
        .cin(1'b0),
        .cout(w1),
        .sum(w2)
    );
    
    add16 u_add16_2(
        .a(a[31:16]),
        .b(b[31:16]),
        .cin(w1),
        .sum(w3)
    );
    
    assign sum = {w3,w2};
    
endmodule

```
>[!hint]- debug log
>1. the width of the vector is 16 
>2. the input of the adder is not 32 

![[png：Pasted image 20260606151114.png|290]]![[png：Pasted image 20260606203335.png|408]]

>[!hint]- Full adder equations
sum = a ^ b ^ cin  
cout = a&b | a&cin | b&cin

``` verilog

module top_module(
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);
    wire w1;wire w2;
    wire [15:0] s1;wire [15:0] s2;wire [15:0] s3;wire [15:0] s4;
    
    add16 u_add16_1(
        .a(a[15:0]),
        .b(b[15:0]),
        .cin(1'b0),
        .cout(w1),
        .sum(s1)
    );
    add16 u_add16_2(
        .a(a[31:16]),
        .b(b[31:16]),
        .cin(1'b0),
        .sum(s2)
    );
    add16 u_add16_3(
        .a(a[31:16]),
        .b(b[31:16]),
        .cin(1'b1),
        .sum(s3)
    );
    
    always@(*)begin
        case(w1)
            1'b0:s4 = s2;
            1'b1:s4 = s3;
        endcase
    end
    assign sum = {s4,s1};
                
endmodule

```
>[!warning] + ⚠️ input vector width

![[png：Pasted image 20260606212400.png]]
```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    input sub,
    output [31:0] sum
);
    wire w1;wire [15:0] s1;wire [15:0] s2;
    add16 u_add16_1(
        .a(a[15:0]),
        .b(w_xorout[15:0]),
        .cin(sub),
        .cout(w1),
        .sum(s1)
    );
    add16 u_add16_2(
        .a(a[31:16]),
        .b(w_xorout[31:16]),
        .cin(w1),
        .sum(s2)
    );
    
    wire [31:0] w_xorout;
    assign w_xorout = b^{32{sub}};
    assign sum = {s2,s1};

endmodule


```
## 4. 1 bit adder
```verilog
assign sum = a^b^cin;
assign cout = (a & b) | (a & cin) | (b & cin);

assign {cout, sum} = a + b + cin;//自取高低位

```
# IV、Procedures
## 1、Alwaysblock1
>[!hint]+ 
>- Combinational: always @(*) 
>- Clocked: always @(posedge clk)
>- The left-hand-side of an assign statement must be a _net_ type (e.g., wire)
>- The left-hand-side of a procedural assignment (in an always block) must be a _variable_ type (e.g., reg)

## 2、Alwaysblock2
#### Blocking vs. Non-Blocking Assignment
There are three types of assignments in Verilog:
- **Continuous** assignments (assign x = y;). Can only be used when **not** inside a procedure ("always block").
- Procedural **blocking** assignment: (x = y;). Can only be used inside a procedure.
- Procedural **non-blocking** assignment: (x <= y;). Can only be used inside a procedure.
![[png：Pasted image 20260606220653.png|411]]
```verilog

// synthesis verilog_input_version verilog_2001
module top_module(
    input clk,
    input a,
    input b,
    output wire out_assign,
    output reg out_always_comb,
    output reg out_always_ff   );
    
    assign out_assign = a^b ;
    always@(*)begin
        out_always_comb = a^b ;
    end
    always@(posedge clk )begin
        out_always_ff <= a^b ;
    end

endmodule
```
## 3 Always casez
```verilog
// synthesis verilog_input_version verilog_2001
module top_module (
    input [7:0] in,
    output reg [2:0] pos 
);

    always @(*) begin
        casez(in) // 👈 必须用 casez，开启模糊通配符功能
            8'b???????1: pos = 3'd0; // 只要最低位 in[0] 是 1，位置就是 0
            8'b??????10: pos = 3'd1; // in[0]=0 且 in[1]=1，位置就是 1
            8'b?????100: pos = 3'd2; // 位置 2
            8'b????1000: pos = 3'd3; // 位置 3
            8'b???10000: pos = 3'd4; // 位置 4
            8'b??100000: pos = 3'd5; // 位置 5
            8'b?1000000: pos = 3'd6; // 位置 6
            8'b10000000: pos = 3'd7; // 最高位 in[7]=1，位置就是 7
            default:     pos = 3'd0; // 全 0 或其他情况，按题目要求输出 0
        endcase
    end
    
endmodule
```
## 4  Always nolatches
>[!hint]+ 把默认值写在前面而不是通过default，能 100% 避免意外生成Latch且大幅减少代码量

```Verilog
always @(*) begin
    // 1. 先给出所有输出的默认安全状态
    up = 1'b0; down = 1'b0; left = 1'b0; right = 1'b0;
    
    // 2. 然后根据条件，覆盖特定的值
    case (scancode)
        16'hE075: up = 1'b1;     // 只需要写发生变化的那一项
        16'hE072: down = 1'b1;
    endcase
end
```

- **兜底保障：** 在代码执行进入 `case` 之前，所有变量都已经有了明确的值（全 0）。当匹配到 `16'hE075` 时，只有 `up` 被重新赋值为 `1`。至于 `down`、`left`、`right`，综合器会向前追溯，发现它们在开头已经被赋值为 `0` 了。
### 2、如果在最后添加default也是4行代码 会出现什么问题
```Verilog
always @(*) begin
    case (scancode)
        16'hE075: up = 1'b1;        // 危险！这里没有给 down, left, right 赋值
        16'hE072: down = 1'b1;      // 危险！这里没有给 up, left, right 赋值
        default: begin
            up = 1'b0; down = 1'b0; left = 1'b0; right = 1'b0;
        end
    endcase
end
```

- 当 `scancode` 是 `16'hE075` 时，综合器看到了 `up = 1'b1`。
    
- 然后它会问：“那在这个条件下，`down`、`left`、`right` 应该是什么状态？”
    
- 因为你在这个分支里没写，Verilog 的硬件映射规则是：**保持原值**。
    
- 为了在纯组合逻辑中“保持原值”，综合器只能被迫为你生成 3 个透明锁存器（Latches）来记住上一个时钟周期的状态。这在组合逻辑设计中通常是致命的，会导致毛刺（Glitch）和时序分析失败。

### 3. 如果硬要用 `default` 做到等效？

如果非要只用 `case` 来写且不生成 Latch，你必须在**每一个** `case` 分支里，把**所有**变量都写全：

```Verilog
always @(*) begin
    case (scancode)
        16'hE075: begin up = 1'b1; down = 1'b0; left = 1'b0; right = 1'b0; end // 极其繁琐
        16'hE072: begin up = 1'b0; down = 1'b1; left = 1'b0; right = 1'b0; end
        // ... 中间省略几十个按键 ...
        default:  begin up = 1'b0; down = 1'b0; left = 1'b0; right = 1'b0; end
    endcase
end
```

# V、More Vrilog Features
## 1 Conditional
>[!hint]+ (condition ? if_true : if_false)

## 2 reduction 
>[!hint]+ reduction operator
- & a[3:0]     // AND: a[3]&a[2]&a[1]&a[0]. Equivalent to (a[3:0] == 4'hf) 
- | b[3:0]     // OR:  b[3]|b[2]|b[1]|b[0]. Equivalent to (b[3:0] != 4'h0)
- ^ c[2:0]     // XOR: c[2]^c[1]^c[0]
1 parity check :  assign parity = ^in;

## 3 for-loop

``` Verilog
integer i; // 👈 必须在块外面（或者块开头）声明一个整数型变量 i

always @(*) begin
    for (i = 0; i < 100; i = i + 1) begin
        // 👈 在这里写你要批量复制的连线逻辑
    end
end
```
 Given a 100-bit input vector [99:0], reverse its bit ordering.
```verilog
module top_module( 
    input [99:0] in,
    output [99:0] out
);
    int i;
    always@(*)begin
        for(i = 0;i<100;i=i+1)begin
            out[99-i] = in[i];
        end
    end
endmodule

```
Build a population count circuit for a 255-bit input vector.
```verilog
module top_module( 
    input [254:0] in,
    output [7:0] out );
    int i;
    
    always@(*)begin
        out = 0;
        for(i=0;i<255;i=i+1)begin
            if(in[i]==1)begin
                out = out + 1;
            end
            end
    end

endmodule
```
## 4 instance array
```verilog

module top_module( 
    input [99:0] a, b,
    input cin,
    output [99:0] cout,
    output [99:0] sum 
);
    
    wire [99:0] middle_wire;
    assign middle_wire = {cout[98:0], cin}; // ‼️⚠️
    
    fadd u_fadd[99:0](
        .a(a),
        .b(b),
        .cin(middle_wire),
        .cout(cout),
        .sum(sum)
    );

endmodule 



module fadd(
    input a, b,
    input cin,
    output cout,
    output sum 
);
        
    
    assign sum = a ^ b ^ cin;
    assign cout = (a & b) | (a & cin) | (b & cin);
        
endmodule 
```
![[png：ChatGPT Image 2026年6月7日 16_17_40.png]]![[png：Pasted image 20260607162259.png]]

# VI、 Circuit - combination logic
## 1、通过真值表写电路
1. 找 f=1 的行  
2. 每一行写一个 AND  0 写取反，1 写原变量  
3. 所有 AND 用 OR 连起来  
4. 有能力再化简
题目真值表是：

|x3|x2|x1|f|
|---|---|---|---|
|0|0|0|0|
|0|0|1|0|
|0|1|0|1|
|0|1|1|1|
|1|0|0|0|
|1|0|1|1|
|1|1|0|0|
|1|1|1|1|

输出 `f=1` 的行是：
010  →  ~x3 &  x2 & ~x1
011  →  ~x3 &  x2 &  x1
101  →   x3 & ~x2 &  x1
111  →   x3 &  x2 &  x1

assign f = (~x3 &  x2 & ~x1) |
           (~x3 &  x2 &  x1) |
           ( x3 & ~x2 &  x1) |
           ( x3 &  x2 &  x1);

![[png：Pasted image 20260607170102.png|497]]
```verilog
module top_module (input x, input y, output z);
    wire w1;wire w2; wire w3;wire w4;
    IA_module u_IA_module_1(.x(x),.y(y),.z(w1));
    IB_module u_IB_module_1(.x(x),.y(y),.z(w2));
    IA_module u_IA_module_2(.x(x),.y(y),.z(w3));
    IB_module u_IB_module_2(.x(x),.y(y),.z(w4));
    
    assign z = (w1|w2)^(w3&w4);


endmodule

module IA_module (input x, input y, output z);
    assign z = (x^y) & x;
endmodule

module IB_module (input x, input y, output z);
    assign z = ~(x^y);
endmodule
```
>[!hint]+ jakiya bravo!
## 2、vector的灵活运用 错位求值
>[!example]+ You are given a four-bit input vector in[3:0]. We want to know some relationships between each bit and its neighbour:
>- **out_both**: Each bit of this output vector should indicate whether _both_ the corresponding input bit and its neighbour to the **left** (higher index) are '1'. For example, out_both[2] should indicate if in[2] and in[3] are both 1. Since in[3] has no neighbour to the left, the answer is obvious so we don't need to know out_both[3].
>- **out_any**: Each bit of this output vector should indicate whether _any_ of the corresponding input bit and its neighbour to the **right** are '1'. For example, out_any[2] should indicate if either in[2] or in[1] are 1. Since in[0] has no neighbour to the right, the answer is obvious so we don't need to know out_any[0].
>- **out_different**: Each bit of this output vector should indicate whether the corresponding input bit is different from its neighbour to the **left**. For example, out_different[2] should indicate if in[2] is different from in[3]. For this part, treat the vector as wrapping around, so in[3]'s neighbour to the left is in[0].

```verilog

module top_module( 
    input [3:0] in,
    output [2:0] out_both,
    output [3:1] out_any,
    output [3:0] out_different );

    assign out_both[2:0] = {in[3]&in[2],in[2]&in[1],in[1]&in[0]};
    assign out_any[3:1] = {in[3]|in[2],in[2]|in[1],in[1]|in[0]};
    assign out_different[3:0] = {in[3]^in[0],in[2]^in[3],in[1]^in[2],in[1]^in[0]};
endmodule

```
```verilog
assign out_both = in[2:0] & in[3:1];
assign out_any  = in[3:1] | in[2:0];
assign out_different = in ^ {in[0], in[3:1]};
```

## 3、Multiplexer
### 1 vetor  Mux256to1v
```verilog
// vector[start +: width] 
//从 start 开始，往高位取 width 位
in[8 +: 4]   // in[11:8]`
// vector[start -: width] 
//从 start 开始，往低位取 width 位`
in[11 -: 4]  // in[11:8]`
```
### 2 Full Adder -> 3bit adder
#debug
![[png：ChatGPT Image 2026年6月7日 21_21_55.png|426]]![[png：Pasted image 20260607211053.png|216]]

```verilog
module top_module( 
    input [2:0] a, b,
    input cin,
    output [2:0] cout,
    output [2:0] sum );
    
    wire w1;wire w2;
    wire s1;wire w3;wire s2;wire s3;
    
    // 报错关键
    fadd u_fadd_1(.a(a),.b(b),.cin(cin),.cout(w1),.sum(s1));
    fadd u_fadd_2(.a(a),.b(b),.cin(w1),.cout(w2),.sum(s2));
    fadd u_fadd_3(.a(a),.b(b),.cin(w2),.cout(w3),.sum(s3));
    
    //需要修改成 按位给就行了
    
     fadd u0 (
        .a(a[0]),
        .b(b[0]),
        .cin(cin),
        .cout(cout[0]),
        .sum(sum[0])
    );

    fadd u1 (
        .a(a[1]),
        .b(b[1]),
        .cin(cout[0]),
        .cout(cout[1]),
        .sum(sum[1])
    );

    fadd u2 (
        .a(a[2]),
        .b(b[2]),
        .cin(cout[1]),
        .cout(cout[2]),
        .sum(sum[2])
    );
    //
    assign cout = w3;
    assign sum = s1+s2+s3;

endmodule

module fadd( 
    input a, b, cin,
    output cout, sum );
    assign{cout,sum} = a+b+cin;
endmodule
```
>[!warning]+ 又一次 由于width出错了！！
>fulladder是一位的 不能直接给

![[png：Pasted image 20260607212433.png]]
```verilog
module top_module (
    input [3:0] x,
    input [3:0] y,
    output [4:0] sum
);

    wire c1, c2, c3;

    FA u0 (
        .a(x[0]),
        .b(y[0]),
        .cin(1'b0),
        .sum(sum[0]),
        .cout(c1)
    );

    FA u1 (
        .a(x[1]),
        .b(y[1]),
        .cin(c1),
        .sum(sum[1]),
        .cout(c2)
    );

    FA u2 (
        .a(x[2]),
        .b(y[2]),
        .cin(c2),
        .sum(sum[2]),
        .cout(c3)
    );

    FA u3 (
        .a(x[3]),
        .b(y[3]),
        .cin(c3),
        .sum(sum[3]),
        .cout(sum[4])
    );

endmodule


module FA (
    input a,
    input b,
    input cin,
    output cout,
    output sum
);

    assign {cout, sum} = a + b + cin;

endmodule

```

## 4、 Karnaugh map to circuit 卡诺图化简

![[png：Pasted image 20260607222858.png]]

# VII、Circuit - Sequential Logic

## 1、同、异步复位

```verilog
module top_module (
    input clk,
    input reset,            // Synchronous reset
    input [7:0] d,
    output [7:0] q
);
    always@(posedge clk or posedge reset)begin
        if(reset)begin
            q = 0;
        end
        else begin
        q = d;
    end
    end

endmodule

module top_module (
    input clk,
    input reset,            // Synchronous reset
    input [7:0] d,
    output [7:0] q
);
    always@(posedge clk )begin //只有这里写法不一样 差了一个时钟
        if(reset)begin
            q = 0;
        end
        else begin
        q = d;
    end
    end

endmodule

```
![[png：Pasted image 20260607232359.png]]

![[png：Pasted image 20260607232446.png]]

同步复位：`always @(posedge clk)`，只有时钟上升沿到来时才检查 `reset`，所以 `reset=1` 后不会立刻清零，要等下一个时钟沿。

异步复位：`always @(posedge clk or posedge reset)`，`reset` 自己也能触发 always 块，所以 `reset` 一变高，`q` 立刻清零。

题目要求同步复位，所以不能把 `reset` 写进敏感列表，否则仿真里 `q` 会提前清零，和参考波形 mismatch。

## 2、Edge
### 2.1 Edgedetect
```verilog
module top_module (
    input clk,
    input [7:0] in,
    output [7:0] pedge，// pose edge only
    output [7:0] anyedge//both edge
    );
    reg [7:0]in_last;
    always@(posedge clk)begin
        pedge <= ~in_last & in;
        anyedge <= in_last^in;
        in_last <= in;
    end

endmodule
```
![[png：Pasted image 20260608095319.png]]![[png：Pasted image 20260608095333.png]]
### 2.2 Edge capture
For each bit in a 32-bit vector, capture when the input signal changes from 1 in one clock cycle to 0 the next. "Capture" means that the output will remain 1 until the register is reset (synchronous reset).

Each output bit behaves like a SR flip-flop: The output bit should be set (to 1) the cycle after a 1 to 0 transition occurs. The output bit should be reset (to 0) at the positive clock edge when reset is high. If both of the above events occur at the same time, reset has precedence. In the last 4 cycles of the example waveform below, the 'reset' event occurs one cycle earlier than the 'set' event, so there is no conflict here.

In the example waveform below, reset, in[1] and out[1] are shown again separately for clarity.
```verilog

module top_module (
    input clk,
    input reset,
    input [31:0] in,
    output [31:0] out
);
    reg [31:0]  in_last ;
    always@(posedge clk)begin
        in_last <= in;
        if(reset)begin
            out <= 32'b0;
        end
        else begin
            out <= out | (in_last & ~in);// ⚠️ 这个部分要用out锁住一下当前的值
        end
    end
    

endmodule

```
![[png：Pasted image 20260608100015.png

### 2.3 Dualedge
Build a circuit that functionally behaves like a dual-edge triggered flip-flop:
![[png：Pasted image 20260608101730.png|476]]
>[!warning]+  @(posedge clk or negedge clk) is illegal

```verilog
module top_module (
    input clk,
    input d,
    output q
);
    reg q_pos;
    reg q_neg;

    always @(posedge clk) begin
        q_pos <= d;
    end

    always @(negedge clk) begin
        q_neg <= d;
    end

    assign q = clk ? q_pos : q_neg; //很巧妙

endmodule

```

## 3、counter
Make a decade counter that counts 1 through 10, inclusive. The reset input is synchronous, and should reset the counter to 1.
![[png：Pasted image 20260608104501.png|495]]
>[!caution]+ 我写的太绕了

```verilog
//我写的
module top_module (
    input clk,
    input reset,        // Synchronous active-high reset
    output reg [3:0] q
);

    always @(posedge clk) begin
        if (reset) begin
            q <= 4'd0;
        end
        //这个部分有点绕
        else if (q < 9) begin
            q <= q + 4'd1;
        else begin
            q <= q - 10 + 1;
       //可以直接写成q == 4'd9 清零
        end
    end
    
    //修改版的
module top_module (
    input clk,
    input reset,        // Synchronous active-high reset
    output reg [3:0] q
);

    always @(posedge clk) begin
        if (reset) begin
            q <= 4'd0;
        end
        else if (q == 4'd9) begin
            q <= 4'd0;
        end
        else begin
            q <= q + 4'd1;
        end
    end

endmodule


```

Build a decade counter that counts from 0 through 9, inclusive, with a period of 10. The reset input is synchronous, and should reset the counter to 0. We want to be able to pause the counter rather than always incrementing every clock cycle, so the slowena input indicates when the counter should increment.
![[png：Pasted image 20260608105859.png|400]]
```verilog
module top_module (
    input clk,
    input slowena,
    input reset,
    output [3:0] q);
    
    always@(posedge clk)begin
        if(reset)begin
            q <= 4'd0;
        end
        else if (slowena)begin
            q <= (q == 4'd9)?(4'd0):(q+1);
        end
        else begin
            q<=q;
        end
    end

endmodule
```
>[!hint]+ bravo jakiya!
### 3.1、 Exams 1000 Hz clock -> 1 Hz signal
![[png：Pasted image 20260608114207.png|377]]
```verilog
module top_module (
    input clk,
    input reset,
    output OneHertz,
    output [2:0] c_enable
);

    wire [3:0] q0, q1, q2;

    assign c_enable[0] = 1'b1;
    assign c_enable[1] = (q0 == 4'd9);
    assign c_enable[2] = (q0 == 4'd9) && (q1 == 4'd9);

    assign OneHertz = (q0 == 4'd9) && (q1 == 4'd9) && (q2 == 4'd9);

    bcdcount counter0 (
        .clk(clk),
        .reset(reset),
        .enable(c_enable[0]),
        .Q(q0)
    );

    bcdcount counter1 (
        .clk(clk),
        .reset(reset),
        .enable(c_enable[1]),
        .Q(q1)
    );

    bcdcount counter2 (
        .clk(clk),
        .reset(reset),
        .enable(c_enable[2]),
        .Q(q2)
    );

endmodule
```
### 3.2 4 digit decimal counter
Build a 4-digit BCD (binary-coded decimal) counter. Each decimal digit is encoded using 4 bits: q[3:0] is the ones digit, q[7:4] is the tens digit, etc. For digits [3:1], also output an enable signal indicating when each of the upper three digits should be incremented.

You may want to instantiate or modify some one-digit [decade counters](https://hdlbits.01xz.net/wiki/Count10 "Count10").

![[png：Pasted image 20260608121223.png]]
```verilog
module top_module (
    input clk,
    input reset,   // Synchronous active-high reset
    output [3:1] ena,
    output [15:0] q
);

    assign ena[1] = (q[3:0] == 4'd9);
    assign ena[2] = (q[3:0] == 4'd9) && (q[7:4] == 4'd9);
    assign ena[3] = (q[3:0] == 4'd9) && (q[7:4] == 4'd9) && (q[11:8] == 4'd9);

    one_digit u0 (
        .clk(clk),
        .reset(reset),
        .ena(1'b1),
        .q(q[3:0])
    );

    one_digit u1 (
        .clk(clk),
        .reset(reset),
        .ena(ena[1]),
        .q(q[7:4])
    );

    one_digit u2 (
        .clk(clk),
        .reset(reset),
        .ena(ena[2]),
        .q(q[11:8])
    );

    one_digit u3 (
        .clk(clk),
        .reset(reset),
        .ena(ena[3]),
        .q(q[15:12])
    );

endmodule


module one_digit (
    input clk,
    input reset,
    input ena,
    output reg [3:0] q
);

    always @(posedge clk) begin
        if (reset)
            q <= 4'd0;
        else if (ena)
            q <= (q == 4'd9) ? 4'd0 : q + 4'd1;
    end

endmodule

```

### 3.3 12h clock
Create a set of counters suitable for use as a 12-hour clock (with am/pm indicator). Your counters are clocked by a fast-running clk, with a pulse on ena whenever your clock should increment (i.e., once per second).

reset resets the clock to 12:00 AM. pm is 0 for AM and 1 for PM. hh, mm, and ss are two **BCD** (Binary-Coded Decimal) digits each for hours (01-12), minutes (00-59), and seconds (00-59). *Reset has higher priority than enable, and can occur even when not enabled.*
>[!hints]- Reset has higher priority than enable
>意味着需要先写 而不是写成两个并列的if结构
>if (reset) begin
end
else if (ena) begin
end


The following timing diagram shows the rollover behaviour from 11:59:59 AM to 12:00:00 PM and the synchronous reset and enable behaviour.

Note that 11:59:59 PM advances to 12:00:00 AM, and 12:59:59 PM advances to 01:00:00 PM. There is no 00:00:00.

不要弄特别复杂 就是简单的 ss --> mm --> hh就可以了

![[png：Pasted image 20260608121109.png]]
```verilog
module top_module(
    input clk,
    input reset,
    input ena,
    output reg pm,
    output reg [7:0] hh,
    output reg [7:0] mm,
    output reg [7:0] ss
); 

    always @(posedge clk) begin
        if (reset) begin
            hh <= 8'h12;
            mm <= 8'h00;
            ss <= 8'h00;
            pm <= 1'b0;      // AM
        end
        else if (ena) begin

            // seconds
            if (ss == 8'h59) begin
                ss <= 8'h00;

                // minutes
                if (mm == 8'h59) begin
                    mm <= 8'h00;

                    // hours
                    if (hh == 8'h11) begin
                        hh <= 8'h12;
                        pm <= ~pm;      // 11:59:59 -> 12:00:00 时 AM/PM 翻转
                    end
                    else if (hh == 8'h12) begin
                        hh <= 8'h01;
                    end
                    else if (hh[3:0] == 4'h9) begin
                        hh <= {hh[7:4] + 4'h1, 4'h0};  // 09 -> 10
                    end
                    else begin
                        hh <= hh + 8'h01;
                    end

                end
                else begin
                    if (mm[3:0] == 4'h9)
                        mm <= {mm[7:4] + 4'h1, 4'h0};
                    else
                        mm <= mm + 8'h01;
                end

            end
            else begin
                if (ss[3:0] == 4'h9)
                    ss <= {ss[7:4] + 4'h1, 4'h0};
                else
                    ss <= ss + 8'h01;
            end
        end
    end

endmodule
```

## 4、Shift Register
### 4.1、4 bit shift register
Build a 4-bit shift register (right shift), with asynchronous reset, synchronous load, and enable.

- areset: Resets shift register to zero.
- load: Loads shift register with data[3:0] instead of shifting.
- ena: Shift right (q[3] becomes zero, q[0] is shifted out and disappears).
- q: The contents of the shift register.

If both the load and ena inputs are asserted (1), the load input has higher priority.

```verilog
module top_module(
input clk,
input areset, // async active-high reset to zero
input load,
input ena,
input [3:0] data,
output reg [3:0] q);

always@(posedge clk or posedge areset)begin
if(areset)begin
q <= 4'b0;
end
else if (load)begin
q <= data;
end
else if(ena)begin
q <= {1'b0,q[3:0]};
end
endmodule
```

### 4.2、Rotate100
Build a 100-bit left/right rotator, with synchronous load and left/right enable. A rotator shifts-in the shifted-out bit from the other end of the register, unlike a shifter that discards the shifted-out bit and shifts in a zero. If enabled, a rotator rotates the bits around and does not modify/discard them.

- load: Loads shift register with data[99:0] instead of rotating.
- ena[1:0]: Chooses whether and which direction to rotate.
    - 2'b01 rotates right by one bit
    - 2'b10 rotates left by one bit
    - 2'b00 and 2'b11 do not rotate.
- q: The contents of the rotator.

