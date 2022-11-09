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

Contert TO: 
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