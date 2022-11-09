1. standard C/C++ types
 Vitis HLS supports the synthesis of all standard C/C++ types, including exact-width integer types.

- (unsigned) char, (unsigned) short, (unsigned) int
- (unsigned) long, (unsigned) long long
- (unsigned) intN_t (where N is 8, 16, 32, and 64, as defined in stdint.h)
- float, double
- Use data type (unsigned)int or (unsigned)int32_t instead of type (unsigned)long for 32-bit.
- Use data type (unsigned)long long or (unsigned)int64_t instead of type (unsigned)long for 64-bit.

Tip: The std::complex<long double> data type is not supported in Vitis HLS and should not be used.

1. Arbitrary Precision (AP) Data Types

For the C++ language ap_[u]int data types the header file ap_int.h defines the arbitrary precision integer data type. To use arbitrary precision integer data types in a C++ function:

- Add header file ap_int.h to the source code.
- Change the bit types to ap_int<N> or ap_uint<N>, where N is a bit-size from 1 to 1024.
- The default maximum width allowed for ap_[u]int data types is 1024 bits. This default may be overridden by defining the macro AP_INT_MAX_W with a positive integer value less than or equal to 4096 before inclusion of the ap_int.h header file.

The following example shows how the header file is added and two variables implemented to use 9-bit integer and 10-bit unsigned integer types:
```C++
#define AP_INT_MAX_W 4096 // Must be defined before next line
#include "ap_int.h"

void foo_top () {

ap_int<9>  var1;           // 9-bit
ap_uint<10>  var2;           // 10-bit unsigned
```

```C++

```
```C++

```
```C++

```
```C++

```
```C++

```
```C++

```