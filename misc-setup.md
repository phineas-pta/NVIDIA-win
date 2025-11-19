# miscellaneous setup

## visual studio

⏬ **download**: https://visualstudio.microsoft.com/visual-cpp-build-tools/

initialize cmd with build tools: `"C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat"`

initialize pwsh with build tools: `&{Import-Module 'C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\Tools\Microsoft.VisualStudio.DevShell.dll'; Enter-VsDevShell -VsInstallPath 'C:\Program Files\Microsoft Visual Studio\2022\Community' -DevCmdArguments '-arch=x64'}`

or edit start menu shortcut by adding `-arch=x64 -host_arch=x64`

## conda / mamba

⏬ **download**: https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Windows-x86_64.exe

initialize cmd with mamba: `███\miniforge3\Scripts\activate.bat ███\miniforge3`

initialize pwsh with mamba: `(& '███\miniforge3\Scripts\conda.exe' 'shell.powershell' 'hook') | Out-String | ?{$_} | Invoke-Expression`

change conda env var: `conda env config vars set -n ███ XDG_CACHE_HOME=███/cache HF_HOME=███/cache PIP_CACHE_DIR=███/cache`

on windows, cuda < 12.3, should add `CUDA_MODULE_LOADING=LAZY`

## some basic python package

`pip install jupyterlab ipywidgets scikit-learn-intelex seaborn sympy tqdm tensorboard cupy-cuda12x`

install deep learning frameworks:
- torch: see https://pytorch.org/get-started/locally/
- onnx: see https://onnxruntime.ai/getting-started
- paddlepaddle: see https://www.paddlepaddle.org.cn/documentation/docs/en/install/pip/windows-pip_en.html

install hf ecosystem: `pip install transformers diffusers accelerate "datasets[audio,vision]" tokenizers peft bitsandbytes "optimum[onnxruntime-gpu]" sentence_transformers timm`

intel distribution for python: `mamba install intelpython3_full -c https://software.repos.intel.com/python/conda/`

## rtools

install Rtools: https://cran.r-project.org/bin/windows/Rtools/ (installation path should not contain whitespace)

initialize cmd with rtools: `C:/rtools██/msys2_shell.cmd -defterm -here -no-start -ucrt64`

*N.B.*: use UCRT64 as per https://stackoverflow.com/a/76552265/10805680

install additional build tools:
```bash
pacman -Suy
pacman -Sy "${MINGW_PACKAGE_PREFIX}-make" "${MINGW_PACKAGE_PREFIX}-gcc"
pacman -Scc
```

more lightweight alternative: https://github.com/skeeto/w64devkit/releases

## other

Julia:
```julia
import Pkg; Pkg.add("CUDA")
using CUDA
CUDA.set_runtime_version!(local_toolkit=true)
# then restart Julia
CUDA.versioninfo() # run this to save settings

N = 2^20
x = CUDA.fill(1.0f0, N);
y = CUDA.fill(2.0f0, N);
z = x .+ y;
```

quick convert onnx trt `trtexec --threads --best --builderOptimizationLevel=5 --onnx=model.onnx --saveEngine=model.trt`

build pytorch from source: https://pytorch.org/docs/stable/notes/windows.html
