1. Managing Pipeline Dependencies
```C++
void histogram(int in[INPUT SIZE], int hist[VALUE SIZE]) f
  int acc = 0;
  int i, val;
  int old = in[0];
  for(i = 0; i < INPUT SIZE; i++)
  {
    #pragma HLS PIPELINE II=1
    val = in[i];
    if(old == val)
    {
        acc = acc + 1;
    }
    else
    {
        hist[old] = acc;
        acc = hist[val] + 1;
    }
     
    old = val;
  }
    
  hist[old] = acc;
```

```C++
void histogram(int in[INPUT SIZE], int hist[VALUE SIZE]) {
  int acc = 0;
  int i, val;
  int old = in[0];
  #pragma HLS DEPENDENCE variable=hist type=intra direction=RAW dependent=false
  for(i = 0; i < INPUT SIZE; i++)
  {
    #pragma HLS PIPELINE II=1
    val = in[i];
    if(old == val)
    {
        acc = acc + 1;
    }
    else
    {
        hist[old] = acc;
        acc = hist[val] + 1;
    }
     
    old = val;
  }
    
  hist[old] = acc;
```