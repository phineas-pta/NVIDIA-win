# Test 3: build cTranslate2 to use `faster-whisper`

📑 in case u need official docs: https://github.com/OpenNMT/CTranslate2/blob/master/docs/installation.md

```
git clone
	--single-branch
	--depth=1
	--recurse-submodules
	--shallow-submodules
	https://github.com/OpenNMT/CTranslate2
```
if intel cpu:
- download intel toolkit: https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit-download.html
- update Visual Studio beforehand coz shitty installer try to update Visual Studio and mess up !!! (both online & offline installer)
- select at least “c++ compiler” + “math kernel library” (need at least 12gb additionally)
- or `pip install dpcpp-cpp-rt mkl-devel mkl-static mkl-include`

if not intel cpu: add `-D OPENMP_RUNTIME=NONE -D WITH_MKL=OFF` to command below

need Visual Studio console + prepare a fresh python env with `pip install pybind11`
```batchfile
cmake -S . -B build -D WITH_CUDA=ON -D WITH_CUDNN=ON -D CUDA_DYNAMIC_LOADING=ON -D CUDA_NVCC_FLAGS="--threads=0" -D CUDA_ARCH_LIST=█.█
msbuild build\ALL_BUILD.vcxproj -noLogo -maxCpuCount -property:Configuration=Release
```

then navigate console to folder `python/`

edit file `setup.py` line 11-12:
```python
include_dirs = [pybind11.get_include(), "../include"]
library_dirs = ["../build/Release"]
```
then run `pip wheel . --wheel-dir="dist" --verbose`

wheel file in `dist\ctranslate2-█.██.█-cp3██-cp3██-win_amd64.whl`

ERROR: `module 'ctranslate2' has no attribute 'StorageView'`
