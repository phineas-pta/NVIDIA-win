# install NVIDIA’s deep learning stack on Windows: <br />CUDA toolkit + cuDNN + TensorRT

✍️ inspired from https://david-littlefield.medium.com/how-to-install-the-nvidia-cuda-driver-toolkit-cudnn-and-tensorrt-on-windows-10-3fcf97e54522

**i wrote this guide coz nvidia docs suck**

📑 in case u still somehow need official docs:
- CUDA: https://docs.nvidia.com/cuda/cuda-installation-guide-microsoft-windows/index.html
- cuDNN: https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html
- TensorRT: https://docs.nvidia.com/deeplearning/tensorrt/install-guide/index.html

📑 in case u need info about cuda compute capability:
- https://developer.nvidia.com/cuda-gpus
- https://docs.nvidia.com/deeplearning/tensorrt/support-matrix/index.html#hardware-precision-matrix
- https://developer.nvidia.com/video-encode-and-decode-gpu-support-matrix-new

prepare at least 15gb disk space (10gb msvc + 5gb nvidia)

tested combination: Visual Studio v17 (2022) + CUDA v12.1 + cuDNN v8.9 + TensorRT v8.6

## 🔖 easy 1st steps with graphical interface 📱

### 0️⃣ NVIDIA driver

⏬ **download**: https://www.nvidia.com/download/index.aspx

alternatively install companion software:
- https://www.nvidia.com/en-us/geforce/geforce-experience/
- https://www.nvidia.com/en-us/design-visualization/software/rtx-experience/

each cuda version requires a minimum driver version, see: https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html

### 1️⃣ C++ tools set (CMake, ninja, cl, MSBuild, vcpkg)

if need compile binaries with cuda

⏬ **download** either:
- https://visualstudio.microsoft.com/visual-cpp-build-tools/ (more lightweight)
- https://visualstudio.microsoft.com/downloads/ (better integration with cuda)

⚠️ Visual Studio (VS) not visual studio code (vscode), the latter has nothing to do in all my guide

👉 when install, select checkbox “Desktop Development with C++”

🔎 verify after install: in Start menu, there’re 2 new items “Developer Command Prompt for VS 2022” & “Developer PowerShell for VS 2022”

if u open any of those 2 and run any of these following commands, it should return messages instead of errors
- `cmake --version` returns `3.27.█-msvc1`
- `cl` returns `19.38.███`
- `msbuild -ver` returns `17.8.█`
- `ninja --version` returns `1.11.█`

see where to find header files: `echo %INCLUDE%` (cmd) or `echo $env:INCLUDE` (pwsh)

### 2️⃣ CUDA toolkit

⏬ **download**:
- https://developer.nvidia.com/cuda-downloads (directly latest version)
- https://developer.nvidia.com/cuda-toolkit-archive (all available versions)

👉 when install, select “Advanced” → select at least “Development” + “Runtime”, and if Visual Studio installed “VS integration”

🔎 verify after install: open System Properties > tab Advanced > Environment Variables > system
- should have `%CUDA_PATH%` and `%CUDA_PATH_V12_1%` set to `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1`
- `%PATH%` should contain `%CUDA_PATH%\bin` &  `%CUDA_PATH%\libnvvp` but not other things from `%CUDA_PATH%`
- optional: if not exist, set `%CUDA_HOME%` & `%CUDA_ROOT%` same as `%CUDA_PATH%`
- optional: if exist `%LD_LIBRARY_PATH%` add `%CUDA_PATH%\lib\x64`

🔎 also verify if `nvcc --version` return correct cuda version

*N.B.* `nvidia-smi` show max cuda version of driver, even if cuda not installed

## 🔖 convoluted steps without installer 🗿

⚠️ need NVIDIA acc + join Developer Program

⚠️ open an **admin cmd console** to use throughout following steps

if u prefer pwsh instead, in following steps replace `COPY` with `Copy-Item` and `%CUDA_PATH%` with `$env:CUDA_PATH`

*N.B.* native Windows consoles are case-insensitive (msys2/cygwin is not native)

⚠️ if multiple cuda versions co-exist, in following steps replace `CUDA_PATH` with the corresponding `CUDA_PATH_V1█_█`

*N.B.* call `CUDA_PATH` in double quotes coz path contain whitespaces

📑 core **principle** of following steps:
| files | copy to |
| --- | --- |
| `.dll` | `%CUDA_PATH%\bin` |
| `.lib` | `%CUDA_PATH%\lib` |
| `.h` | `%CUDA_PATH%\include` |

### 3️⃣ cuDNN

⏬ **download** either:
- https://developer.nvidia.com/rdp/cudnn-download (latest version)
- https://developer.nvidia.com/rdp/cudnn-archive (older version)

👉 extract zip to a location, navigate console to the extracted folder (in some old cudnn version need to descend into `.\cuda`) then run
```batchfile
COPY bin\cudnn*.dll     "%CUDA_PATH%\bin"
COPY lib\x64\cudnn*.lib "%CUDA_PATH%\lib\x64"
COPY include\cudnn*.h   "%CUDA_PATH%\include"
```

#### 🐍 boost PyTorch with cudnn fix

normal pytorch usually not shipped with most recent cudnn, so users can replace pytorch linked `.dll` files to get more boost from recent update

get pytorch lib directory: `python -c "import torch as _; print(_.__path__[0] + r'\lib')"`

copy 7 cudnn `.dll` files to pytorch lib directory

#### 🚩 zlib

*N.B.* this library is in official docs but poorly explained, should be improved in future version of cudnn

⏬ **download**: https://www.winimage.com/zLibDll/zlib123dllx64.zip

👉 extract zip to a location, navigate console to in the extracted folder then run
```batchfile
COPY dll_x64\zlibwapi.dll "%CUDA_PATH%\bin"
COPY dll_x64\zlibwapi.lib "%CUDA_PATH%\lib\x64"
```
*N.B.* official code to build from source: https://zlib.net/ (must rename as `zlibwapi`)

### 4️⃣ TensorRT

⏬ **download**: https://developer.nvidia.com/tensorrt-download

if latest version has multiple choices: select “GA” (general availability) not “EA” (early access) nor “RC” (release candidate)

👉 extract zip to a location, navigate console to in the extracted folder then run
```batchfile
COPY bin\trtexec.exe "%CUDA_PATH%\bin"
COPY lib\nv*.dll     "%CUDA_PATH%\bin"
COPY lib\nv*.lib     "%CUDA_PATH%\lib\x64"
COPY include\Nv*.h   "%CUDA_PATH%\include"
```

#### 🐍 python packages for tensorrt (optional)

only if u need those python pkgs

no need admin console, remember to activate if u use conda/venv
```batchfile
pip install graphsurgeon\graphsurgeon-….whl
pip install uff\uff-….whl
pip install onnx_graphsurgeon\onnx_graphsurgeon-….whl
pip install python\tensorrt-….whl
```
alternatively `pip install onnx-graphsurgeon polygraphy uff --extra-index-url=https://pypi.ngc.nvidia.com`<br />(or `https://developer.download.nvidia.com/compute/redist`)

*N.B.* `pip install tensorrt` only available on Linux; for Windows need version ≥9

# other possible libraries with cuda

see full list
- https://developer.nvidia.com/gpu-accelerated-libraries
- https://developer.nvidia.com/accelerated-computing-toolkit

for e.g.
- https://developer.nvidia.com/cutensor-downloads
- http://developer.nvidia.com/cudss-downloads
- https://developer.nvidia.com/nvidia-video-codec-sdk/download
- https://developer.nvidia.com/nvjpeg2000-downloads

cuda header files: https://gitlab.com/nvidia/headers

*appendix*: other hardware toolkits:
- https://www.intel.com/content/www/us/en/developer/tools/oneapi/toolkits.html
- https://www.amd.com/en/developer/resources/rocm-hub/hip-sdk.html
