# Test 8: build `llama.cpp` with cuBLAS on Windows

**N.B.:** pre-built binaries: https://github.com/ggerganov/llama.cpp/releases

official docs: https://github.com/ggerganov/llama.cpp/blob/master/docs/build.md

need 600 MiB disk space
```bash
git clone --single-branch --depth=1 --recurse-submodules --shallow-submodules https://github.com/ggerganov/llama.cpp
```
need Visual Studio console
```powershell
cmake -S . -B build -D CMAKE_BUILD_TYPE=Release -D GGML_CUDA=ON -D GGML_CUDA_FA_ALL_QUANTS=ON -D CMAKE_CUDA_ARCHITECTURES=native -D LLAMA_ALL_WARNINGS=OFF
msbuild build\ALL_BUILD.vcxproj -noLogo -maxCpuCount -property:Configuration=Release -verbosity:minimal
# cmake --build build --config Release --parallel --target llama-quantize llama-server
```
normal console (no need Visual Studio nor python)
```powershell
build\bin\Release\llama-server.exe -ngl 99 --no-mmap -c 8192 -ctk q8_0 -ctv q8_0 -fa -m Phi-3.5-mini-instruct-Q8_0.gguf
```
usage:
- https://github.com/ggerganov/llama.cpp/blob/master/examples/main/README.md
- https://github.com/ggerganov/llama.cpp/blob/master/examples/server/README.md
