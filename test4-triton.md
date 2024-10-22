# Test 4: build `triton` to use `xformers` on Windows

**update:** unofficial windows wheels: https://github.com/woct0rdho/triton-windows/releases

prepare at least 5 GiB disk space (4 GiB llvm + 1 GiB triton)

tested version: triton v2.3.0 + llvm v17

## preparation

need Visual Studio console + prepare a fresh python env
```
git clone
	--single-branch
	--branch=release/2.3.x
	--depth=1
	--recurse-submodules
	--shallow-submodules
	https://github.com/openai/triton
```
take note of LLVM commit hash in: file `cmake/llvm-hash.txt` or file `python/setup.py` (line `version = "…"` or `rev = "…"`)

## build LLVM

get full LLVM repo in zip: `https://github.com/llvm/llvm-project/archive/5e5a22caf88ac1ccfa8dc5720295fdeba0ad9372.zip`<br />
change commit hash accordingly<br />
DO NOT DO git clone & checkout coz repo too big, also don’t get `main HEAD` coz possible incompatibility

unzip to a location, rename to avoid long path error (*e.g.* `llvm-project/`)<br />navigate console to folder `llvm-project/`
```
pip install numpy pybind11 pygments pyyaml
cmake -S llvm -B build -G Ninja
	-D CMAKE_BUILD_TYPE=Release
	-D LLVM_ENABLE_WARNINGS=OFF
	-D LLVM_INCLUDE_BENCHMARKS=OFF
	-D LLVM_INCLUDE_EXAMPLES=OFF
	-D LLVM_INCLUDE_TESTS=OFF
	-D LLVM_ENABLE_TERMINFO=OFF
	-D LLVM_TARGETS_TO_BUILD="X86;NVPTX;AMDGPU"
	-D LLVM_ENABLE_PROJECTS=mlir
	-D MLIR_ENABLE_BINDINGS_PYTHON=ON
ninja -C build
```
(amdgpu is required by triton so cannot skip) may take >1h, when finish take note of absolute path to `llvm-project/build`

### attempt to enable `-D MLIR_ENABLE_CUDA_RUNNER=ON`

edit file `mlir/lib/Dialect/GPU/CMakeLists.txt`: replace `${CUDA_DRIVER_LIBRARY}` with `${CUDA_LIBRARY_DIRS}`

edit file `mlir/lib/ExecutionEngine/CMakeLists.txt`: remove `${CUDA_CUSPARSE_LIBRARY}` and replace `${CUDA_RUNTIME_LIBRARY}` with `${CUDA_LIBRARY_DIRS}`

edit file `mlir/lib/ExecutionEngine/CudaRuntimeWrappers.cpp` add block:
```cpp
#include <malloc.h>
#define alloca _alloca
```
FAILED: error `CudaRuntimeWrappers.cpp.obj : error LNK2019: unresolved external symbol`: various object from `cuda.h` cannot be linked

## build triton wheel

navigate console back to folder `triton/` cloned before

edit file `CMakeLists.txt`:
- delete line `set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror -Wno-covered-switch-default")`
- replace `-fPIC -std=gnu++17 -fvisibility=hidden -fvisibility-inlines-hidden` with `/std:c++17 /W0 /MP`
- replace line `set(LLVM_LDFLAGS "-L${…}")` with `set(LLVM_LDFLAGS "/LIBPATH:${…}")`

edit file `lib/Target/HSACO/HSACOTranslation.cpp`: change `kernel_dir.stem()` to `kernel_dir.stem().string()`

edit file `python/setup.py`: change funtion `get_build_type()` to simply return `"Release"` and delete block:
```python
            if sys.maxsize > 2**32:
                cmake_args += ["-A", "x64"]
            build_args += ["--", "/m"]
```
navigate console to folder `python/` then build wheel
```batchfile
SET LLVM_SYSPATH=███/llvm-project/build
SET LLVM_INCLUDE_DIRS=%LLVM_SYSPATH%/include
SET LLVM_LIBRARY_DIR=%LLVM_SYSPATH%/lib

pip wheel . --wheel-dir=dist --verbose
```
if error `fatal error LNK1104: cannot open file 'python3██.lib'` then edit file `CMakeLists.txt` add more `link_directories(${Python3_LIBRARIES})` additionally

if other error, wait for fix see https://github.com/openai/triton/issues/1640#issuecomment-1695521195

wheel file in `dist\triton-2.3.0-cp3██-cp3██-win_amd64.whl`

still cannot use wheel: `No module named 'triton._C.libtriton'`

clean up temp folder `%USERPROFILE%\.triton`
