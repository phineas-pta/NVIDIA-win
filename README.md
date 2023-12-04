![License](https://img.shields.io/github/license/phineas-pta/NVIDIA-win?logo=creativecommons)
![Platform](https://img.shields.io/badge/Platform-win_64-0078D4?logo=windows)
![Cuda](https://img.shields.io/badge/CUDA-v11.8_%7c_v12.1-76B900?logo=nvidia)
![Cudnn](https://img.shields.io/badge/cuDNN-v8.9-76B900?logo=nvidia)
![Tensorrt](https://img.shields.io/badge/TensorRT-v8.6-76B900?logo=nvidia)
![Python](https://img.shields.io/badge/python-v3.10_%7c_v3.11-3776AB?logo=python)
![VS](https://img.shields.io/badge/Visual_Studio-v17_2022-5C2D91?logo=visualstudio)
![Cmake](https://img.shields.io/badge/CMake-v3.27-064F8C?logo=cmake)

# NVIDIA-win

how to install the NVIDIA’s deep learning stack on Windows: CUDA toolkit + cuDNN + TensorRT,<br />also various attempts to compile deep learning libraries with cuda

only for Windows 10/11 + recent RTX cards recommended

all comands here are for cmd, if u prefer pwsh must change to adapt

some values are censored with black box ███ so u have to fill in with your use case

**1st of all:** 👉👉👉 [install CUDA toolkit + cuDNN + TensorRT](NVIDIA-win.md) 👈👈👈

my attempts to build various deep learning libraries:
1. [build onnxruntime](test1-onnxruntime.md) ✅
2. [build whisper.cpp with cuBLAS](test2-whisper.cpp-cublas.md) ✅
3. [build CTranslate2 to use faster-whisper](test3-ctranslate2.md) ❌
4. [build triton to use xformers](test4-triton.md) ❌
5. [try out TensorRT-LLM](test5-trt-llm.md) ❌
6. [build OpenCV with CUDA](test6-opencv.md) ✅
7. [build Torch-TensorRT](test7-torch-tensorrt.md) ❌

all above guides are under CC-BY-4.0

more complicated tweaking with TensorRT (under GPLv3)
- build Real-ESRGAN with TensorRT on Windows: https://github.com/phineas-pta/RealESRGAN-trt-win ✅
- run Stable Diffusion XL with TensorRT natively on Windows: https://github.com/phineas-pta/SDXL-trt-win ✅
