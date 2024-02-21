# Test 0: compiling cuda binaries in Visual Studio<br />(just a draft, dont pay attention to this file)

Visual Studio docs: https://learn.microsoft.com/en-us/cpp/build/building-on-the-command-line

initialize cmd with build tools: `"C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat"`

initialize pwsh with build tools: `&{Import-Module 'C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\Tools\Microsoft.VisualStudio.DevShell.dll'; Enter-VsDevShell -VsInstallPath 'C:\Program Files\Microsoft Visual Studio\2022\Community' -DevCmdArguments '-arch=x64'}`

or edit start menu shortcut by adding `-arch=x64 -host_arch=x64`

initialize cmd with mamba: `███\mambaforge\Scripts\activate.bat ███\mambaforge`

initialize pwsh with mamba: `(& '███\mambaforge\Scripts\conda.exe' 'shell.powershell' 'hook') | Out-String | ?{$_} | Invoke-Expression`

change conda env var: `conda env config vars set -n ███ XDG_CACHE_HOME=███/cache`

on windows, cuda < 12.3, should add `CUDA_MODULE_LOADING=LAZY`

install torch: `pip install torch torchvision torchaudio --find-links=https://download.pytorch.org/whl/torch_stable.html`

install onnx: `pip install onnxruntime-gpu --extra-index-url=https://aiinfra.pkgs.visualstudio.com/PublicPackages/_packaging/onnxruntime-cuda-12/pypi/simple/`

quick convert onnx trt `trtexec --threads --best --builderOptimizationLevel=5 --onnx=model.onnx --saveEngine=model.trt`

build pytorch from source: https://pytorch.org/docs/stable/notes/windows.html

## dump info:

- `python -m torch.utils.collect_env`
- `nvidia-smi --format=csv --query-gpu=name,compute_cap,driver_version,vbios_version,inforom.image`
- `nvidia-smi --query --display=MEMORY,UTILIZATION,POWER,CLOCK,COMPUTE,PERFORMANCE,VOLTAGE`

```python
import subprocess as sp
print(sp.Popen(["nvidia-smi"], stdout=sp.PIPE, stderr=sp.PIPE).communicate()[0].split(b" "*48)[0].decode("utf-8"))
```
official python interface: `pip install nvidia-ml-py`
```python
import pynvml
pynvml.nvmlInit()
print("Driver Version:", pynvml.nvmlSystemGetDriverVersion())
GB_NUM = 1024**3
for i in range(pynvml.nvmlDeviceGetCount()):
	handle = pynvml.nvmlDeviceGetHandleByIndex(i)
	info = pynvml.nvmlDeviceGetMemoryInfo(handle)
	print(f"Device {i}:", pynvml.nvmlDeviceGetName(handle))
	print(f"\tTotal memory: {info.total / GB_NUM :.1f} GiB")
	print( f"\tFree memory: {info.free  / GB_NUM :.1f} GiB")
	print( f"\tUsed memory: {info.used  / GB_NUM :.1f} GiB")
pynvml.nvmlShutdown()
```

## test compile something

download the relevant https://github.com/NVIDIA/cuda-samples/releases

open a  `.sln` file with Visual Studio

select “Project” → “Properties” → “Linker” → “Input” → “Additional Dependencies” → “Edit”

if not included add `.lib` files from `%CUDA_PATH%\lib\x64`

select “Build” → “Build Solution”

## build `torch2trt`

need Visual Studio console + prepare a fresh python env
```bash
pip install packaging torch torchvision --find-links=https://download.pytorch.org/whl/torch_stable.html
pip install git+https://github.com/NVIDIA-AI-IOT/torch2trt.git
```

## build `bitsandbytes`

see https://huggingface.co/docs/bitsandbytes/main/en/installation?OS+system=Windows

get https://github.com/TimDettmers/bitsandbytes/archive/refs/heads/main.zip
```bash
pip install -r requirements-dev.txt
cmake -S . -D COMPUTE_BACKEND=cuda -D COMPUTE_CAPABILITY=██
cmake --build . --config Release --parallel
pip wheel . --wheel-dir="dist" --verbose --find-links=https://download.pytorch.org/whl/torch_stable.html
```
verify after install: `python -m bitsandbytes`

## build `xformers`

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
verify after install: `python -m xformers.info`

## build `jax`

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
