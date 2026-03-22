![License](https://img.shields.io/github/license/phineas-pta/NVIDIA-win?logo=creativecommons)
![Platform](https://img.shields.io/badge/Platform-win_64-0078D4?logo=windows)
![Cuda](https://img.shields.io/badge/CUDA-v13.2-76B900?logo=nvidia)
![Cudnn](https://img.shields.io/badge/cuDNN-v9.20-76B900?logo=nvidia)
![Tensorrt](https://img.shields.io/badge/TensorRT-v10.16-76B900?logo=nvidia)
![Python](https://img.shields.io/badge/python-v3.1x-3776AB?logo=python)
![VS](https://img.shields.io/badge/Visual_Studio-v18_2026-5C2D91?logo=visualstudio)
![Cmake](https://img.shields.io/badge/CMake-v4-064F8C?logo=cmake)

# NVIDIA-win

how to install the NVIDIA’s deep learning stack on Windows: CUDA toolkit + cuDNN + TensorRT,<br />also various attempts to compile deep learning libraries with cuda

only for Windows 10/11 + recent RTX cards recommended

all comands here are for cmd, if u prefer pwsh must change to adapt

some values are censored with black box ███ so u have to fill in with your use case

**1st of all:** 👉👉👉 [install CUDA toolkit + cuDNN + TensorRT](NVIDIA-win.md) 👈👈👈

my attempts to build various deep learning libraries:
1. [build onnxruntime](test01-onnxruntime.md) ✅
2. [build whisper.cpp with cuBLAS](test02-whisper.cpp-cublas.md) ✅
3. ~~build CTranslate2 to use faster-whisper~~ ❌
4. ~~build triton to use xformers~~ ❌
5. ~~try out TensorRT-LLM~~ ❌
6. [build OpenCV with CUDA](test06-opencv.md) ✅
7. ~~build Torch-TensorRT~~ ❌
8. [build llama.cpp with cuBLAS](test08-llama.cpp-cublas.md) ✅
9. [build Stan with OpenCL](test09-stan-opencl.md) ✅

all above guides are under CC-BY-4.0

more complicated tweaking with TensorRT (under GPLv3)
- build Real-ESRGAN with TensorRT on Windows: https://github.com/phineas-pta/RealESRGAN-trt-win ✅
- run Stable Diffusion XL with TensorRT natively on Windows: https://github.com/phineas-pta/SDXL-trt-win ✅
