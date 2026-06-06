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

## 2. Module add
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
