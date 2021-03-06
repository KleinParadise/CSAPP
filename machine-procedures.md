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

### Control Flow Example
1.准备使用Call指令调用mult2函数,当前栈顶0x120  
![pic_one](/pic/example1.png)  
2.栈顶自减8位,更新当前栈顶为0x118,在该栈顶0x118位置保存call指令的后一指令地址0x40059,更新当前指令寄存器(%rip)为call需要跳转的地址  
![pic_two](/pic/example2.png)  
3.mult2函数执行完毕,从栈顶寄存器%rsp保存的栈顶位置0x180弹出下一条指令执行的地址0x400549  
![pic_three](/pic/example3.png)  
4.更新栈顶的位置自增8位为0x120,更新当前指令寄存器(%rip)为弹出地址0x400549继续执行改程序    
![pic_four](/pic/example4.png)


### Procedure Data Flow
  - Fist 6 arguments in %rdi,%rsi,%rdx,%rcx,%r8,%r9
  - 多于6个参数存入栈中
  - Return value in %rax


### Stack-Based Languages
  - Languages that support recursion 支持递归的语言
    - e.g. C,Java
    - Code must be "Reentrant" 代码必须可重入
      - Multiple simultaneous instantiations of single procedure 单线程多个实例
    - Need some place to store state of each instantiation 需要有地方存以下栈帧信息
      - arguments
      - local variables
      - return pointer
    - Stack discipline
      - state for given procedure needed for limited time 在有限的时间内需要给定过程的状态
        - From when called to when return 从何时调用到何时返回
      - Callee returns before caller does Callee先于Caller返回
    - Stack allocated in Frames 栈帧存于栈中
      - state for single procedure instantiation 单线程实例化状态信息   



### Stack Frames
- Contents
  - return information 返回信息
  - local storage (if needed) 本地存储
  - Temporary space(if needed）临时空间

- Management 栈帧管理
  - Space allocated when enter procedure 进入是分配空间
    - "set-up" code 
    - includes push by call instruction

  - Deallocated when return return是释放空间
    - "finis" code 
    - includes pop by ret instruction

### x86-64/Linux Stack Frame
- Current Stack Frame ("Top" to Bottom) 当前栈帧需要保存的
  - "Argument build"
    - parameters for function about to call 要调用函数的参数
  - Local variables
    - if canot keep in registers 不能保存在寄存器的本地变量
  - Saved register content 寄存器内容
  - old frame pointer (optional) 前面栈帧的指针  


Caller Stack Frame
- Return address
  - pushed by call instruction
- Arguments for this call 


### Example Calling
![pic_one](/pic/incr_0.png)
```c
movq (%rdi),%rax //将指针p指向的内存值放入%rax
addq %rax, %rsi  //计算 x + val
movq %rsi,(%rdi) //更改指针p的内存值为x+val
```

![pic_two](/pic/incr_1.png)  
```c
subq $16, %rsp //根据栈顶寄存器%rsp,在栈中开辟16字节内存
movq $15213 , 8(%rsp) 将立即数15213放于偏于栈顶8字节的位置
```
![pic_three](/pic/incr_2.png)  
```c
movl $3000, %esi //将立即数3000放入寄存器%esi(参数寄存器1)
leaq 8(%rsp),%rdi //将指向立即数15213的地址放入寄存器%rdi (参数寄存器2)
```

![pic_four](/pic/incr_3.png)
```c
call incr //调用incr函数,开辟incr栈帧
```

![pic_four](/pic/incr_4.png)
```c
addq 8(%rsp),%rax //将栈中8(%rsp)存的值15213与函数incr的返回值相,存入%rax
addq $16,%rsp 从栈中回收16字节的内存
```

![pic_five](/pic/incr_5.png)
```c
ret //弹出返回地址,跳转
```

### Register Saving conventions 寄存器保存规则
- When procedure yoo calls who 当函数yoo调用函数who
  - yoo is the caller
  - who is the callee

- Conventions
  - "caller Saved"
    - caller saves temporary values in its frame before the call caller在调用call之前将临时值保存在自己栈帧中
  - "callee Saved"
    - callee saves temporary values in its frame before using callee在执行前把临时变量保存在自己当前栈帧中
    - callee restores them before returning to caller 在返回到caller之前callee负责将其恢复


### x86-64/Linux Register Usage
- %rax
  - return value
  - also caller-saved
  - can be modified by procedure 可以通过程序修改  

- %rdi,...,%r9
  - arguments
  - also caller -saved
  - can be modified by procedure 可以通过程序修改

- %r10,%r11
  - caller-saved
  - can be modified by procedure 可以通过程序修改

- %rbx,%r12,%r13,%r14
  - callee-saved
  - callee must save & restore

- %rbp
  - callee-saved
  - callee must save & restore
  - may be used as frame pointer
  - can mix & mach

- %rsp
  - special from of callee save
  - restored to original value upon exit from procedure 从程序退出时恢复到原始值


### Callee-Saved Example
```c
long call_incr2(long x){
  long v1 = 15213;
  long v2 = incr(&v1,3000);
  return x + v2;
  
// ass
call_incr2:
  pushq %rbx              #将%rbx压栈
  subq $16,%rsp           #栈中开辟16字节内存
  movq %rdi,%rbx          #将call_incr2函数的参数x存入寄存器%rbx中
  movq $15213,8(%rsp)     #将立即数15213压入栈顶偏移8字节处
  movl $3000,%esi         #将立即数3000放入参数寄存器%rsi中 
  leaq 8(%rsp),%rdi       #将栈顶偏移8字节处的值放入参数寄存器%rdi中 (此时%rdi里面存入的值要被覆盖故要用都寄存器%rbx存call_incr2的参数x)
  call incr               #调用incr函数(此时%rdi,%rsi已放入incr函数需要的参数)
  addq %rbx,%rax          #incr函数返回值放入%rax中,将之前放入%rbx的call_incr2的参数x拿出,两者相加
  addq $16,%rsp           #释放栈内存
  popq %rbx               #%rbx出栈
  ret
}
```

### Recursive Function 递归函数
```c
long pcount_r(unsigned long x){
  if(x == 0)
    return 0;
  else
    return(x&1) 
          + pcount_r(x>>1);
}

//ass
pcount_t:
  movl $0,%eax
  testq %rdi,%rdi   # if(x == 0) 判断x是否为0,递归终止的条件
  je  .L6           # 结束递归
  pushq %rbx        # %rbx压栈(上一次递归的结果) callee-save
  movq  %rdi,%rbx   # 参数放入%rbx中
  andl  $1,%ebx     # x&1
  shrq  %rdi        # x>> 1
  call  pcount_r    # call pcount_r
  addq  %rbx,%rax   # 上次结果与本地结果相加
  popq  %rbx        # 弹出%rbx
  
.L6
  rep;ret
```

### Observations About Recursion
- handled without special consideration 与普通函数无异,不用特殊处理
  - stack frame mean that each function call has private storage 栈帧中都有自己的一块private的空间
    - saved register & local variables
    - saved return pointer
  - Register saving conventions prevent one function call from corrupting another's data  寄存器保存约定可防止一个函数调用破坏另一个数据
    - unless the C code explicitly does so 除非C代码明确这样做
  - Stack discipline follows call/return pattern 堆栈规则遵循调用/返回模式
    - if P calls Q,the Q returns before P
    - Last in Frist out 先进后出 
  - also works for mutual recursion 也适用相互递归
    - P calls Q; Q calls P 

  - important Points
    - stack is the right data structure for procedure call / return 堆栈是过程调用/返回的正确数据结构
    - if P calls Q, the Q returns before P
  
  - Recursion(& mutual recursion) handled by normal calling conventions 递归(相互调用递归)与函数的正常调用无异
    - Can safely store values in local stack frame and in callee-saved registers 可以安全地将值存储在本栈帧和被调用者保存的寄存器中
    - Put function arguments at top of stack 函数的参数放在栈顶
    - Result return in %rax 结果存储在rax中
  - Pointers are addresses of values 指针是值的地址
    - on stack or global 
  
