课件地址:  
http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/08-machine-data.pdf

### Array Allocation
- basic principle 基本原则
T A[L];
	- Array of data type T and length L 数组的类型为T长度为L
	- Contiguously allocated region of L * sizeof(T) bytes in memory 内存中L * sizeof（T）个字节的连续分配区域
	- char *p[3] int型,char型,double型指针所占内存大小相同。在64位机器下,都占8个字节。char *p即指针占8个字节,
	  指针所指向的内存占1(char)字节

### Array Access
- basic principle 基本原则
T A[L];
	- Array of data type T and length L 数组的类型为T长度为L
	- Identifier A can be used as a pointer to array element 0:Type T* 标识符A可用作指向数组元素0的指针：
	  类型T *(即A就是该数组的开始地址)

int val[5];  
![pic_ont](/pic/int_val[5].png)

| Reference  | Type  |   Value   |
| :--------: | :---: | :-------: |
|   val[4]   |  int  |     3     |
|    val     | int * |     x     |
|    val     | int * |   x + 4   |
|  &val[2]   | int * |   x +8    |
|   val[5]   |  int  |  ??error  |
| *(val + 1) |  int  |     5     |
|  val + i   | int * | x + 4 * i |



### Array Accessing Example

```c
int zip_dig[5] = {1,5,2,1,3}
int get_digit(zip_dig z,int digit){
	return z[digit]
}
//ass
                                # %rdi = z , # %rsi = digit
movl(%rdi,%rsi,4),%eax          #z[digit]
```
- Register %rdi contains starting address of array 寄存器%rdi存数组zip_dig的开始地址
- Register %rsi contains array index 数组zip_dig的index
- Desired digit at %rdi + 4 * %rsi
- Use memory reference (%rdi + 4 * %rsi) 数组在内存对应的取值

### Array Loop Example
```c
int zip_dig[5] = {1,5,2,1,3}
void zincr(zip_dig z){
	size_t i;
	for(i = 0; i < ZLEN; i++){
		z[i]++;
	}	
}

//ass
	# %rdi = z
	movl $0,%eax          # i=0
	jmp .L3               # goto middle
.L4
	addl $1,(%rdi,%rax,4) # z[i]++ 数组中值各自增1
	addq $1,%rax          # i++ int i 自增1
.L3
	cmpq $4,%rax          # i:4
	jbe	 .L4          # if <=,goto loop
	rep;ret
```

### Multidimensional(Nested) Arrays 多维数组
- Declaration
	T A [R] [C];
		- 2D array of data type T
		- R rows, C columns
		- Type T element requires K bytes 类型T元素需要K个字节
- Array Size
	- R * C * K bytes

- Arrangement
	- Row-Major Ordering 按行排序

- Nested Array Example
	- zip_dig pgh[4] "equivalent to" int pgh[4][5]
		- variable pgh : array of 4 elements,allocatee contiguously
		- Each element is an array of 5 int's,allocatee contiguously
	- "Row-Major" ordering of all elements in memory


#### Nested Array Row Access Code 访问多维数组行
- Row Vector
	- pgh[index] is array of 5 int's
	- starting address pgh+20*index

- Machine Code
	- Computes and return address
	- Compute as pgh + 4 *(index + 4 * index)

```c
# define PCOUNT 4
zip_dig pgh[PCOUNT] = {
	{1,5,2,0,6},
	{1,5,2,0,3},
	{1,5,2,0,7},
	{1,5,2,0,1},
};

int *get_pgh_zip(int index){
	return pgh[index];
}
//ass
# %rdi = index
leaq (%rdi,%rdi,4),%rax       # 5*index
leaq pgh(,%rax,4), %rax       # pgh + (20 * index)
```



#### Nested Array Element Access 多维数组具体元素访问

- Array Elements
  - A[i] [j] is element of type T,which requires k bytes
  - Address A + i * (C * K) + j * K = A + (i + C + j) * K

##### Nested Array Element Access Code
- Array Elements
  - pgh[index] [dig] is int
  - address: pgh + 20 * index + 4*dig
  - = pgh + 4 *（5 * index + dig）

```c
# define PCOUNT 4
zip_dig pgh[PCOUNT] = {
	{1,5,2,0,6},
	{1,5,2,0,3},
	{1,5,2,0,7},
	{1,5,2,0,1},
};

int get_pgh_digit(int index,int dig){
    return pgh[index][dig];
}
//ass
leaq (%rdi,%rdi,4),%rax           # 5 * index 一维数组
addl %rax,%rsi                    # 5 * index + dig 二维数组具体dig	
movel pgh(,%rsi,4),%eax           # M[pgh + 4 * (5 * index + dig)] 4为int 字节位置偏移
```



### Multi-Level Array Example

- Variable univ denotes array of 3 elements

- Each element is a pointer

  - 8 bytes

- Each pointer points to array of ints

### Element Access in Multi-Level Array
- Computsation
	- Element access Mem[Mem [univ + 8 * index] + 4 * digit]
	- must do two memory reads
		- Fist get pointer to row array
		- Then access element within array

![pic_two](/pic/univ.png)
```c
zip_deg cmu = {1，2，2，1，3};
zip_deg mit = {0，2，2，1，3};
zip_deg ucb = {9，2，2，1，3};
#define UCOUNT 3
int *univ[UCOUNT] = {mit,cmu,ucb};

int get_univ_digit(size_t index,size_t digit){
	return univ[index][digit];
}

//ass
salq $2,%rsi                # 4 * digit 单个数组某个digit的长度
addq univ(,%rdi,8),%rsi     # p = univ[index] + 4 * digit univ(,%rdi,8)取得指针指向的位置
movl (%rsi),%eax            # return *p
ret
```

### Array Element Accesses 数组元素存取对比
- Accesses look similar in C,but address computations very different 访问在C中看起来相似,实际地址计算方式不同

```c
//nested array 嵌套数组
int get_pgh_digit(size_t index,size_t digit){
	return pgh[index][digit];
}
Mem[pgh + 20 * index + 4 * digit]

//multi-level array 多级数组
int get_univ_digit(size_t index,size_t digit){
	return univ[index][digit];
}
Mem[ Mem [univ + 8 * index] + 4 * digit]
```

### N X N Matrix Code
- Fixed dimensions 固定尺寸
	- know value of N at compile time 知道n大小
- Array Elements
	- Address A + i * ( C * K) + j * K
	- C = 16 ,K = 4
```c
# define N 16
typedef int fix_matrix[N][N]
int fix_ele(fix_matrix a,size_t i,size_t j){
	return[i][j]
}

//ass
# a in %rdi, i in %rsi, j in %rdx
salq $6,%rsi              # 64 * i
addq %rsi,%rdi            # a + 64 * i
movl (%rdi,%rdx,4),%eax   # M[a + 64 * i + 4 * j]
ret
```

- Variable dimensions,explicit indexing 显式的指定index算法
```c
# define IDX(n,i,j) ((i) * (n) + (j))
int vec_ele(size_t n, int *a,size_t i, size_t j){
	return a[ IDX(n,i,j) ]
}
```

- Variable dimensions,implicit indexing n通过参数传递,数组可变大小
	- now supported by gcc

```c
int var_ele(size_t n, int a[n][n],size_t i,size_t j) {
	return a[i][j];
}

//ass
# n in %rdi, a in % rsi, i in %rdx, j in % rcx

imulq %rdx,%rdi 		# n * i
leaq (%rsi,%rdi,4),%rax		# a + 4 * n * i
movl (%rax,%rcx,4),%eax		# a + 4 * n * i + 4 * j
ret
```


## Structures

### Structure Representation
- structure represented as block of memory 结构的实现是通过内存一块连续的区域实现
  - big enough to hold all of the fields 足够大已支持结构中的所有字段
  
- Fields ordered according to declaration 根据在结构中声明的字段排序
  - Even if another ordering could yield a more compact representation 更合理的声明方式能实现更紧凑的效果,节省内存

- Compiler determines overall size + positions of fields 编译器确定整体大小+字段位置
  - machine-level program has no understanding or the structures in the source code 机器级程序不了解源代码中的结构
```c
struct rec{
  int a[4];
  size_t i;
  struct rec *next
}
```
![pic_two](/pic/struct_1.jpg)

### Generating Pointer to Structure Member
- Generating Pointer to Array Element
  - offset of each structure member determined at compile time 在编译时确定的每个结构成员的偏移量
  - compute as r + 4 * idx
```c
struct rec{
  int a[4];
  size_t i;
  struct rec *next
}

int *get_ap(struct rec *r,size_t idx){
  return &r->a[idx]; //返回a[idx]内存的地址
}
//ass
# r in %rdi , idx in %rsi
leaq (%rdi,%rsi,4), %rax
ret
```

### Following Linked List
```c
//该函数实现了三种不同的内存访问形式
struct rec{
  int a[4];
  size_t i;
  struct rec *next
}

void set_val(struct rec *r,int val){
  while(r){
    int i = r->i;
    r->a[i] = val;
    r = r->next;
  }
}
//ass
# r in %rdi, val in %rsi
.L11				#loop
  movslq 16(%rdi), %rax		#  i = M[r+16]
  movl   %esi,(%rdi,%rax,4)     #  M[r + 4 * i] = val
  movq 24(%rdi),%rdi		#  r = M[r + 24]
  testq %rdi,%rdi		#  Test r
  jne   .L11			# if != 0 goto loop

```

### Structures & Alignment
```c
struct S1{
  char c;
  int i[2];
  double v;
} *p;
```
- Unaligned Data 未对齐的结构  


![pic_two](/pic/struct_2.jpg)
- Aligned Data 
  - Primitive data type requires K bytes
  - Address must be multiple of K 地址必须是K的倍数  


![pic_two](/pic/struct_3.jpg)


### Alignment Principles
- Aligned Data
  - Primitive data type requires K bytes
  - Address must be multiple of K 地址必须是K的倍数
  - required on some machines;advised on x86-64 在一些机器上面是必须的

- Motivation(动机) for Aligning Data 
  - memory accessed by (aligned) chunks of 4 or 8 bytes(system dependent) 由4或8个字节的（对齐的）块访问的内存（取决于系统）
    - inefficient to load or store datum that spans quad word 加载或存储跨越四字的数据效率低下
    - virtual memory trickier when datum spans 2 pages 当数据跨越2页时，虚拟内存会更棘手


- Compiler
  - inserts gaps in structure to ensure correct alignment of fields 在结构中插入间隙以确保正确对齐字段


### Satisfying alignment with structures
- within structure
  - must satisfy each element's alignment requirement 必须满足每个元素的对齐要求

- Overall structure has alignment requirement K 整体结构有对准要求K
  - Each structure has alignment requiremnet K 每个结构都有对齐要求
    - K = Largest alignment of any element 
  - initial address & structure length must be multiples of K 初始地址和结构长度必须为K的倍数

- Example:

```c
struct S1{
  char c;
  int i[2];
  double v;
} *p;
```
  - K = 8,due to double element  


![pic_two](/pic/struct_4.jpg)


### Meeting overall alignment requirement
- for largest alignment requirement K
- overall structure must be multiple of K

![pic_two](/pic/struct_5.jpg)

### Arrays of Structures
- overall structure must be multiple of K
- satisty alignment requirement for every element 每个元素要满足对齐需求
```c
struct S2{
  double v;
  int i[2];
  char c;
} a[10]
```
![pic_two](/pic/struct_6.jpg)


### Accessing Array Elements 访问包含n个struct数组某个struct中的元素
- Compute array offset 12 * idx
  - sizeof(S3),including alignment spacers

- Element j is at offset 8 within structure
- Assembler gives offset a + 8
  - resolved duiing linking

```c
struct S3{
  short i;
  float v;
  short j;
} a[10]

short get_j(int idx){
  return a[idx].j
}

//ass
# %rdi = idx
leaq (%rdi,%rdi,2),%rax 	# 3* idx
movzwl a + 8(,%rax,4),%eax	# a + 12 * idx + 8
```
![pic_two](/pic/struct_7.jpg)

### Saving Space
- put large data types first 把结构中的大的字节放在前面
- Effect(K =4) 结构S4/S5中int为最大字节

```c
Struct S4{
  char c;
  int i;
  char d;
} *p;
```
![pic_two](/pic/struct_8.jpg)

```c
Struct S5{
  int i;
  char c;
  char d;
} *p;
```
![pic_two](/pic/struct_9.jpg)

