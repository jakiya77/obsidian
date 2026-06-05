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
