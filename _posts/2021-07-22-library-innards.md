---
layout: post
title: "Library Innards"
date: 2021-07-22
categories: Posts
---

## What exactly happens under the hood

This post aims to delve deeper into how the library works. Topics closely related to **EGL**, **GLES** and **GPU** vs **CPU** computing will be elaborated on here.

First of all, the library's aim is to utilize BBB's GPU block for computations. In **OpenGL ES 3.0** we have _compute shaders_ which can be used to perform computations utilizing GPUs. Also **OpenCL** is used for heterogeneous computations on general platforms, such as PC or embedded devices running Linux operating System. Since BBB's GPU chips (SGX530) don't support **OpenGL ES 3.0** but support it's **2.0** version, we have to utilize  _vertex_ and _fragment_ shaders for our computations.

This is the old-school way of doing **GPGPU** computing before _compute shaders_ were introduced. The data goes through _vertex shader_ and then the actual computation happen in _fragment shader_. There of course remain some key decision and issues that have to be tackled, such as data format used for computations, size of the data to be computed and some GPU limitations (such as [inability to sample textures in branches on SGX architectures](https://forums.imgtec.com/t/gradient-calculation-inside-a-conditional-block/3404/5?u=jduchniewicz)).

This post will feature a lot of abbreviations which may be cryptic at first, 
### Context creation


### Data transfers

### Single-shot vs chain API


Nice explanations and graphs of both rendering paths (chaining API as well)

Some explanations of what is going on the HW
