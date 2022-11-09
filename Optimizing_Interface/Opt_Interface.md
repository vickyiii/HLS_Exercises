
## Array
1. Array Interface

The INTERFACE pragma or directive lets you explicitly define which type of RAM or ROM is used with the storage_type=<value> option. This defines which ports are created (single-port or dual-port). If no storage_type is specified, Vitis HLS uses:

A single-port RAM by default.
A dual-port RAM if it reduces the initiation interval or reduces latency.
The ARRAY_PARTITION and ARRAY_RESHAPE pragmas can re-configure arrays on the interface. Arrays can be partitioned into multiple smaller arrays, each implemented with its own interface. This includes the ability to completely partition the array into a set of scalars. On the function interface, this results in a unique port for every element in the array. This provides maximum parallel access, but creates many more ports and might introduce routing issues during implementation.

By default, the array arguments in the function shown in the following code example are synthesized into a single-port RAM interface.

```C++
#include "array_RAM.h"

void array_RAM (dout_t d_o[4], din_t d_i[4], didx_t idx[4]) {
 int i;

 For_Loop: for (i=0;i<4;i++) {
 d_o[i] = d_i[idx[i]];
 }

}
```
A single-port RAM interface is used because the for-loop ensures that only one element can be read and written in each clock cycle. There is no advantage in using a dual-port RAM interface.

If the for-loop is unrolled, Vitis HLS uses a dual-port RAM. Doing so allows multiple elements to be read at the same time and improves the initiation interval. The type of RAM interface can be explicitly set by applying the INTERFACE pragma or directive, and setting the storage_type.

2. FIFO Interface

Vitis HLS allows array arguments to be implemented as FIFO ports in the RTL. If a FIFO ports is to be used, you must ensure that the accesses to and from the array are sequential.

Note: If the accesses at the interface are not sequential, there is an RTL simulation mismatch.
The following code example shows a case in which the tool cannot determine whether the accesses are sequential. In this example, both d_i and d_o are specified to be implemented with a FIFO interface during synthesis. In this case, you must ensure the access is sequential or you will be introducing errors into your system.

```C++
#include "array_FIFO.h"

void array_FIFO (dout_t d_o[4], din_t d_i[4], didx_t idx[4]) {
 int i;
#pragma HLS INTERFACE mode=ap_fifo port=d_i
#pragma HLS INTERFACE mode=ap_fifo port=d_o
 For_Loop: for (i=0;i<4;i++) {
 d_o[i] = d_i[idx[i]];
 }
}
```

- If the values of the elements of idx are sequential, then a FIFO interface could be created
- If random values are used for idx, a FIFO interface fails in Co-simulation when implemented in RTL, and may also fail during runtime

The following general rules apply to arrays that are implemented with a FIFO interface:

- The array must be only read or written in the loop or function. This can be transformed into a point-to-point connection that matches the characteristics of FIFO links.
- The array reads must be in the same order as the array writes. Because random access is not supported for FIFO channels, the array must be used in the program following first in, first out semantics.
- The following conditions apply when the data type of an array is a struct, and the array is sequentially accessed (i.e. the array is specified with the axis or ap_fifo interface or is marked with STREAM pragma or directive:
- You cannot access struct members directly from I/O arguments that use array-to-stream, or are in streaming interfaces. You can make a local copy of the struct in order to read/write member elements.
You must ensure sequential order access, as shown below:

Bad example 1:

```C++
struct A {
  short foo;
  int bar;
};
void dut(A in[N], A out[out], bool flag) {
  #pragma HLS interface ap_fifo port=in,out
  for (unsigned i=0; i<N; i++) {
    out[i] = in[i];
    if (flag)
      out[i].bar += 5;
  }
}
```
Bad example 2:

```C++
void dut(A in[N], A out[out], bool flag) {
  #pragma HLS interface ap_fifo port=in,out
  for (unsigned i=0; i<N; i++) {
    out[i].foo = in[i].foo;
    if (flag)
      out[i].bar = in[i].bar + 5;
    else
      out[i].bar = in[i].bar;
  }
}
```
Convert TO:
```C++
void dut(A in[N], A out[out], bool flag) {
  #pragma HLS interface ap_fifo port=in,out
  for (unsigned i=0; i<N; i++) {
    A tmp = in[i];
    if (flag)
      tmp.bar += 5;
    out[i] = tmp;
  }
}
```

```C++
```C++
```C++
```C++