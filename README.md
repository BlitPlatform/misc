# GPU Acceleration Task: Description & Instructions
An **antenna simulation tool** is being developed, for designing antennas. The tool works by getting a **geometry model** as an input (i.e. a CAD model which theoretically consists of an infinite number of points), that may look something like this:

![image](https://user-images.githubusercontent.com/25392776/177608442-c2307cb8-1702-41fb-8811-4d31f156f0be.png)

We then split the model into many small "pieces"/cuboids/cells, by applying an appripriate grid on each axis. We call the result of this gridding process a **mesh**:

![image](https://user-images.githubusercontent.com/25392776/177608592-7d97ac33-5d1e-4624-a782-849ac40e37cb.png)

The end result of the meshed geometry would look a bit "pixelated"/"blocky" (depending on how finely we have meshed our geometry), and will therefore not be a perfect representation of our geometry **(but it's good enough)**. However, now that we have a discrete amount of cells, it is easier to solve physics equations on each cell and get a result for the radiating characteristics of our antenna.

To summarize, the computationally-expensive part (called the **"solver"** or **"engine"**, which basically solves physics/electromagnetics equations) essentially takes the meshed geometry (so grids are translated to arrays), and using some array operations inside certain loops, the solver finishes and outputs out the results of the simulation.

## Goal
Our goal is to speed the solver up by using GPU acceleration (using e.g. CUDA). Since it is based on array operations, we are confident GPU acceleration can offer a significant boost in performance, so that we can get our antenna simulation results a lot faster.

## Code
The entire codebase of the tool can be found here: https://github.com/BlitPlatform/openEMS-Project/tree/develop

The code you'll be focusing on is in the [`FDTD`](https://github.com/BlitPlatform/openEMS-Project/tree/develop/openEMS/FDTD) folder (FDTD stands for finite-difference time-domain; a simulation method to solve electromagnetic equations). This is the computationally-intensive part, which requires acceleration.

The main part of code that performs the array operations (which can be few or many and take minutes or hours depending on the size of the problem) can be found here: https://github.com/BlitPlatform/openEMS-Project/blob/develop/openEMS/FDTD/engine.cpp

There is also an SSE implementation: https://github.com/BlitPlatform/openEMS-Project/blob/develop/openEMS/FDTD/engine_sse.cpp

This particular file uses [SSE instructions](https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions) on a CPU, so it is very slow. Similarly, there are other C++ files on the same folder which are meant for different types of acceleration (depending on the architecture of the machine running the solver engine), such as [`engine_multithread.cpp`](https://github.com/BlitPlatform/openEMS-Project/blob/develop/openEMS/FDTD/engine_multithread.cpp), which supports multithreading.

(You don't have to be an expert in SSE; you only have to understand how the C++ code performs the array operations, so that you can apply GPU acceleration to them and speed things up - it might in fact be better if you completely ignore the SSE version and only focus on the basic version of the solver (`engine.cpp`). The basic engine is by far the slowest (since it doesn't use SSE or multithreading), but it is easier to study/understand how it works and create the GPU code from, so it might be better to work from the basic engine instead of the SSE)

## Installation
We have created a built script for the tool ([`build_solver.sh`](https://raw.githubusercontent.com/BlitPlatform/compute-servers/main/build_solver.sh)) so that you can directly use it as follows (the following was tested on Ubuntu 20.04, but it should work on any Debian-based GNU/Linux OS):
```sh
mkdir gpu_task
cd gpu_task
wget https://raw.githubusercontent.com/BlitPlatform/compute-servers/main/build_solver.sh
chmod +x build_solver.sh
./build_solver.sh --setup # You only have to run this on your machine once
./build_solver.sh
```
(Note: it can take a bit of time to compile everything on your machine, so give it some time)

The above script clones a repository and compiles everything for you.

### Running benchmark tests
To test how fast the solver runs, you can use the same `build_solver.sh` script with the `--benchmark` tag (it just runs an example case for 20 sec and gives you the result):
```sh
./build_solver.sh --benchmark
```
```py
============================================
Running benchmark (this will take 20 sec)...
Benchmark complete.
============================================
multithreading engine
============================================
Speed:   10.2 MC/s (6.849e-03 s/TS)
Speed:   10.9 MC/s (6.444e-03 s/TS)
Speed:   11.1 MC/s (6.302e-03 s/TS)
Speed:   11.9 MC/s (5.898e-03 s/TS)
============================================
```
(MC/s stands for million cells per second; the higher the number, the faster the solver performs)

By default, it uses the "fastest" available engine, but that is often inaccurately determined. We recommend picking a C++ file to apply your GPU acceleration to (e.g. the basic `engine.cpp` solver), and specifying that to the benchmark using the `--engine` tag:

```sh
./build_solver.sh --benchmark --engine basic
```

These are the `--engine` options you can specify:
- `fastest`
- `basic`
- `sse`
- `sse-compressed`
- `multithreaded`

## Commiting
All changes to the code (even if they are not tested) can be applied to the `develop` branch: https://github.com/BlitPlatform/openEMS-Project/tree/develop (no one uses this repository, so feel free to experiment freely). You can apply your GPU-accelerated (or any other) changes and use the same script to re-build everything (and then use `--benchmark` to see if you managed to improve the performance). The build script reads data from the `develop` branch, so make sure the changes are applied there.

(If you wish to test things locally instead of committing to the git repository, that is also fine)

You are also encouraged to introduce debugging techniques to investigate the array sizes, what kind of operations take place, or any other information that might be useful to you to test.
