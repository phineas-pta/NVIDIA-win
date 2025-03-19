# Test 7: build Torch-TensorRT on Windows

> [!IMPORTANT]
> **update:** official windows wheels strating from v2.3: `pip install torch_tensorrt`

---

📑 in case u need official docs:
- https://pytorch.org/TensorRT/getting_started/installation.html (full setup on linux)
- https://pytorch.org/TensorRT/getting_started/getting_started_with_windows.html (no python wheel)
- https://github.com/pytorch/TensorRT/issues/2371 (help to compile with bazel)
- https://github.com/pytorch/TensorRT/issues/856 (help to build python wheel)
- https://github.com/pytorch/TensorRT/compare/main...yuriishutkin:Torch-TensorRT:windows

prepare at least 5 GiB disk space (4 GiB libtorch + 1 GiB torch_tensorrt)

tested version: torch-tensorrt v2.2

## preparation
```bash
git clone --single-branch --branch=release/2.2 --depth=1 https://github.com/pytorch/TensorRT
```
get bazel from either: (rename downloaded file to `bazel.exe`)
- https://github.com/bazelbuild/bazel/releases
- https://github.com/bazelbuild/bazelisk/releases (bug failed to auto detect Visual Studio)

get LibTorch for C++ on Windows from https://pytorch.org/get-started/locally/ then extract to a location

edit file `WORKSPACE`:
- remove `local_repository("torch_tensorrt", …)` and `new_local_repository("cuda", …)`
- remove any `http_archive(…)` containing `"libtorch"`, `"libtorch_pre_cxx11_abi"`, `"cudnn"`, `"tensorrt"`
- add following lines:
```starlark
new_local_repository(
    name = "libtorch",
    build_file = "@//third_party/libtorch:BUILD",
    path = "<path to>/libtorch",
)

new_local_repository(
    name = "libtorch_pre_cxx11_abi",
    build_file = "@//third_party/libtorch:BUILD",
    path = "<path to>/libtorch",
)

new_local_repository(
    name = "cuda",
    build_file = "@//third_party/cuda:BUILD",
    path = "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.8",
)

new_local_repository(
    name = "cublas",
    build_file = "@//third_party/cublas:BUILD",
    path = "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.8",
)

new_local_repository(
    name = "cudnn",
    build_file = "@//third_party/cudnn/local:BUILD",
    path = "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.8",
)

new_local_repository(
    name = "tensorrt",
    build_file = "@//third_party/tensorrt/local:BUILD",
    path = "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.8",
)
```
edit file `.bazelrc`: change `-fdiagnostics-color=always` to `/diagnostics:caret` and `-std=c++17` to `/std:c++17`

edit file `core/conversion/converters/impl/activation.cpp`: change `if (slopes.ndimension() == 1 and …)` to `if (slopes.ndimension() == 1 && …)`

edit file `core/runtime/TRTEngine.h`: add new line after 1st line: `#define _SILENCE_EXPERIMENTAL_FILESYSTEM_DEPRECATION_WARNING`

edit file `cpp/lib/BUILD`: change block
```starlark
cc_binary(
    name = "torch_tensorrt.dll",
    srcs = [],
    linkshared = True,
    linkstatic = True,
    deps = [
        "//cpp:torch_tensorrt",
    ],
)
```
to
```starlark
cc_binary(
    name = "torch_tensorrt.dll",
    srcs = [],
    linkshared = True,
    linkstatic = True,
    deps = [
        "//cpp:torch_tensorrt",
        "//core/runtime:runtime",
        "//core/plugins:torch_tensorrt_plugins",
    ],
)
```
edit file `cpp/include/torch_tensorrt/macros.h`: change line containing 2nd occurence of `#define TORCHTRT_API` to `#define TORCHTRT_API __declspec(dllexport)`

edit file `third_party/cudnn/local/BUILD`: change line `cudnn64_7.dll` to `cudnn64_8.dll`

edit file `third_party/tensorrt/local/BUILD`:
- change line `":windows": ["lib/nvinfer_plugin.dll"],` to `":windows": ["lib/x64/nvinfer_plugin.lib", "bin/nvinfer_plugin.dll"],`
- change any line `":windows": "lib/nv███.dll",` to `":windows": "bin/nv███.dll",`
- change any line `":windows": "lib/nv███.lib",` to `":windows": "lib/x64/nv███.lib",`
- also change block
```starlark
    deps = [
        "nvinfer",
        "@cuda//:cudart",
        "@cudnn",
    ],
    alwayslink = True,
```
to
```
    deps = [
        "nvinfer",
        "@cuda//:cudart",
        "@cudnn",
    ] + select({
        ":windows": ["@cuda//:cublas", "nvinfer_static_lib"],
        "//conditions:default": ["@cuda//:cublas"],
    }),
    alwayslink = True,
```
**need Visual Studio console + prepare a fresh python env**

## if only need compile `torch_tensorrt.dll`

compile with `bazel build //:libtorchtrt --compilation_mode=opt`

if bug bazel cannot detect Visual Studio see https://github.com/bazelbuild/bazel/pull/18608#issuecomment-1598294872 to download https://github.com/redsun82/bazel/tree/fix-vs-autodetection/tools/cpp

bazel output folder `bazel-out/` is a symlink from `%USERPROFILE%\_bazel_███\███\execroot\Torch-TensorRT\bazel-out`

*N.B.* save folder `bazel-out/x64_windows-opt/bin/` elsewhere before clear cache with `bazel clean --expunge`

## if need python wheel

rename file `BUILD` to `BUILD.bazel`

edit file `setup.py`:
- change line `BAZEL_EXE = which("bazelisk")` to path to bazel bin downloaded
- remove block `if … os.mknod(…)`
- inside function `build_libtorchtrt_pre_cxx11_abi(…)` add `cmd.append("--features=windows_export_all_symbols")` properly
- change line `libraries=["torchtrt"],` to `libraries=["torch_tensorrt.dll.if"],`
- change block `extra_compile_args=[…] + […]` to `extra_compile_args=["/DPYBIND11_EXPORT=", "/DNO_EXPORT", f"-D_GLIBCXX_USE_CXX11_ABI={int(CXX11_ABI)}"]`
- change block `extra_link_args=[…] + […]` to `extra_link_args=["/OPT:NOREF", f"-D_GLIBCXX_USE_CXX11_ABI={int(CXX11_ABI)}"]`

install python packages in file `pyproject.toml` then remove `torch` & `tensorrt` from file
```batchfile
SET DISTUTILS_USE_SDK=1
pip wheel . --wheel-dir=dist --verbose --no-build-isolation
```
**bazel build successfully but failed to create python wheel (weird linker error)**
