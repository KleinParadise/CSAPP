课件地址:  
http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/06-machine-control.pdf

### Single bit registers
- CF Carry Flag(for unsigned)
- SF Sign Flag(for signed)
- ZF Zero Flag
- OF Overflow Flag(for signed)

### Condition Codes(Implicit Setting)条件码隐式设置
- Implicitly set by arithmetic operations 通过算术运算隐式设置条件码
   - Example: addq Src,Dest <-> t = a + b  
     CF set if carry out from most significant bit  
     ZF set if t == 0  
     SF set if t < 0  
     OF set if two's-complement overflow  (a > 0 && b > 0 && t < 0) || (a < 0 && b < 0 && t >= 0)
     
### Condition Codes(Explicit Setting : Compare) 条件码显示设置通过Compare
