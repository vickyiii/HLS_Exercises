# HLS Practice

HLS Practice is a collection of small circuit design exercises for practicing digital hardware design using C++ Language. Earlier problems follow a tutorial style, while later problems will increasingly challenge your HLS kernel design skills.

Each problem requires you to design a small circuit in C++. HLS Practice gives you immediate feedback on the circuit module you submit. Your circuit is checked for correctness by simulating with a set of test vectors and comparing it to our reference solution.

## How to use HLS Practice？

Choose a problem: Browse the problem set or go to the first problem
1. Write a solution in C++
2. Submit, simulate, and debug if necessary

## Which exercises should I do?
The exercises are organized by topic and by approximately difficulty within each topic. Start first with the "Getting Started" section to get familiar with how to use HLS Practice. Then start with the easier problems of each topic, and not in a strict top-to-bottom order. 

1. Getting Started
2. Coding Styles
3. Defining Interface
4. Optimization Techniques

## Getting Started

├─Getting_Started.md
├─README.md
├─Coding_Style
|      ├─Coding_Styles.md
|      ├─unsupported_transfer
|      |          ├─unsupported.md
|      |          ├─malloc_removed
|      ├─Loops
|      |   ├─Loops.md
|      |   ├─variable_bound_loops
|      ├─Data_type
|      |     ├─data_type.md
|      |     ├─Standard_C
|      |     |     ├─using_float_and_double
|      |     |     ├─using_fixed_point
|      |     |     ├─fixed_point_sqrt
|      |     |     ├─C++
|      |     ├─pointers
|      |     |    ├─stream_good
|      |     |    ├─stream_better
|      |     ├─Arbitrary_Precision
|      |     |          ├─using_arbitrary_precision_casting
|      |     |          ├─using_arbitrary_precision_arith
|      ├─Arrays
|      |   └arrays.md
├─Defining_Interface
|         ├─Streaming
|         |     ├─using_axi_stream_with_struct
|         |     ├─using_axi_stream_with_side_channel_data
|         |     ├─using_axi_stream_no_side_channel_data
|         |     ├─using_axis_array_stream_no_side_channel_data
|         |     ├─using_array_of_streams
|         |     ├─axi_stream_to_master
|         ├─Register
|         |    ├─using_axi_lite_with_user_defined_offset
|         |    ├─using_axi_lite
|         ├─Memory
|         |   ├─memory_interface.md
|         |   ├─using_axi_master
|         |   ├─rom_lookup_table_math
|         |   ├─ram_uram
|         |   ├─memory_bottleneck
|         |   ├─max_widen_port_width
|         |   ├─manual_burst
|         |   |      ├─manual_burst_with_conditionals
|         |   |      ├─manual_burst_example
|         |   |      |          ├─manual_burst_inference_success
|         |   |      |          ├─auto_burst_inference_failure
|         |   ├─ecc_flags
|         ├─Array
|         |   └array_interface.md
├─Optimizing_for_Throughput
|             ├─Unroll
|             |   └README.md
|             ├─Pipelining
|             |     ├─pipeline.md
|             |     ├─Loops
|             |     |   ├─using_free_running_pipeline
|             |     |   ├─pipelined_loop
|             |     |   ├─perfect_loop
|             |     |   ├─imperfect_loop
|             |     ├─Functions
|             |     |     ├─function_instantiate
|             ├─Loop_tripcount
|             |       └README.md
|             ├─Dependence
|             |     └README.md
|             ├─Dataflow
|             |    ├─Channels
|             |    |    ├─Vitis
|             |    |    ├─using_stream_of_blocks
|             |    |    ├─using_pipos
|             |    |    ├─using_fifos
|             |    ├─Bypassing
|             |    |     ├─output_bypass
|             |    |     ├─middle_bypass
|             |    |     ├─input_bypass
|             ├─Array_partition
|             |        └README.md
|             ├─Aggregation_Disaggregation
|             |             ├─struct_ii_issue
|             |             ├─disaggregation_of_axis_port
|             |             ├─auto_disaggregation_of_struct
|             |             ├─aggregation_of_struct
|             |             ├─aggregation_of_nested_structs
|             |             ├─aggregation_of_m_axi_ports
├─Optimizing_for_Latency
|           ├─Opt_latency.md
|           ├─Loop_merge
|           |     └README.md
|           ├─Loop_flatten
|           |      └README.md
|           ├─Bind_op
|           |    └README.md
├─Optimizing_for_Area
|          ├─Opt_area.md
|          ├─Inline
|          |   └README.md
|          ├─Function_instantiate
|          |          └README.md
|          ├─Bind_storage
|          |      └README.md
|          ├─Array_reshape
|          |       └README.md
├─Optimizing_Logic
|        ├─Optimizing Logic Expressions
|        |              └README.md
|        ├─Inferring Shift Registers
|        |             └README.md
├─Application
|      ├─DSP
|      |     ├─CORDIC
|      |     ├─FFT
|      ├─Computer Vision
|      |     ├─Sobel
|      |     ├─canny
|      ├─Neural Network
|      |     ├─Systolic Array
|      |     ├─Convolutional neural network(CNN)




