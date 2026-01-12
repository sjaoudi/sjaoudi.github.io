---
author: 
title: CUDA Raytracer - GPU-Accelerated 3D Mesh Rendering
date: 2017-06-01
tags: 
    - coding samples
categories:
    # - heyo
showToc: false
---

[Code Link](https://github.com/sjaoudi/cuda-raytracer)

A first venture into GPU-Accelerated programming!

**Overview**
- This is a CUDA C++ raytracer that loads STL meshes (via Assimp), computes ray–triangle intersections on the GPU, and visualizes with OpenGL/GLUT.
- Achieves big speedups over CPU tracing by running per-pixel work in parallel on the GPU.

<!-- **Performance (GeForce GTX 980 Ti)**
- ~390 ms for ~3.7k triangles
- ~28 s for ~200k triangles
- ~1:45 for ~716k triangles -->

**Key Components**
- Ray tracing core and kernel: [ray.cu](https://github.com/sjaoudi/cuda-raytracer/blob/main/ray.cu)
- Math and scene structs: [structs.cpp](https://github.com/sjaoudi/cuda-raytracer/blob/main/structs.cpp)
- STL import and camera setup: [importer_stuff.cpp](https://github.com/sjaoudi/cuda-raytracer/blob/main/importer_stuff.cpp)
- Triangle construction helpers: [constructors.cpp](https://github.com/sjaoudi/cuda-raytracer/blob/main/constructors.cpp)
- CUDA utilities and error macros: [book.h](https://github.com/sjaoudi/cuda-raytracer/blob/main/book.h), [cudahelpers.h](https://github.com/sjaoudi/cuda-raytracer/blob/main/cudahelpers.h)
- Bitmap display via OpenGL: [cpu_bitmap.h](https://github.com/sjaoudi/cuda-raytracer/blob/main/cpu_bitmap.h)
- CPU/GPU timers: [timer.h](https://github.com/sjaoudi/cuda-raytracer/blob/main/timer.h)

**Rendering Pipeline**
1) Import STL → triangulate and gather bounds.  
2) Auto-fit camera and view plane to the mesh.  
3) Upload triangles, lights, and scene constants to CUDA.  
4) Kernel per pixel: build a ray, find the closest triangle, shade with Phong, test shadows, write RGBA.  
5) Copy framebuffer back and display with GLUT.

**Lighting Model**
- Implements the [Phong Lighting Model](https://en.wikipedia.org/wiki/Phong_reflection_model)

**Intersection Approach**
- Ray–plane hit, then an inside-triangle test (“left-of” checks on edges) to confirm the hit

**Camera Auto-Fit**
- Computes mesh bounds, picks the larger of width/height for the view plane, centers the origin, and pulls the eye back along z so the model fills the frame.

**CUDA Setup Notes**
- Scene and lights in constant memory for fast reads
- Triangles in device memory
- Kernel launch: grids(32,32), threads(25,25) for a 1024×1024 image
- Colors clamped and packed into RGBA before copying back

**Extension Ideas**
- Add reflections/refractions or textures
- Experiment with soft shadows or ambient occlusion
- Tweak camera/background for nicer renders
<!-- 
**One-Line Hook**
> “We load an STL, auto-frame the camera, and let thousands of CUDA threads chew through ray–triangle hits. Four lights, a tiny Phong shader, and a GLUT window later, the mesh shows up in under a second for small models and under two minutes for ~700k triangles on a 980 Ti.” -->

## Benchmarks

Card: GeForce GTX 980 Ti

| model | time (ms) | #triangles |
|---|---:|---:|
| corgi.stl | 390 | 3740 |
| snowman.stl | 519 | 5558 |
| kangarphin.stl | 27705 (28s) | 200098 |
| kang.stl | 105,000 (1 min, 45s) | 716044 |

**Images**

![](/images/cuda/1-corgi-front.png)
Corgi
<br><br><br>

![](/images/cuda/1-corgi-side.png)
Corgi
<br><br><br>

![](/images/cuda/2-snowman-front.png)
Snowman
<br><br><br>

![](/images/cuda/2-snowman-side.png)
Snowman
<br><br><br>

![](/images/cuda/3-kangarphin-front.png)
Kangarphin
<br><br><br>

![](/images/cuda/3-kangarphin-side.png)
Kangarphin
<br><br><br>

![](/images/cuda/kangaroo-front.png)
Kangaroo
<br><br><br>

![](/images/cuda/kangaroo-side.png)
Kangaroo
<br><br><br>