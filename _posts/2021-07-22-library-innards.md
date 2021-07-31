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

This post will feature a lot of abbreviations which may be cryptic at first (or not if you are familiar with the Linux graphics stack). If you feel you are stuck and do not understand them, please refer to [this post](https://blog.mecheye.net/2012/06/the-linux-graphics-stack/). (I of course did not find all of my answers there but at least had a stronger understanding thanks to it).

### Context creation
When we usually render something we need a _context_ which, citing Apple's developer pages is:

> a container for state information. When you designate a rendering context as the current rendering context, subsequent OpenGL commands modify that context’s state, objects attached to that context, or the drawable object associated with that context. The actual drawing surfaces are never owned by the rendering context but are created, as needed, when the rendering context is actually attached to a drawable object. 

In our case we are using **EGL** to create and manage our context. More specifically we use a pixel buffer which is an off-screen rendering target, which means we do not render to any window and instead save our data in an arbitrary memory location.

The following code creates and uses the context for our library:
```c
// g_helper contains following fields:
typedef struct
{
  EProgramState state;
  GLint ESShaderProgram;
  EGLDisplay display;
  EGLConfig config;
  EGLContext context;
  EGLSurface surface;
  int height;
  int width;
} GLHelper;

GLHelper g_helper;

  g_helper.display = eglGetDisplay((EGLNativeDisplayType)0);
  if (!g_helper.display)
      ERR("Could not create display");
      
  if (eglInitialize(g_helper.display, &major, &minor) == 0)
      ERR("Could not initialize display");

  printf("EGL API version %d.%d\n", major, minor);   

  // checking the extensions and choosing a config is skipped for brevity
  
  if (eglBindAPI(EGL_OPENGL_ES_API) == 0)
    ERR("Could not bind the API"); 
    
  EGLint eglSurfaceAttibutes[] = {
        EGL_WIDTH, width,
        EGL_HEIGHT, height,
        EGL_NONE
  };
  g_helper.surface = eglCreatePbufferSurface(g_helper.display, g_helper.config, eglSurfaceAttibutes);

  if (g_helper.surface == EGL_NO_SURFACE)
      ERR("Could not create EGL surface");

  EGLint eglContextAttributes[] = {
      EGL_CONTEXT_CLIENT_VERSION, 2,
      EGL_NONE
  };

  g_helper.context = eglCreateContext(g_helper.display, g_helper.config, EGL_NO_CONTEXT, eglContextAttributes);
  if (g_helper.context == EGL_NO_CONTEXT)
      ERR("Could not create EGL context");

  if (eglMakeCurrent(g_helper.display, g_helper.surface, g_helper.surface, g_helper.context) != EGL_TRUE) // EGL_NO_SURFACE??
      ERR("Could not bind the surface to context");

  EGLint version = 0;
  eglQueryContext(g_helper.display, g_helper.context, EGL_CONTEXT_CLIENT_VERSION, &version);
  printf("EGL Context version: %d\n", version);
```

With the context set up, we can create our textures and bind them to memory and the _framebuffer object_. But before we do this, one small 

### Framebuffer Object creation

### Data transfers

### Single-shot vs chain API


Nice explanations and graphs of both rendering paths (chaining API as well)

Some explanations of what is going on the HW
