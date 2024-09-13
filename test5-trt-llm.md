# Test 5: try out TensorRT-LLM on Windows

**update:** this sucks: need MPI, need a lot VRAM, no memory off-loading ⇒ instead: use https://github.com/ggerganov/llama.cpp/releases

📑 in case u need official docs: https://github.com/NVIDIA/TensorRT-LLM/blob/main/windows/README.md

tested version: tensorrt-llm v0.?.?
```
git clone
	--single-branch
	--branch=main
	--depth=1
	--recurse-submodules
	--shallow-submodules
	https://github.com/NVIDIA/TensorRT-LLM
cd TensorRT-LLM
pip install -r requirements-dev-windows.txt
```
download checkpoints:
```python
from huggingface_hub import snapshot_download
LIST_MODS = {
	"gpt2-xl": "examples/gpt/gpt2-xl",
	"microsoft/phi-2": "examples/phi/phi-2",
	"bigscience/bloom-3b": "examples/bloom/bloom-3b",
	# "openai/whisper-large-v3": "examples/whisper/large-v3",  # prefer original checkpoint instead of huggingface
}
for k, v in LIST_MODS.items():
	snapshot_download(
		repo_id=k, # revision = "███", # branch / tag / commit hash
		local_dir=v, local_dir_use_symlinks=False,
		allow_patterns=["*.safetensors", "*.pth"],
		ignore_patterns=[".git*", "*.md", "*.msgpack", "*.h5", "*.ot", "*.bin"],
		resume_download=True, # max_workers=1, # token = "███",
	)
```
test gpt
```
cd examples/gpt

python hf_gpt_convert.py -i gpt2-xl -o gpt2-c-model -tp 1 -t float16
python build.py
	--model_dir=gpt2-c-model/1-gpu
	--dtype=float16
	--use_gpt_attention_plugin
	--remove_input_padding
	--enable_context_fmha
	--use_gemm_plugin
	--output_dir=gpt2-trt
python run.py --max_output_len=20 --engine_dir=gpt2-trt --input_text="hello i am"
```
test bloom
```
cd examples/bloom

python build.py
	--model_dir=bloom-3b
	--dtype=float16
	--use_gemm_plugin
	--use_gpt_attention_plugin
	--output_dir=bloom-trt
```
somehow no engine file eventhough no error
