课件地址:  
http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/05-machine-basics.pdf

MachineCode(机器码): The	byte-level programs	that	a	processor	executes 	
  
AssemblyCode(汇编码): A	text	representation of	machine	code 


### Turning C into Object Code

- Code in files p1.c p2.c
- Compile with command: gcc -Og p1.c p2.c -o p 使用gcc命令编译p1,p2
  - Use basic optimizations (-Og) 使用基本的优化
  - Put resulting binary in file p 编译结果输出到文件p  
  
### Compiling Into Assembly
C Code(sum.c)
  ```c
  long plus(long x, long y);
  
  void sumstore(long x, long y, long *dest){
    long t = plus(x,y);
    *dest = t;
  }
  ```
  obtain with command : gcc -Og -S sum.c 使用gcc命令编译sum.c生成汇编代码sum.s  
  produces file sum.s -S = Stop 终止后续步骤  
 
 Assembly Code
  ```asm
  sumstore:
    pushq %rbx
    moveq %rdx %rbx
    call plus
    moveq %rax %rbx
    popq %rbx
    ret
  ```
  
  
### Assembly Characteristics: Data Types
- "Integer" data of 1,2,4 or 8 bytes 汇编语言中的数据类型
  - Data values 数值
  - Addresses 内存地址
- Floating point data of 4,8 or 10 bytes 有专门寄存器处理Floating point data
- Code:Byte sequences encoding series of instructions 字节执行顺序说明
- No aggregate types such as arrays or structures 没有数组或者自定义struct类型
  - Just contiguously allocated bytes in memory 只有在内存中连续的数组
  
### Assembly Characteristics: Operations 运作方式
- Perform arithmetic function on register or memory data 在寄存器或者内存数据上执行算术函数
- Transfer data between memory and register 在内存和寄存器中间传输数据
  - Load data from memory into register 从内存加载数据到寄存器
  - Store register data into memory 将寄存器里面的数据存储到内存
- Transter control 控制传递
  - Unconditional jumps to/from procedures 跳入/跳出程序
  - Conditional branches 条件选择
  
### Object Code
- Assembler
  - Translates .s into .o 将.s结尾的汇编文件转换以.o文件结尾的目标代码
  - binary encoding of each instruction 将每个指令转换为二进制命令
  - Nearly-complete image of executable code 即将成为可执行代码
  - Missing linkages between code in different files 多个不同的目标文件之间缺少链接
- Linker
  - Resolves references between files 解决文件与文件直接的引用问题
  - Combines with static run-time libraries 结合运行期间的静态库
    - E.g., code for malloc,printf
  - Some libraries are dynamically linked 动态链接一些库
  
### C Code -> Assembly -> Object Code
```c
  *dest = t;
```
 ```asm
  moveq %rax , (%rbx)
 ``` 
 ```
  0x40059e: 48 89 03
 ```
- Assembly
  - Operands:
    t: Register %rax
    dest: Register %rbx
    *dest: Memory M[%rbx]

- Object Code
  - 3-bytes instruction 3个字节二进制指令
  - Stored at address 0x40059e 指令地址在0x40059e
  

### Disassembling Object Code
- Disassembler 
  - objdump -d sum 使用objdump命令反编译sum
  - Can be run on either a.out or o.file 
- Within gdb Debugger
  - gdb sum  
    disassemble sumstore
    
### Moving Data
- Operand Types
  - Immediate:立即数
    - Example:$0x400 , $-533
    - Like C constant, but prefixed with '$' 类似C的不变数,但是需要$符号
    - Encoded with 1,2 or 4 bytes 最大支持4字节long
- Register
  - Example:%rax,%r13
  - But %rsp reserved for special use 栈指针

 - Memory
  - Example:(%rax)
  - 8 consecutive bytes of memory at address given by register 内存中寄存器给定地址处的8个连续字节的
  
  
 ### moveq Operand Combinations
  ```C
  Imm -> Reg moveq $0x4,%rax  temp = 0x4;
  
  Imm -> Mem moveq -157,(%rax) *p =-157;
  
  Reg -> Reg moveq %rax,%rdx temp2 = temp1;
  
  Reg -> Mem moveq %rax,(%rdx) *p = temp1;
  
  Mem-> Reg moveq (%rax),%rdx temp = *p;
  ```
  
### Simple Memory Addressing Modes
- Normal (R) Mem[Reg[R]]
  - Register R specifies memory address 寄存器存了访问内存的具体地址
  - Pointer dereferencing in C
  - Example: moveq (%rcx),%rax
  
- Displacement D(R) Mem[Reg[R] + D] 寄存器移位操作
  - Register R specifies start of memory region 寄存器存在了访问内存的开始位置
  - Constant displacement D specifies offset 从起始位置偏移D
  - Example: moveq 8(%rbp),%rdx
  
### Complete Memory Addressing Modes
- most General Form D(Rb,Ri,S) Mem[Reg[Rb] + S*Reg[Ri] + D]
  - D:  Constant "displacement" 1,2 or 4 bytes 偏移位
  - Rb: Base Rwgister Any of 16 interger registers 开始位置
  - Ri: Index register Any,except for %rsp index
  - S:  Scale 1,2,4 or 8 步长
  
- Special Cases
  - (Rb,Ri)   Mem[Reg[Rb] + Reg[Ri]]
  - D(Rb,Ri)  Mem[Reg[Rb] + Reg[Ri]+ D]
  - (Rb,Ri,S) Mem[Reg[Rb] + S * Reg[Ri]]
  
