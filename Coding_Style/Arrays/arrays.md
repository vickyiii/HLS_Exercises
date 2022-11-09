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

 Please refer to [Opt_Interface](../Defining_Interface/Array/)
