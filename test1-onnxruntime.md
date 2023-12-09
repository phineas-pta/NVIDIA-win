# Test 1: build onnxruntime with cuda + cudnn + tensorrt on windows

📑 in case u need official docs:
- https://onnxruntime.ai/docs/build/inferencing.html
- https://github.com/microsoft/onnxruntime/blob/main/tools/ci_build/build.py

prepare at least 3gb disk space

tested version: onnxruntime v1.16.3
```
git clone
	--single-branch
	--branch=rel-1.16.3
	--depth=1
	--recurse-submodules
	--shallow-submodules
	https://github.com/microsoft/onnxruntime
```
need Visual Studio console + prepare a fresh python env

list of cmake generators: see `cmake --help`
```
python tools/ci_build/build.py
	--build_dir=build
	--config=Release
	--build
	--update
	--parallel
	--skip_tests
	--enable_pybind
	--build_wheel
	--compile_no_warning_as_error
	--skip_submodule_sync
	--use_cuda
	--use_tensorrt
	--cuda_home="%CUDA_PATH%"
	--cudnn_home="%CUDA_PATH%"
	--tensorrt_home="%CUDA_PATH%"
	--cmake_generator="Visual Studio 17 2022"
	--numpy_version="1.██.█"
	--cmake_extra_defines="CMAKE_CUDA_ARCHITECTURES=██"
```
may take >1h

wheel file in `build\Release\Release\dist\onnxruntime_gpu-1.16.3-cp3██-cp3██-win_amd64.whl`

test python code
```python
import onnxruntime as ort
print(ort.get_available_providers())

MY_PROVIDERS = [("TensorrtExecutionProvider", {
	"trt_fp16_enable": True,
	"trt_int8_enable": True,
	"trt_engine_cache_path": "cache",
	"trt_engine_cache_enable": True,
	"trt_timing_cache_enable": True,
	"trt_builder_optimization_level": 5,
	"trt_build_heuristics_enable": True,
})]
# [("CUDAExecutionProvider", {"cudnn_conv_algo_search": "HEURISTIC"})]

# DeepFaceLab *.dfm files are actually *.onnx: get at https://github.com/iperov/DeepFaceLive/releases
session = ort.InferenceSession("model.onnx", providers=MY_PROVIDERS)
```
