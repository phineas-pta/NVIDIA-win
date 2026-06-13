# Test 8: build `llama.cpp` with cuBLAS on Windows

> [!TIP]
> pre-built binaries: https://github.com/ggml-org/llama.cpp/releases/latest

official docs: https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md

need 600 MiB disk space
```bash
git clone --single-branch --depth=1 --recurse-submodules --shallow-submodules https://github.com/ggml-org/llama.cpp
```

need Visual Studio console
```powershell
cmake -S . -B build -D CMAKE_BUILD_TYPE=Release -D GGML_CUDA=ON -D GGML_CUDA_FA_ALL_QUANTS=ON
msbuild build\ALL_BUILD.vcxproj -noLogo -maxCpuCount -property:Configuration=Release -verbosity:minimal
# cmake --build build --config Release --parallel --target llama-quantize llama-server
```

usage: ttps://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md

test model:
- https://huggingface.co/google/gemma-4-E4B-it-qat-q4_0-gguf/blob/main/gemma-4-E4B_q4_0-it.gguf
- https://huggingface.co/google/gemma-4-E4B-it-qat-q4_0-gguf/blob/main/gemma-4-E4B-it-mmproj.gguf

```
build\bin\Release\llama-server.exe
	--model=gemma-4-E4B_q4_0-it.gguf
	--mmproj=gemma-4-E4B-it-mmproj.gguf
	--n-gpu-layers=all
	--ctx-size=8192
	--no-mmap
	--flash-attn
	--cache-type-k=q8_0
	--cache-type-k=q8_0
	--temperature=1.0
	--top-p=0.95
	--top-k=64
	--tools=all
```

interface at `localhost:8080`

```python
import openai
client = openai.OpenAI(base_url="http://localhost:8080/v1", api_key="EMPTY")

response = client.responses.create(
	model="gemma-4-E4B_q4_0-it",
	instructions="You are ChatGPT, an AI assistant. Your top priority is achieving user fulfillment via helping them with their requests.",
	input="Write a limerick about python exceptions"
)
print(response.output_text)

completion = client.chat.completions.create(model="gemma-4-E4B_q4_0-it", messages=[
	{"role": "system", "content": "You are ChatGPT, an AI assistant. Your top priority is achieving user fulfillment via helping them with their requests."},
	{"role": "user", "content": "Write a limerick about python exceptions"}
])
print(completion.choices[0].message)
```
