# Test 0: compiling cuda binaries<br />(just a draft, dont pay attention to this file)

Visual Studio docs: https://learn.microsoft.com/en-us/cpp/build/building-on-the-command-line

## dump info:

- `python -m torch.utils.collect_env`
- `nvidia-smi --format=csv --query-gpu=name,compute_cap,driver_version,vbios_version,inforom.image`
- `nvidia-smi --query --display=MEMORY,UTILIZATION,POWER,CLOCK,COMPUTE,PERFORMANCE,VOLTAGE`

```python
import subprocess as sp
print(sp.Popen(["nvidia-smi"], stdout=sp.PIPE, stderr=sp.PIPE).communicate()[0].split(b" "*48)[0].decode("utf-8"))
```
official python interface: `uv pip install nvidia-ml-py`
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

## build `jax`

no need git clone just get zip from https://github.com/jax-ml/jax/archive/refs/heads/main.zip

need Visual Studio console + prepare a fresh python env
```
pip install numpy build
python build/build.py
	--bazel_path="<path to>/bazel.exe"
	--bazel_options=--repo_env=LOCAL_CUDA_PATH="C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v13.2"
	--bazel_options=--repo_env=LOCAL_CUDNN_PATH="C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v13.2"
	--enable_cuda
	--cuda_compute_capabilities="sm_██"
```
cannot use `%CUDA_PATH%` coz it failed to parse windows path

FAILED: problem with bazel build, no support for cuda windows

linux-only: `pip install 'jax[cuda13]'`
