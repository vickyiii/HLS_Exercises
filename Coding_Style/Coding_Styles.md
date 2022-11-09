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