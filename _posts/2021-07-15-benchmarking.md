---
layout: post
title: "Benchmarking"
date: 2021-07-15
categories: Posts
---

This post/page is a collection of benchmarks made both on the Beaglebone Black rev. C and my PC with NVIDIA GTX960M GPU.
The operations are performed with 32-bit buffer on both devices. 

## Array addition floating-point
#### Without optimizations
<p float="left">
  <img src="https://raw.githubusercontent.com/JDuchniewicz/gsoc2021-blog/gh-pages/data/array_addition_float/array_addition_float%20on%20BBB.png" width="45%" />
  <img src="https://raw.githubusercontent.com/JDuchniewicz/gsoc2021-blog/gh-pages/data/array_addition_float/array_addition_float%20on%20host%20PC.png" width="45%" />
</p>
It can be seen that in case of BBB the GPU is always slower than CPU and this is exacerbated with the increasing size of the data transferred to and fro the GPU. The overhead of data transfers is very big, so in future tests I will see how expensive the operations on the GPU have to be to make up for the overhead.

On the PC the differences are much smaller, which shows that the bus transfers between CPU and GPU are much faster. Also the execution time is much faster, because the CPU and GPU are much stronger.

#### -O3 enabled
<p float="left">
  <img src="https://raw.githubusercontent.com/JDuchniewicz/gsoc2021-blog/gh-pages/data/array_addition_float/array_addition_float%20on%20BBB%20-O3.png" width="45%" />
  <img src="https://raw.githubusercontent.com/JDuchniewicz/gsoc2021-blog/gh-pages/data/array_addition_float/array_addition_float%20on%20BBB%20-O3-element.png" width="45%" />
</p>

It can be seen that enabling optimizations made the code execute faster in both cases. Also the time per-element is a good metric because it shows whether the processor is executing optimally or sub-optimally and in our case we can see that the relationship is almost linear in both cases.

Time per-element is shown in logarithmic vertical scale and is increasing exponentially in both cases.

## Array addition fixed-point 16-bit
#### Without optimizations
<p float="left">
  <img src="https://raw.githubusercontent.com/JDuchniewicz/gsoc2021-blog/gh-pages/data/array_addition_fixed16/array_addition_fixed16%20on%20BBB.png" width="45%" />
  <img src="https://raw.githubusercontent.com/JDuchniewicz/gsoc2021-blog/gh-pages/data/array_addition_fixed16/array_addition_fixed16%20on%20host%20PC.png" width="45%" />
</p>

In case of fixed-point operations, the results are very similar to floating-point for the BBB. However, for the host PC GPU execution is significantly slower (probably due to fixed-to-float transformations).

#### -O3 enabled
<p float="left">
  <img src="https://raw.githubusercontent.com/JDuchniewicz/gsoc2021-blog/gh-pages/data/array_addition_fixed16/array_addition_fixed16%20on%20BBB%20-%20O3.png" width="45%" />
  <img src="https://raw.githubusercontent.com/JDuchniewicz/gsoc2021-blog/gh-pages/data/array_addition_fixed16/array_addition_fixed16%20on%20BBB%20-%20O3-element.png" width="45%" />
</p>

For the fixed-point operations, we can see that a significant speedup is achieved on the CPU side, with a slighly lesser speedup on the GPU. Also time-per-element for the GPU is rising which shows that it is less resilient to scalabilty than in the floating-point case.

## 2D convolution floating-point
<p float="left">
  <img src="https://raw.githubusercontent.com/JDuchniewicz/gsoc2021-blog/gh-pages/data/conv2d_float/conv2d_float%20on%20BBB%20-O3.png" width="45%" />
  <img src="https://raw.githubusercontent.com/JDuchniewicz/gsoc2021-blog/gh-pages/data/conv2d_float/conv2d_float%20on%20host%20PC.png" width="45%" />
</p>


The -O3 optimizations are enabled in this benchmark case.

Finally, we can see some operations in which GPU is faster than the CPU (covers up for the data transfer overhead). On my host PC passing the magic mark of 1024 elements causes the GPU to be faster! Unfortunately on the BBB it is not the case and faster operations are required still. The benchmark here is a 3x3 convolution kernel of floats.

<p float="left">
  <img src="https://raw.githubusercontent.com/JDuchniewicz/gsoc2021-blog/gh-pages/data/conv2d_float/conv2d_float%20on%20BBB%20-%20O3-element.png" width="45%" />
</p>

Interestingly, in this case the time per-element for GPU is asymptotic to the one for CPU and can probably surpass it if we could have infinite textures available on our BBB. However, we are limited to 2048x2048 texture sizes.

## Noop operations floating-point
Small introduction - this benchmark is supposed to be a baseline to show how expensive are CPU<=>GPU data transfers on BBB and host PC. This knowledge is valuable when assessing when to use this library and what sizes of the data/complexity of operations to use.

<p float="left">
  <img src="https://raw.githubusercontent.com/JDuchniewicz/gsoc2021-blog/gh-pages/data/noop/noop%20on%20BBB%20-O3.png" width="45%" />
  <img src="https://raw.githubusercontent.com/JDuchniewicz/gsoc2021-blog/gh-pages/data/noop/noop%20on%20BBB%20-%20O3-element.png" width="45%" />
</p>

As we can see, the noops are a cinch to our CPU as the compiler is clever enough to do them all in just a single assembly instruction (probably our friend `mov` or its relative). The situation is much worse in case of GPU, because we have to copy the data over and copy it back to the CPU once we are done with out noop. The problem worsenes with the data size however not very much. The time-per-element metric shows us that this time is fairly linear.

On the PC the problem is quite similar and the CPU noops take literally no time, so I did not include that graph here.

Equipped with this knowledge, we can now deduct the time it takes for the data to travel to the GPU to assess when we should increase our computation complexity to cover up for the data transfer time.

## Chain API testing with 2D convolution 5x5 kernels - 2 repetitions
With the 

-----------
For anyone interested in the actual data that backs up these graphs - [here it is](https://docs.google.com/spreadsheets/d/1CSCAirU1Zo2bp7UKmpXpbvrvRv_IZArDhxX1b4W2Bl4/edit?usp=sharing). 
