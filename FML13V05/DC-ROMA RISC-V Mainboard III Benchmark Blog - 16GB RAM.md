# Benchmarking the DC-ROMA RISC-V Mainboard III - RAM 16GB

## Introduction
The DC-ROMA RISC-V Mainboard III represents the next generation of open RISC-V computing for developers. Powered by the SpacemiT K3 RISC-V AI SoC, the platform combines modern Linux support with AI acceleration capabilities in a modular Framework Laptop 13 form factor.
In this blog, we explore real-world performance across:
-  Ubuntu Desktop 26.04 
-  Software development workflows 
-  Desktop responsiveness 
-  Linux-native benchmarks 
## Test Configuration
### Hardware
-  DC-ROMA RISC-V Mainboard III 
-  SpacemiT K3 RISC-V AI SoC 
-  8-core CPU up to 2.5GHz 
-  16GB RAM 
-  1TB NVMe SSD 
-  Framework Laptop 13 chassis 
### Software
-  Ubuntu Desktop 26.04 
-  Linux Kernel 6.18.3-generic
-  GCC version 15.2.0 
<img width="1280" height="853" alt="image" src="https://github.com/user-attachments/assets/7135f942-8723-47e7-9931-e944cf82835a" />



## Hardware
### CPU
- Geekbench 6: https://browser.geekbench.com/v6/cpu/17915638
<img width="1280" height="645" alt="image" src="https://github.com/user-attachments/assets/607005cd-8ca1-4466-8866-65c676187bc3" />



### GPU
```
roma@roma ~> glmark2-es2-wayland
=======================================================
    glmark2 2023.01
=======================================================
    OpenGL Information
    GL_VENDOR:      Imagination Technologies
    GL_RENDERER:    PowerVR B-Series BXM-4-64
    GL_VERSION:     OpenGL ES 3.2 build 24.2@6603887
    Surface Config: buf=32 r=8 g=8 b=8 a=8 depth=24 stencil=0 samples=0
    Surface Size:   800x600 windowed
=======================================================
[build] use-vbo=false: FPS: 492 FrameTime: 2.035 ms
[build] use-vbo=true: FPS: 2635 FrameTime: 0.380 ms
[texture] texture-filter=nearest: FPS: 2822 FrameTime: 0.354 ms
[texture] texture-filter=linear: FPS: 2774 FrameTime: 0.361 ms
[texture] texture-filter=mipmap: FPS: 2579 FrameTime: 0.388 ms
[shading] shading=gouraud: FPS: 2209 FrameTime: 0.453 ms
[shading] shading=blinn-phong-inf: FPS: 2106 FrameTime: 0.475 ms
[shading] shading=phong: FPS: 1803 FrameTime: 0.555 ms
[shading] shading=cel: FPS: 1657 FrameTime: 0.604 ms
[bump] bump-render=high-poly: FPS: 899 FrameTime: 1.113 ms
[bump] bump-render=normals: FPS: 2824 FrameTime: 0.354 ms
[bump] bump-render=height: FPS: 2569 FrameTime: 0.389 ms
[effect2d] kernel=0,1,0;1,-4,1;0,1,0;: FPS: 1575 FrameTime: 0.635 ms
[effect2d] kernel=1,1,1,1,1;1,1,1,1,1;1,1,1,1,1;: FPS: 574 FrameTime: 1.744 ms
[pulsar] light=false:quads=5:texture=false: FPS: 3313 FrameTime: 0.302 ms
[desktop] blur-radius=5:effect=blur:passes=1:separable=true:windows=4: FPS: 472 FrameTime: 2.122 ms
[desktop] effect=shadow:windows=4: FPS: 1711 FrameTime: 0.585 ms
[buffer] columns=200:interleave=false:update-dispersion=0.9:update-fraction=0.5:update-method=map: FPS: 234 FrameTime: 4.275 ms
[buffer] columns=200:interleave=false:update-dispersion=0.9:update-fraction=0.5:update-method=subdata: FPS: 232 FrameTime: 4.324 ms
[buffer] columns=200:interleave=true:update-dispersion=0.9:update-fraction=0.5:update-method=map: FPS: 321 FrameTime: 3.119 ms
[ideas] speed=duration: FPS: 1351 FrameTime: 0.740 ms
[jellyfish] <default>: FPS: 824 FrameTime: 1.214 ms
[terrain] <default>: FPS: 54 FrameTime: 18.677 ms
[shadow] <default>: FPS: 952 FrameTime: 1.051 ms
[refract] <default>: FPS: 119 FrameTime: 8.454 ms
[conditionals] fragment-steps=0:vertex-steps=0: FPS: 3451 FrameTime: 0.290 ms
[conditionals] fragment-steps=5:vertex-steps=0: FPS: 2077 FrameTime: 0.482 ms
[conditionals] fragment-steps=0:vertex-steps=5: FPS: 3425 FrameTime: 0.292 ms
[function] fragment-complexity=low:fragment-steps=5: FPS: 2801 FrameTime: 0.357 ms
[function] fragment-complexity=medium:fragment-steps=5: FPS: 1591 FrameTime: 0.629 ms
[loop] fragment-loop=false:fragment-steps=5:vertex-steps=5: FPS: 2697 FrameTime: 0.371 ms
[loop] fragment-steps=5:fragment-uniform=false:vertex-steps=5: FPS: 2716 FrameTime: 0.368 ms
[loop] fragment-steps=5:fragment-uniform=true:vertex-steps=5: FPS: 2571 FrameTime: 0.389 ms
=======================================================
                                  glmark2 Score: 1769
=======================================================
```

### Memory
```
roma@roma ~> tinymembench
tinymembench v0.4.9 (simple benchmark for memory throughput and latency)

==========================================================================
== Memory bandwidth tests                                               ==
==                                                                      ==
== Note 1: 1MB = 1000000 bytes                                          ==
== Note 2: Results for 'copy' tests show how many bytes can be          ==
==         copied per second (adding together read and writen           ==
==         bytes would have provided twice higher numbers)              ==
== Note 3: 2-pass copy means that we are using a small temporary buffer ==
==         to first fetch data into it, and only then write it to the   ==
==         destination (source -> L1 cache, L1 cache -> destination)    ==
== Note 4: If sample standard deviation exceeds 0.1%, it is shown in    ==
==         brackets                                                     ==
==========================================================================

 C copy backwards                                     :   5727.7 MB/s (2.2%)
 C copy backwards (32 byte blocks)                    :   5343.5 MB/s (2.7%)
 C copy backwards (64 byte blocks)                    :   4875.4 MB/s (3.3%)
 C copy                                               :   5833.8 MB/s (3.0%)
 C copy prefetched (32 bytes step)                    :   3002.8 MB/s (2.1%)
 C copy prefetched (64 bytes step)                    :   2981.1 MB/s (1.8%)
 C 2-pass copy                                        :   5480.5 MB/s (1.4%)
 C 2-pass copy prefetched (32 bytes step)             :   2339.6 MB/s (0.9%)
 C 2-pass copy prefetched (64 bytes step)             :   2345.4 MB/s (0.4%)
 C fill                                               :  16147.1 MB/s (0.4%)
 C fill (shuffle within 16 byte blocks)               :  16261.1 MB/s (0.6%)
 C fill (shuffle within 32 byte blocks)               :  16185.9 MB/s (0.4%)
 C fill (shuffle within 64 byte blocks)               :  14054.9 MB/s (0.4%)
 ---
 standard memcpy                                      :   6284.3 MB/s (2.5%)
 standard memset                                      :  16634.5 MB/s (1.3%)

==========================================================================
== Memory latency test                                                  ==
==                                                                      ==
== Average time is measured for random memory accesses in the buffers   ==
== of different sizes. The larger is the buffer, the more significant   ==
== are relative contributions of TLB, L1/L2 cache misses and SDRAM      ==
== accesses. For extremely large buffer sizes we are expecting to see   ==
== page table walk with several requests to SDRAM for almost every      ==
== memory access (though 64MiB is not nearly large enough to experience ==
== this effect to its fullest).                                         ==
==                                                                      ==
== Note 1: All the numbers are representing extra time, which needs to  ==
==         be added to L1 cache latency. The cycle timings for L1 cache ==
==         latency can be usually found in the processor documentation. ==
== Note 2: Dual random read means that we are simultaneously performing ==
==         two independent memory accesses at a time. In the case if    ==
==         the memory subsystem can't handle multiple outstanding       ==
==         requests, dual random read has the same timings as two       ==
==         single reads performed one after another.                    ==
==========================================================================

block size : single random read / dual random read, [MADV_NOHUGEPAGE]
      1024 :    0.1 ns          /     0.0 ns
      2048 :    0.1 ns          /     0.0 ns
      4096 :    0.0 ns          /     0.0 ns
      8192 :    0.1 ns          /     0.0 ns
     16384 :    0.0 ns          /     0.0 ns
     32768 :    0.1 ns          /     0.1 ns
     65536 :    0.0 ns          /     0.0 ns
    131072 :    3.7 ns          /     8.7 ns
    262144 :    7.5 ns          /    12.1 ns
    524288 :    9.2 ns          /    13.5 ns
   1048576 :   10.2 ns          /    14.1 ns
   2097152 :   10.9 ns          /    14.4 ns
   4194304 :   25.2 ns          /    40.1 ns
   8388608 :   88.9 ns          /   102.7 ns
  16777216 :  128.6 ns          /   161.4 ns
  33554432 :  149.7 ns          /   173.5 ns
  67108864 :  160.3 ns          /   179.5 ns

block size : single random read / dual random read, [MADV_HUGEPAGE]
      1024 :    0.1 ns          /     0.0 ns
      2048 :    0.0 ns          /     0.0 ns
      4096 :    0.1 ns          /     0.1 ns
      8192 :    0.0 ns          /     0.1 ns
     16384 :    0.0 ns          /     0.0 ns
     32768 :    0.0 ns          /     0.1 ns
     65536 :    0.0 ns          /     0.0 ns
    131072 :    3.6 ns          /     8.7 ns
    262144 :    5.7 ns          /    10.9 ns
    524288 :    6.7 ns          /    11.5 ns
   1048576 :    7.1 ns          /    12.0 ns
   2097152 :    7.8 ns          /    12.2 ns
   4194304 :   11.3 ns          /    15.2 ns
   8388608 :   77.6 ns          /   120.5 ns
  16777216 :  110.2 ns          /   148.7 ns
  33554432 :  126.7 ns          /   154.8 ns
  67108864 :  134.7 ns          /   159.7 ns
```

### Disk
- ZHITAI TiPlus7100 1TB, iozone mark testing

|Test Type|Result(MB/s)|
|:---:|:---:|
|iozone 4K Random Write|≈204.74|
|iozone 4K Random Read |≈78.56|
|iozone 4K Sequential Write|≈114.99|
|iozone 4K Sequential Read|≈211.41|
|iozone 1M Random Write|≈2054.27|
|iozone 1M Random Read|≈1817.73|
|iozone 1M Sequential Write|≈2104.90|
|iozone 1M Sequential Read|≈1934.82|

## Desktop Experience
### Boot Time
- Cold boot: Startup finished in 5.824s (kernel) + 11.824s (userspace) = 17.648s
### Browser Performance
- Speedometer score:2.71
<img width="2256" height="1504" alt="image" src="https://github.com/user-attachments/assets/ab770e56-c25b-4fb0-a76f-5869c395fed2" />


### Multitasking
- Chromium + VS Code + terminal + local AI inference 
- Multiple tasks run simultaneously, window switching is smooth, and process switching is fluid.

<img width="1280" height="853" alt="image" src="https://github.com/user-attachments/assets/c9fe72f5-aad4-431b-81a3-71d428758bf1" />

<img width="1280" height="853" alt="image" src="https://github.com/user-attachments/assets/0cb096b3-a26b-42e7-bdac-ce465b46e80e" />




## Key Takeaways
The DC-ROMA RISC-V Mainboard III is not just an experimental development board — it is becoming a practical Linux workstation for developers exploring the future of open computing and AI on RISC-V.
Key highlights:
- Smooth Ubuntu 26.04 desktop experience 
- Practical developer workflows 
- AI inference capabilities 
- Strong ecosystem momentum
- 16GB memory configuration tested successfully, with 32GB testing planned for the next blog update
