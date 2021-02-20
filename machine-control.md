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
cmpq %rsi,%rdi    #compare x:y
setg %al          #Set when >
movzbl %al,%eax   #zero rest of %rax 其他位都设置为0
ret
```

### Jumping
- jX instructions
    - jump to different part of code depending on condition codes 根据条件码跳转到不同代码部分    
    - Example
      jX | condition | description
      ------------ | -------------  | -------------
      jmp | 1 | unconditional
      je | ZF | Equal/Zero


### Conditional Branch Example
```c
long adsdiff(long x, long y){
   long result;
   if(x > y)
      result = x - y;
   else
      result = y - x;
   reutrn result;
}

//asm
absdiff:
cmpq %rsi,%rdi #x:y
jle .L4
moveq %rdi,%rax
subq  %rsi,%rax
ret
.L4:
moveq %rsi,%rax
subq  %rdi,%rax
ret
```

### Using Conditional Moves
- Conditional Move instructions
    - Instruction supports:  
      if(Test) Dest <-- Src
    - Gcc tries to use them
      - but,only when known to be safe 

- Why
    - Branches are very disruptive to instruction flow through pipelines 分支对流水线中的指令流造成很大破坏
    - Conditional moves do not require control transfer 有条件的移动不需要控制转移


```C
//有条件判断跳转,先计算出两个结果,再根据条件进行赋值,避免跳转计算
val = Test ? Then_Expr : Else_Expr;

//Goto Version
result = Then_Expr;
eval = Else_Expr;
nt = !Test;
if(nt) result = eval;
return result;
```

### Conditional Move Example
```c
long adsdiff(long x, long y){
   long result;
   if(x > y)
      result = x - y;
   else
      result = y - x;
   reutrn result;
}

//asm
absdiff:
movq  %rdi,%rax #x
subq  %rsi,%rax #result = x - y
movq  %rsi,%rdx 
subq  %rdi,%rdx #eval = y -x 
cmpq  %rsi,%rdi #x:y
cmovle %rdx,%rdx # if <= ,result = eval
ret
```

### Bad Cases for Conditional Move 不能使用Conditional Move的情况
- Expensive Computations
    - Both values get computed
    - Only makes sense when computations are very simple //确保两个条件的计算不会太复杂  
```c
var = Test(x) ? Hard1(x) : Hard2(x);
```

- Risky Computations
    - Both values get computed
    - May have undesirable effects  
```c
var = p ? *p : 0; //p 可能为空不能直接取值
```

- Computations with side effects
    - Both values get computed
    - Must be side-effect free  
```c
var = x > 0 ? x*=7 : x +=3; //计算条件时改变了x的值
```

## Loop


### "Do-While" Loop Compilation 非转换 c to asm
```c
//goto version
long pcount_goto(unsigned long x){
   long result = 0;
   loop:
   result += x & 0x1;
   x >>= 1;
   if(x) goto loop;
   return result;
}

//asm
   movl $0,%eax      #result = 0
.L2:                 #loop:
   movq %rdi,%rdx
   andl $1,%edx      #t = x & 0x1
   addq %rdx,%rax    #result +=t
   shrq %rdi         # x>>1
   jne  .L2          #if(x) goto loop
   ret
```

### "Do-While" Translation #1 
- "jump-to-middle" translation 
- used with -Og 
- initial goto starts loops at test

```c
//c code
long pcount_while(unsigned long x){
   long result = 0;
   while(x){
      result += x & 0x1;
      x >>= 1;
   }
   return restult;
}

//goto version 类似汇编代码
long pcount_goto_jtm(unsigned long x){
   long result = 0;
   goto test;
   loop:
      result += x & 0x1;
      x >>= 1;
   test:
      if(x) goto loop;
   return result;   
}
```
 
