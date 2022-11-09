This chapter explains how various constructs of C and C++11/C++14 are synthesized into an FPGA hardware implementation and and discusses restrictions with regard to standard C coding.

## Unsupported C/C++ Constructs Transfer
While Vitis HLS supports a wide range of the C/C++ languages, some constructs are not synthesizable, or can result in errors further down the design flow. This section discusses areas in which coding changes must be made for the function to be synthesized and implemented in a device.

To be synthesized:

- The function and its calls must contain the entire functionality of the design.
- None of the functionality can be performed by system calls to the operating system.
- The C/C++ constructs must be of a fixed or bounded size.
- The implementation of those constructs must be unambiguous.

1. Dynamic Memory Usage

Any system calls that manage memory allocation within the system, for example, malloc(), alloc(), and free(), are using resources that exist in the memory of the operating system and are created and released during runtime. To be able to synthesize a hardware implementation the design must be fully self-contained, specifying all required resources.

Memory allocation system calls must be removed from the design code before synthesis. Because dynamic memory operations are used to define the functionality of the design, they must be transformed into equivalent bounded representations. The following code example shows how a design using malloc() can be transformed into a synthesizable version and highlights two useful coding style techniques:
```C++
 long long *out_accum = malloc (sizeof(long long));
 int* array_local = malloc (64 * sizeof(int));
```
Converting Into:

```C++
 long long _out_accum;
 long long *out_accum = &_out_accum;
 int _array_local[64];
 int* array_local = &_array_local[0];
 ```
exmpale: [malloc_removed](./malloc_removed/)
2. Recursive Function

Recursive functions cannot be synthesized because their function call depth is data-dependent, thus non-determined at compiler time.
This applies to functions that can form multiple recursions:
```C++
unsigned foo (unsigned n) 
{  
    if (n == 0 || n == 1) return 1;  
    return (foo(n-2) + foo(n-1)); 
} 
```
Vitis HLS also does not support tail recursion, in which there is a finite number of function calls.
```C++
unsigned foo (unsigned m, unsigned n)  
{  
    if (m == 0) return n;  
    if (n == 0) return m; 
    return foo(n, m%n); 
} 
```
In C++, templates can implement tail recursion and can then be used for synthesizable tail-recursive designs. Because tail-recursion is a loop in disguise, the simple function easily be transformed as below.

```C++
Unsigned foo (unsigned m, unsigned n)  {
    while(m!=0 & n!=0){
        unsigned int mmodn=m%n;
        m=n;
        n=mmodn;
    }
    if(m==0) return n;
    else return m;
}
```
## Loop

1. Variable Loop Bounds
When a loop or function is pipelined, Vitis HLS unrolls all loops in the hierarchy below the function or loop. If there is a loop with variable bounds in this hierarchy, it prevents pipelining.

The solution to loops with variable bounds is to make the number of loop iteration a fixed value with conditional executions inside the loop. The code from the variable loop bounds example can be rewritten as shown in the following code example. Here, the loop bounds are explicitly set to the maximum value of variable width and the loop body is conditionally executed:

```C++
#include "ap_int.h"
#define N 32

typedef ap_int<8> din_t;
typedef ap_int<13> dout_t;
typedef ap_uint<5> dsel_t;

dout_t code028(din_t A[N], dsel_t width) {  

 dout_t out_accum=0;
 dsel_t x;

 LOOP_X:for (x=0;x<width; x++) {
 out_accum += A[x];
 }

 return out_accum;
}
```

```C++
#include "ap_int.h"
#define N 32

typedef ap_int<8> din_t;
typedef ap_int<13> dout_t;
typedef ap_uint<5> dsel_t;

dout_t loop_max_bounds(din_t A[N], dsel_t width) {  

 dout_t out_accum=0;
 dsel_t x;

 LOOP_X:for (x=0; x<N; x++) {
 if (x<width) {
  out_accum += A[x];
 }
 }

 return out_accum;
}
```

2. Loop Parallelism

Vitis HLS schedules logic and functions early as possible to reduce latency while keeping the estimated clock period below the user-specified period. To perform this, it schedules as many logic operations and functions as possible in parallel. It does not schedule loops to execute in parallel.

If the following code example is synthesized, loop SUM_X is scheduled and then loop SUM_Y is scheduled: even though loop SUM_Y does not need to wait for loop SUM_X to complete before it can begin its operation, it is scheduled after SUM_X.
```C++
#include "loop_sequential.h"

void loop_sequential(din_t A[N], din_t B[N], dout_t X[N], dout_t Y[N], 
 dsel_t xlimit, dsel_t ylimit) {  

 dout_t X_accum=0;
 dout_t Y_accum=0;
 int i,j;

 SUM_X:for (i=0;i<xlimit; i++) {
 X_accum += A[i];
 X[i] = X_accum;
}

 SUM_Y:for (i=0;i<ylimit; i++) {
 Y_accum += B[i];
 Y[i] = Y_accum;
 }
} 
```
Because the loops have different bounds (xlimit and ylimit), they cannot be merged. By placing the loops in separate functions, as shown in the following code example, the identical functionality can be achieved and both loops (inside the functions) can be scheduled in parallel.

```C++
#include "loop_functions.h"

void sub_func(din_t I[N], dout_t O[N], dsel_t limit) {
 int i;
 dout_t accum=0;
  
 SUM:for (i=0;i<limit; i++) {
 accum += I[i];
 O[i] = accum;
 }

}

void loop_functions(din_t A[N], din_t B[N], dout_t X[N], dout_t Y[N], 
 dsel_t xlimit, dsel_t ylimit) {

 sub_func(A,X,xlimit);
 sub_func(B,Y,ylimit);
}
```
3. Unrolling Loops in C++ Classes

When loops are used in C++ classes, care should be taken to ensure that the loop induction variable is not a data member of the class as this prevents the loop from being unrolled.
In this example, loop induction variable k is a member of class foo_class

```C++
template <typename T0, typename T1, typename T2, typename T3, int N>
class foo_class {
private:
 pe_mac<T0, T1, T2> mac;
public:
 T0 areg;
 T0 breg;
 T2 mreg;
 T1 preg;
    T0 shift[N];
 int k;             // Class Member
   T0 shift_output;
 void exec(T1 *pcout, T0 *dataOut, T1 pcin, T3 coeff, T0 data, int col)
 {
Function_label0:;
#pragma HLS inline off
 SRL:for (k = N-1; k >= 0; --k) {
#pragma HLS unroll // Loop will fail UNROLL
 if (k > 0) 
 shift[k] = shift[k-1];
 else 
 shift[k] = data;
 }

    *dataOut = shift_output;
   shift_output = shift[N-1];
 }

 *pcout = mac.exec1(shift[4*col], coeff, pcin);
};
```
For Vitis HLS to be able to unroll the loop as specified by the UNROLL pragma directive, the code should be rewritten to remove k as a class member.

```C++
template <typename T0, typename T1, typename T2, typename T3, int N>
class foo_class {
private:
 pe_mac<T0, T1, T2> mac;
public:
 T0 areg;
 T0 breg;
 T2 mreg;
 T1 preg;
    T0 shift[N];
   T0 shift_output;
 void exec(T1 *pcout, T0 *dataOut, T1 pcin, T3 coeff, T0 data, int col)
 {
Function_label0:;
 int k;             // Local variable
#pragma HLS inline off
 SRL:for (k = N-1; k >= 0; --k) {
#pragma HLS unroll // Loop will unroll
 if (k > 0) 
 shift[k] = shift[k-1];
 else 
 shift[k] = data;
 }

    *dataOut = shift_output;
   shift_output = shift[N-1];
 }

 *pcout = mac.exec1(shift[4*col], coeff, pcin);
};
```
## Arrays

1. Large Arrays

Before discussing how the coding style can impact the implementation of arrays after synthesis, it is worthwhile discussing a situation where arrays can introduce issues even before synthesis is performed, for example, during C/C++ simulation.

If you specify a very large array, it might cause C/C++ simulation to run out of memory and fail, as shown in the following example:
```C++
#include "ap_int.h"
  
  int i, acc; 
  // Use an arbitrary precision type
  ap_int<32>  la0[10000000], la1[10000000]; 
  
  for (i=0 ; i < 10000000; i++) { 
      acc = acc + la0[i] + la1[i]; 
  } 
```

The simulation might fail by running out of memory, because the array is placed on the stack that exists in memory rather than the heap that is managed by the OS and can use local disk space to grow.

This might mean the design runs out of memory when running and certain issues might make this issue more likely:

On PCs, the available memory is often less than large Linux boxes and there might be less memory available.
Using arbitrary precision types, as shown above, could make this issue worse as they require more memory than standard C/C++ types.
Using the more complex fixed-point arbitrary precision types found in C++ might make the issue of designs running out of memory even more likely as types require even more memory.
The standard way to improve memory resources in C/C++ code development is to increase the size of the stack using the linker options such as the following option which explicitly sets the stack size -z stack-size=10485760. This can be applied in Vitis HLS by going to **Project Settings > Simulation > Linker flags**, or it can also be provided as options to the Tcl commands:

```
csim_design -ldflags {-z stack-size=10485760} 
cosim_design -ldflags {-z stack-size=10485760} 
```
A solution is to use dynamic memory allocation for simulation but a fixed sized array for synthesis, as shown in the next example. This means that the memory required for this is allocated on the heap, managed by the OS, and which can use local disk space to grow.

A change such as this to the code is not ideal, because the code simulated and the code synthesized are now different, but this might sometimes be the only way to move the design process forward. If this is done, be sure that the C/C++ test bench covers all aspects of accessing the array. The RTL simulation performed by cosim_design will verify that the memory accesses are correct.
```C++
#include "ap_int.h"
  
  int i, acc; 
#ifdef __SYNTHESIS__
  // Use an arbitrary precision type & array for synthesis
  ap_int<32>  la0[10000000], la1[10000000]; 
#else 
  // Use an arbitrary precision type & dynamic memory for simulation
 ap_int<int32> *la0 = malloc(10000000  * sizeof(ap_int<32>));
 ap_int<int32> *la1 = malloc(10000000  * sizeof(ap_int<32>));
#endif
  for (i=0 ; i < 10000000; i++) { 
      acc = acc + la0[i] + la1[i]; 
  } 
```

2. Array Accesses and Performance

The following code example shows a case in which accesses to an array can limit performance in the final RTL design. In this example, there are three accesses to the array mem[N] to create a summed result.

```C++
#include "array_mem_bottleneck.h"
 
dout_t array_mem_bottleneck(din_t mem[N]) {  

 dout_t sum=0;
 int i;

 SUM_LOOP:for(i=2;i<N;++i)
   sum += mem[i] + mem[i-1] + mem[i-2];
    
 return sum;
}
```
The issue here is that the single-port RAM has only a single data port: only one read (or one write) can be performed in each clock cycle.

SUM_LOOP Cycle1: read mem[i];
SUM_LOOP Cycle2: read mem[i-1], sum values;
SUM_LOOP Cycle3: read mem[i-2], sum values;
A dual-port RAM could be used, but this allows only two accesses per clock cycle. Three reads are required to calculate the value of sum, and so three accesses per clock cycle are required to pipeline the loop with an new iteration every clock cycle.

```C++
#include "array_mem_perform.h"
 
dout_t array_mem_perform(din_t mem[N]) {  

 din_t tmp0, tmp1, tmp2;
 dout_t sum=0;
 int i;

 tmp0 = mem[0];
 tmp1 = mem[1];
 SUM_LOOP:for (i = 2; i < N; i++) { 
 tmp2 = mem[i];
 sum += tmp2 + tmp1 + tmp0;
 tmp0 = tmp1;
 tmp1 = tmp2;
 } 
    
 return sum;
}
```
3. Array Initialization
Although not a requirement, Xilinx recommends specifying arrays that are to be implemented as memories with the static qualifier. This not only ensures that Vitis HLS implements the array with a memory in the RTL; it also allows the initialization behavior of static types to be used.

In the following code, an array is initialized with a set of values. Each time the function is executed, array coeff is assigned these values. After synthesis, each time the design executes the RAM that implements coeff is loaded with these values. For a single-port RAM this would take eight clock cycles. For an array of 1024, it would of course take 1024 clock cycles, during which time no operations depending on coeff could occur.
```
int coeff[8] = {-2, 8, -4, 10, 14, 10, -4, 8, -2};
```
The following code uses the static qualifier to define array coeff. The array is initialized with the specified values at start of execution. Each time the function is executed, array coeff remembers its values from the previous execution. A static array behaves in C/C++ code as a memory does in RTL.

Convert TO:
```
static int coeff[8] = {-2, 8, -4, 10, 14, 10, -4, 8, -2};
```
4. Implementing ROMs

Vitis HLS does not require that an array be specified with the static qualifier to synthesize a memory or the const qualifier to infer that the memory should be a ROM. Vitis HLS analyzes the design and attempts to create the most optimal hardware.

Important: Xilinx highly recommends using the static qualifier for arrays that are intended to be memories. As noted in Array Initialization, a static type behaves in an almost identical manner as a memory in RTL.

The const qualifier is also recommended when arrays are only read, because Vitis HLS cannot always infer that a ROM should be used by analysis of the design. The general rule for the automatic inference of a ROM is that a local (non-global), static array is fully written to before being read, and never written again. The following practices in the code can help infer a ROM:

- Initialize the array as early as possible in the function that uses it.
- Group writes together.
- Do not interleave array(ROM) initialization writes with non-initialization code.
- Do not store different values to the same array element (group all writes together in the code).

Element value computation must not depend on any non-constant (at compile-time) design variables, other than the initialization loop counter variable.
If complex assignments are used to initialize a ROM (for example, functions from the math.h library), placing the array initialization into a separate function allows a ROM to be inferred. In the following example, array sin_table[256] is inferred as a memory and implemented as a ROM after RTL synthesis.

```C++
#include "array_ROM_math_init.h"
#include <math.h>

void init_sin_table(din1_t sin_table[256])
{
 int i;
 for (i = 0; i < 256; i++) {
 dint_t real_val = sin(M_PI * (dint_t)(i - 128) / 256.0);
 sin_table[i] = (din1_t)(32768.0 * real_val);
 }
}

dout_t array_ROM_math_init(din1_t inval, din2_t idx)
{
 short sin_table[256];
 init_sin_table(sin_table);
 return (int)inval * (int)sin_table[idx];
}
```
5. Array Interface 

 Please refer to [Opt_Interface](../Optimizing_Interface/Array/)

 ## Data Types

1. standard C/C++ types
 Vitis HLS supports the synthesis of all standard C/C++ types, including exact-width integer types.

- (unsigned) char, (unsigned) short, (unsigned) int
- (unsigned) long, (unsigned) long long
- (unsigned) intN_t (where N is 8, 16, 32, and 64, as defined in stdint.h)
- float, double

The C/C++ standard dictates that type (unsigned)long is implemented as 64 bits on 64-bit operating systems and as 32 bits on 32-bit operating systems. Synthesis matches this behavior and produces different sized operators, and therefore different RTL designs, depending on the type of operating system on which Vitis HLS is run. On Windows OS, Microsoft defines type long as 32-bit, regardless of the OS.

- Use data type (unsigned)int or (unsigned)int32_t instead of type (unsigned)long for 32-bit.
- Use data type (unsigned)long long or (unsigned)int64_t instead of type (unsigned)long for 64-bit.

Xilinx highly recommends defining the data types for all variables in a common header file, which can be included in all source files.

During the course of a typical Vitis HLS project, some of the data types might be refined, for example to reduce their size and allow a more efficient hardware implementation.
One of the benefits of working at a higher level of abstraction is the ability to quickly create new design implementations. The same files typically are used in later projects but might use different (smaller or larger or more accurate) data types.

Tip: The std::complex<long double> data type is not supported in Vitis HLS and should not be used.

1. Arbitrary Precision (AP) Data Types


```C++
```C++
```C++
```C++
```C++
```C++
```C++
```C++
```C++
```C++
```C++
```C++
```C++
```C++
```C++
```C++