# HLS Exercises



## Optimizing for Throughput

### Array_partition

1. factor

2. dimension

3. combined with unroll



### Dataflow

1. pipeline and dataflow

2. bypass

3. change BRAM to FIFO 

4. balance the latency of functions

5. single-producer-consumer violations



### Pipeline

1. pipeline and unroll

2. change dependency



### Loop_tripcount



### Unroll

1. factor

2. combined with array_partition



### Dependence

1. inter dependence



## Optimizing for Latency

### Loop_flatten

1. nested loop

2. reconstruct imperfect loop



### Loop_merge

1. reduce transition

2. in parallel



### Bind_op

1. mul and dsp



## Optimizing for Area

### Inline



### Array_reshape



### Function_instantiate



### Bind_storage

1. RAM_1P and RAM_2P

2. BRAM and LUT



## Optimizing Logic

### Inferring Shift Registers



### Optimizing Logic Expressions



## Optimizing AXI System Performance

### Aggregate_Disaggregate

1. aggregation_of_m_axi_ports

2. aggregation_of_nested_structs

3. aggregation_of_struct

4. auto_disaggregation_of_struct

5. disaggregation_of_axis_port

6. struct_ii_issue



### Memory(m_axi)

1. ecc_flags

2. manual_burst

3. max_widen_port_width

4. memory_bottleneck

5. ram_uram

6. rom_lookup_table_math

7. using_axi_master



### Register(s_axilite)

1. using_axi_lite

2. using_axi_lite_with_user_defined_offset



### Stream(axis)

1. axi_stream_to_master

2. using_array_of_streams

3. using_axi_stream_no_side_channel_data

4. using_axi_stream_with_side_channel_data

5. using_axi_stream_with_struct

6. using_axis_array_stream_no_side_channel_data



## Applications

### DSP

1. Cordic
  
2. FFT


  
### CV

1. Sobel
  
2. Canny



### Neural Network

1. Systolic array
  
2. Convolutional neural network(CNN)


