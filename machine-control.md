课件地址:  
http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/06-machine-control.pdf

### Single bit registers
- CF Carry Flag(for unsigned) 进位标志 
- SF Sign Flag(for signed) 符号标志
- ZF Zero Flag 零标志
- OF Overflow Flag(for signed) 溢出标志

### Condition Codes(Implicit Setting)条件码隐式设置
- Implicitly set by arithmetic operations 通过算术运算隐式设置条件码
   - Example: addq Src,Dest <-> t = a + b  
     CF set if carry out from most significant bit  
     ZF set if t == 0  
     SF set if t < 0  
     OF set if two's-complement overflow  (a > 0 && b > 0 && t < 0) || (a < 0 && b < 0 && t >= 0)
     
### Condition Codes(Explicit Setting : Compare) 条件码显示设置通过Compare
- Explicit Setting by Compare Instruction
    - Example: cmpq Src2,Src1
    - cmpq b,a like computing a-b without setting destination 注意a与b的顺序
     CF set if carry out from most significant bit  
     ZF set if a == b  
     SF set if (a - b) < 0  
     OF set if two's-complement overflow  (a > 0 && b < 0 && (a - b) < 0) || (a < 0 && b > 0 && (a - b) > 0)
     
### Condition Codes(Explicit Setting : Test) 条件码显示设置通过Test
- Explicit Setting by Test Instruction
- Example: testq Src2,Src1
    - testq b,a like computing a&b without setting destination (& = and)
    - Sets condition codes based on value of Src1 & Src2
    - Useful to have one of the operands be a mask 判断一个寄存器是否为空即test两个值都是自己  
    ZF set when a & b == 0  
    SF set when a & b < 0  
    
### Reading Condition Codes
- set low-order byte of sestination to 0 or 1 based on combinations of condition codes 根据条件码的组合将目标的低位字节设置为0或1
- does not alter remaining 7 bytes 不会更改剩余的7个字节
```cpp
int gt (long x, long y){
   return x > y
}
//AssemblyCode
cmpq %rsi,%rdi #compare x:y
setg %al #Set when >
movzbl %al,%eax #zero rest of %rax 其他位都设置为0
ret
```
