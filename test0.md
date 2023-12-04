# Test 0: compiling cuda binaries in Visual Studio<br />(just a draft, dont pay attention to this file)

Visual Studio docs: https://learn.microsoft.com/en-us/cpp/build/building-on-the-command-line

initialize cmd with build tools: `"C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat"`

or edit start menu shortcut by adding `-arch=x64 -host_arch=x64`

initialize cmd with mamba: `███\mambaforge\Scripts\activate.bat ███\mambaforge`

initialize pwsh with mamba: `(& "███\mambaforge\Scripts\conda.exe" "shell.powershell" "hook") | Out-String | ?{$_} | Invoke-Expression`

change conda env var: `conda env config vars set -n ███ XDG_CACHE_HOME=███/cache CUDA_MODULE_LOADING=LAZY`

install torch: `pip install torch torchvision torchaudio --find-links=https://download.pytorch.org/whl/torch_stable.html`

dump info:
- `python -m torch.utils.collect_env`
- `nvidia-smi --format=csv --query-gpu=name,compute_cap,driver_version,vbios_version,inforom.image`
- `nvidia-smi --query --display=MEMORY,UTILIZATION,POWER,CLOCK,COMPUTE,PERFORMANCE,VOLTAGE`

quick convert onnx trt `trtexec --onnx=model.onnx --saveEngine=model.trt --best --builderOptimizationLevel=5 --threads`

build pytorch from source: https://pytorch.org/docs/stable/notes/windows.html

## test compile something

download the relevant https://github.com/NVIDIA/cuda-samples/releases

open a  `.sln` file with Visual Studio

select “Project” → “Properties” → “Linker” → “Input” → “Additional Dependencies” → “Edit”

if not included add `.lib` files from `%CUDA_PATH%\lib\x64`

select “Build” → “Build Solution”

## compile torch2trt

need Visual Studio console + prepare a fresh python env
```bash
pip install packaging torch torchvision --find-links https://download.pytorch.org/whl/torch_stable.html
pip install git+https://github.com/NVIDIA-AI-IOT/torch2trt.git
```

## compile `xformers`

need Visual Studio console + prepare a fresh python env
```batchfile
pip install ninja torch --find-links=https://download.pytorch.org/whl/torch_stable.html

SET TORCH_CUDA_ARCH_LIST=█.█
SET XFORMERS_BUILD_TYPE=Release
SET DISTUTILS_USE_SDK=1
```
try
```bash
pip install git+https://github.com/facebookresearch/xformers.git@main#egg=xformers
```
if above not works (error file path too long) then compile manually
```
git clone
	--single-branch
	--depth=1
	--recurse-submodules
	--shallow-submodules
	https://github.com/facebookresearch/xformers
cd xformers

pip wheel . --wheel-dir=dist --verbose --find-links=https://download.pytorch.org/whl/torch_stable.html
```
after install
```bash
python -m xformers.info
```

## compile jax

no need git clone just get zip from https://github.com/google/jax/archive/refs/heads/main.zip

need Visual Studio console + prepare a fresh python env
```
pip install numpy build
python build/build.py
	--bazel_path="<path to>/bazel.exe"
	--enable_cuda
	--cuda_path="C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.1"
	--cudnn_path="C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.1"
	--cuda_version="12.1"
	--cudnn_version="8.9.6"
	--cuda_compute_capabilities="sm_██"
```
cannot use `%CUDA_PATH%` coz it failed to parse windows path

FAILED: problem with bazel build, no support for cuda windows
