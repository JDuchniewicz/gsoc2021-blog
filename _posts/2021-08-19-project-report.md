---
layout: post
title: "Project Report"
date: 2021-08-19
categories: GSoC updates
---

# About
* **Student Name:** [Jakub Duchniewicz](https://jduchniewicz.com/)
* **Mentors:** [Iain Hunter](http://www.hunterembedded.co.uk/) and [Hunyue Yau](http://hy-research.com/)
* **GSoC entry link:** [Project#5734780890513408](https://summerofcode.withgoogle.com/dashboard/project/5734780890513408/overview/)
* **Blog and wiki link:** here :)
* **Project link:** [GPGPU with GLES](https://github.com/JDuchniewicz/GPGPU-with-GLES)
* **Introductory video:** [here](https://www.youtube.com/watch?v=I5FnOTc8OP8)
* **Proposal link:** [here](https://elinux.org/BeagleBoard/GSoC/2021_Proposal/GPGPU_with_GLES)

# Introduction
Since BeagleBoards are heterogeneous platforms, why not use them to their full extent? 

Until this project the GPU block lied mostly useless and could not assist with any computations due to non-technical reasons. However, now **OpenGL ES 2.0** is being used in conjunction with **EGL** to perform computations in the shader code. The GPU's targetted by this library are from the SGX 5xx family and they are readily available on both BBB and BBAI/X15. This allows for using the library in almost all models of the Beagle family.

With another computing unit, one can now offload some code to the GPU (remembering about data transfer overhead) and meanwhile continue with computations on the CPU. The process of extending the library with additional computations is quite straightforward and requires adding a relevant shader (sometimes unrolled due to limitations of GLSL compiler) and wrapping it in some glue code.

The two main blocks of the API is the _single-shot_ and _chain_ API, which allow for either a single opertaion or a sequence of opertaions to be performed on the input data.

# Detailed Implementation
As mentioned above, the API relies on using **OpenGL ES 2.0** for performing the computations in shaders. With the newer version of this GPU bindings (notably 3.0) we could just use a _compute shader_ and be done with everything - just present the data to our shader code by means of _uniforms_ and _attributes_. However, with the older API we need to do the computations in the _fragment shader_.

**In order to do our computations we need to undertake a number of steps, briefly outlined below:**
1. Context creation
2. Context binding
3. Framebuffer Object creation
4. Transferring of the data to the GPU
5. Computations
6. Transferring the data back to the CPU or chaining

With these steps our data is transformed by whatever operation we specify in our shader code.



For more details, please refer to the [_Library Innards_ blog post](https://jduchniewicz.github.io/gsoc2021-blog/posts/2021/07/22/library-innards.html).

# Achieved Milestones
The project achieved its 
# Friction Points
No project is without its challenges and blockers. For this project the most prominent one is the inability to render headless and thus necessity to use a dummy plug which emulates a connected display. This is however nearly solved thanks to [help from Omar](https://forums.imgtec.com/t/headless-rendering-with-pvr-sgx530-egl-opengl-contination/3413/2) from Imagination team.

Another issue which took a lot of time to debug was receiving just a small slice of input data on output. [This turned out to be a missing call to `glViewport()`](https://jduchniewicz.github.io/gsoc2021-blog/gsoc/updates/2021/07/07/week-5.html) which is responsible for setting up the x and y coordinates of our framebuffer.

Also, I was unable to render using a 32-bit context due to some driver issues, which yet again was implemented using a workaround suggested by Omar [in this query](https://forums.imgtec.com/t/sgx530-argb8888-support/3403). Interestingly, the shader code used all 32-bits no matter which context was selected, which is a head-scratcher.

Lastly, since the _glslcompiler_ on the BBB is unwilling to compile convolved loops, some operations had to be unrolled when computed on the BBB. This is mostly a nuissance and a small annoyance but let's face it - you only write this code once :)

# TODOs
Though the project is mostly complete, it could benefit from some additions. Notably, more functions could be added, such as:
* Matrix multiplication
* 1D convolution
* Some ML/DL specific operations

Also some extensions and niceties could be added:
* Fix paths to shaders to be non-relative and not hardcoded
* Debug weird glslcompiler memory management issues (sometimes the complier throws errors and has issues with parsing the shader code!?)
* Allowing users to provide their own shaders without editing the library code
* Create Rust language bindings and add some examples in Rust programming language

# Benefit
_With this project, the beagleboard.org community is now able to use the GPU as a regular computation block, thus having a fully heterogeneous platform at the tips of their hands. Extensible and succint API will allow for easily extending it with additional operations and hopefully basing some other libraries which need GPU's computing power and parallellism on this library. Example of such a library is Tensorflow Lite, a ML and DL acceleration library._

# References
