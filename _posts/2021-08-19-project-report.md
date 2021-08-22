---
layout: post
title: "Project Report"
date: 2021-08-19
categories: GSoC updates
---

## About
* **Student Name:** [Jakub Duchniewicz](https://jduchniewicz.com/)
* **Mentors:** [Iain Hunter](http://www.hunterembedded.co.uk/) and [Hunyue Yau](http://hy-research.com/)
* **GSoC entry link:** [Project#5734780890513408](https://summerofcode.withgoogle.com/dashboard/project/5734780890513408/overview/)
* **Blog and wiki link:** here :)
* **Project link:** [GPGPU with GLES](https://github.com/JDuchniewicz/GPGPU-with-GLES)
* **Introductory video:** [here](https://www.youtube.com/watch?v=I5FnOTc8OP8)
* **Proposal link:** [here](https://elinux.org/BeagleBoard/GSoC/2021_Proposal/GPGPU_with_GLES)

## Introduction
Since BeagleBoards are heterogeneous platforms, why not use them to their full extent? 

Until this project the GPU block lay mostly useless and could not assist with any computations due to non-technical reasons. However, now **OpenGL ES 2.0** is being used in conjunction with **EGL** to perform computations in the shader code. The GPU's targetted by this library are from the SGX 5xx family and they are readily available on both BBB and BBAI/X15. This allows for using the library in almost all models of the Beagle family.

With another computing unit, one can now offload some code to the GPU (remembering about data transfer overhead) and meanwhile continue with computations on the CPU. The process of extending the library with additional computations is quite straightforward and requires adding a relevant shader (sometimes unrolled due to limitations of GLSL compiler) and wrapping it in some glue code.

The two main blocks of the API is the _single-shot_ and _chain_ API, which allow for either a single opertaion or a sequence of opertaions to be performed on the input data.

For people interested in using the library/contributing, there is a [README section in the library repository](https://github.com/JDuchniewicz/GPGPU-with-GLES/blob/master/README.md). 
### Caveat
As mentioned above, there is a small overhead depending on the sizes of data you want to do the computations on. It is outlined in more detail in [the benchmarking post](https://jduchniewicz.github.io/gsoc2021-blog/_posts/2021-07-15-benchmarking.html)(section noop). The general rule of thumb I adopted is using texture sizes of 2048x2048 (maximum possible size on the BBB) and at least something complex enough to be non-implementable in a single instruction on the CPU (let's face it, they are pretty powerful in terms of branch prediction and instruction prefetching). 

_For example:_

_For a size of 2048x2048 2D 3x3 convolution, the **time consumed by transfers is around 1.5s**, while the actual **convolution takes around 0.5s**._

_On the **CPU it takes 0.8s** to do the convolution **without** doing **any data transfers**._

## Detailed Implementation
As mentioned above, the API relies on using **OpenGL ES 2.0** for performing the computations in shaders. With the newer version of this GPU bindings (notably 3.0) we could just use a _compute shader_ and be done with everything - just present the data to our shader code by means of _uniforms_ and _attributes_. However, with the older API we need to do the computations in the _fragment shader_.

**In order to do our computations we need to undertake a number of steps, briefly outlined below:**
1. Context creation
2. Context binding
3. Framebuffer Object creation
4. Transferring of the data to the GPU
5. Computations
6. Transferring the data back to the CPU or chaining

With these steps our data is transformed by whatever operation we specify in our shader code.

An operation worth mentioning here is the transformation from a quartet of `unsigned bytes` to IEEE754 floating-point format in the shader. This is outlined in the [developer diary post](https://jduchniewicz.github.io/gsoc2021-blog/gsoc/updates/2021/07/07/week-5.html).

For more details, please refer to the [_Library Innards_ blog post](https://jduchniewicz.github.io/gsoc2021-blog/posts/2021/07/22/library-innards.html).

## Achieved Milestones
The project was scoped well as all milestones were achieved. The only milestone which I decided to replace was matrix multiplication (it got replaced with various broadcast operations). I decided not to do any PR's because I am the sole contributor to this library at the time being :)

The project contains a rich developer diary (this page :), documentation in form of [_Library Innards_ blog post](https://jduchniewicz.github.io/gsoc2021-blog/posts/2021/07/22/library-innards.html), where various details describing workings of the library are described. Also a [benchmarking page](https://jduchniewicz.github.io/gsoc2021-blog/_posts/2021-07-15-benchmarking.html) is present to describe how to use the library and how optimal various computations are for different sizes.

Additionally, the project contains the _examples_ and _benchmark_ directories, which contain various self-contained examples and extensible benchmark suite respectively. 

## Friction Points
No project is without its challenges and blockers. For this project the most prominent one is the inability to render headless and thus necessity to use a dummy plug which emulates a connected display. This is however nearly solved thanks to [help from Omar](https://forums.imgtec.com/t/headless-rendering-with-pvr-sgx530-egl-opengl-contination/3413/2) from Imagination team.

Another issue which took a lot of time to debug was receiving just a small slice of input data on output. [This turned out to be a missing call to `glViewport()`](https://jduchniewicz.github.io/gsoc2021-blog/gsoc/updates/2021/07/07/week-5.html) which is responsible for setting up the x and y coordinates of our framebuffer.

Also, I was unable to render using a 32-bit context due to some driver issues, which yet again was implemented using a workaround suggested by Omar [in this query](https://forums.imgtec.com/t/sgx530-argb8888-support/3403). Interestingly, the shader code used all 32-bits no matter which context was selected, which is a head-scratcher.

Lastly, since the _glslcompiler_ on the BBB is unwilling to compile convolved loops, some operations had to be unrolled when computed on the BBB. This is mostly a nuissance and a small annoyance but let's face it - you only write this code once :)

## TODOs
Though the project is mostly complete, it could benefit from some additions. Notably, more functions could be added, such as:
* Matrix multiplication
* 1D convolution
* Some ML/DL specific operations

Also some extensions and niceties could be added:
* Fix paths to shaders to be non-relative and not hardcoded
* Debug weird glslcompiler memory management issues (sometimes the complier throws errors and has issues with parsing the shader code!?)
* Allowing users to provide their own shaders without editing the library code
* Create Rust language bindings and add some examples in Rust programming language
* Integrate DSPs into the library to allow further parallelism of the data
* Proposing the optimal data sizes the user should use with each operation so the data transfer overhead is masked

## Benefit
_With this project, the beagleboard.org community is now able to use the GPU as a regular computation block, thus having a fully heterogeneous platform at the tips of their hands. Extensible and succint API will allow for easily extending it with additional operations and hopefully basing some other libraries which need GPU's computing power and parallellism on this library. Example of such a library is Tensorflow Lite, a ML and DL acceleration library._

## References
This is a curated collection of links I used throughout the project:
* https://blogs.igalia.com/itoral/2014/07/29/a-brief-introduction-to-the-linux-graphics-stack/ - contains comprehensive introduction to Linux Graphics stack
* http://www.vizitsolutions.com/portfolio/webgl/gpgpu/speedBumps.html - the whole page is a good resource on GPGPU computing and IEEE754 float transformations
* https://blogs.igalia.com/elima/2016/10/06/example-run-an-opengl-es-compute-shader-on-a-drm-render-node/ - shows an alternative approach (which I am using on the host) to PBuffers rendering
* https://pabloariasal.github.io/2018/02/19/its-time-to-do-cmake-right/ - general CMake setup post - made setting up this library much easier (and taught me exporting targets by the way!)
