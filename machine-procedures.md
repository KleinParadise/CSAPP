课件地址:  
http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/07-machine-procedures.pdf

### Stack
- Region of memory managed with stack discipline 通过栈来规范管理内存区域
- Grows toward lower addresses 向低位地址增长
- register %rsp contains lowest stack address 寄存器％rsp包含最低的堆栈地址
  - %rsp 指向栈顶的位置


### Stack:Push <Call会隐式调用Push>
- pushq Src
  - fetch operand at src 从Src中获取操作数(获取到call之后一个指令的地址)
  - Decrement %rep by 8  将寄存器递减8位
  - Write operand at address given by %rsp 在％rsp给定的地址上写入操作数(call之后一个指令的地址压入栈中)



### Stack:Pop <Ret会隐式调用Pop>
- popq Dest
  - Read value at address given by %rsp 读取％rsp给定地址的值(从%rsp中读取下一个指令的执行地址)
  - Increment %rep by 8  将寄存器递增8位
  - Stroe value at Dest(must be register) 把下一个指令的执行地址写入Dest(即写入%rip)


### Procedure Control Flow
- Use stack to support procedure call and return 使用栈来支持运行时的调用和返回
- Procedure call
  - push return address on stack (return address = address of the next instruction right after call ) 将调用后下一条指令的地址压入栈中
  - jump to lable 跳转到新函数

- Procedure return:ret
  - pop address from stack 将调用后下一条指令的地址从栈中取出
  - jump to address 跳转到调用后下一条指令的地址



### Code Examples
```c
void multstore
 (long x, long y, long *dest) {
    long t = mult2(x, y);
    *dest = t;
}
//ass
0000000000400540 <multstore>:
400540: push   %rbx		# Save %rbx
400541: mov    %rdx,%rbx	# Save dest
400544: callq  400550 <mult2>	# mult2(x,y)
400549: mov    %rax,(%rbx)	# Save at dest
40054c: pop    %rbx		# Restore %rbx
40054d: retq

long mult2
  (long a, long b)
{
  long s = a * b;
  return s;
}

//ass
0000000000400550 <mult2>:
400550:  mov    %rdi,%rax	# a 
400553:  imul   %rsi,%rax	# a * b
400557:  retq			# Return




```
