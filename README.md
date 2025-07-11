# Edge Detection using NVIDIA NPP with CUDA

## Overview

This project demonstrates the use of NVIDIA Performance Primitives (NPP) library with CUDA to perform edge detection. The goal is to utilize GPU acceleration to efficiently find edges in a given image by using a sobel operator. The project is a part of the CUDA at Scale for the Enterprise course from coursera. It is based on the [image rotation project](https://github.com/PascaleCourseraCourses/CUDAatScaleForTheEnterpriseCourseProjectTemplate).

## Code Organization

```bin/```
After building using make, this folder will hold the executable `edgeDetection`.

```data/```
This folder holds example data, an image [`Lena.png`](./data/Lena.png). By default, the output is stored in the same folder.

```src/```
Holds the source code of the project

```README.md```
This file describes the project. It holds human-readable instructions for building and executing the code.

```Makefile```
This file contains the instructions for building the project using the make utility. It specifies the dependencies, compilation flags, and the target executable to be generated.

## Key Concepts

The application performs multiple steps:
1. Loading the data from disk (either image, or video)
2. Manipulate image or single frames from the video
3. Store manipulated data back to disk, either as an image, or as a video file.

The image processing makes use of several NPP functions:
1. Convert RGB image to grayscale `nppiRGBToGray_8u_AC4C1R`
2. Apply a sobel operator to detect edges: `nppiFilter32f_8u16s_C1R` to compute a convolution, 
`nppiAbs_16s_C1R` to compute the absolute value (only keep positive values for the edges), and 
`nppiConvert_16s8u_C1R` for converting back to 8 bit grayscale. This step is repeated two times,
for horizontal edges, as well as for vertical edges, by providing two different kernels.
3. Combine the horizontal and vertical edges into a single grayscale image, simply by bitwise or
`nppiOr_8u_C1R`
4. Finally multiplying the grayscale edges image with the original RGB image, to colorize the edges.
To do that, first broadcast the grayscale image into an RGB image (`nppiCopy_8u_C1C4R`, `nppiSet_8u_C4CR`), 
and finally multiply both (`nppiMul_8u_C4RSfs`).

## Supported OSes

The project was testet on Ubuntu 24.04. 

## Supported CPU Architecture

The project was tested on x86_64.

## CUDA APIs involved

NVIDIA Performance Primitives (NPP)

## Dependencies needed to build/run
* [FreeImage](https://freeimage.sourceforge.io/) On Ubuntu, `apt install libfreeimage-dev` 
* [NPP](https://developer.nvidia.com/npp) Need to [install NVidia CUDA Toolkit](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)
* [opencv](https://opencv.org/) `apt install libopencv-dev`
* [cuda-samples](https://github.com/NVIDIA/cuda-samples) You need to adapt the [`Makefile`](./Makefile) accordingly

## Prerequisites

Download and install the [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads) for your corresponding platform. The project was tested with CUDA 11.8 and 12.5. 
Make sure the dependencies mentioned in [Dependencies]() section above are installed.

## Build and Run

The project has been tested on Ubuntu 24.04. There is a [`Makefile`](./Makefile), therefore the project can be built using
```
$ make
```

## Running the Program
You can run the program using the following command:

```bash
make run
```

This command will execute the compiled binary, applying the edge filter on the example input image (Lena.png), and save the result as Lena_edge.png in the data/ directory.

If you wish to run the binary directly with custom input/output files, you can use:

```bash
./bin/edgeDetection --input data/Lena.png --output data/Lena_edges.png
```

You can also process a video file. The i/o is implemented using opencv:

```bash
./bin/edgeDetection --input some_input.mp4 --output edges_video.mp4
```

- Cleaning Up
To clean up the compiled binaries and other generated files, run:


```bash
make clean
```

This will remove all files in the bin/ directory.
