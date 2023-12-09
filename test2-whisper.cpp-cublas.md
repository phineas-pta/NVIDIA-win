# Test 2: build `whisper.cpp` with cuBLAS on Windows

**update:** executable with cublas included https://github.com/ggerganov/whisper.cpp/releases/latest

no need git clone just get zip https://github.com/ggerganov/whisper.cpp/archive/refs/heads/master.zip

need Visual Studio console
```batchfile
cmake -S . -B build -D WHISPER_CUBLAS=1 -D CMAKE_BUILD_TYPE=Release
msbuild build\ALL_BUILD.vcxproj -noLogo -maxCpuCount -property:Configuration=Release
```
download to folder `models/` any of https://huggingface.co/ggerganov/whisper.cpp/tree/main

normal console (no need Visual Studio nor python)
```batchfile
build\bin\Release\main --threads=█ --print-colors --print-progress --model models\ggml-large.bin --file=samples\jfk.wav
```
additional options: `--language=fr --translate --prompt="initial prompt" --output-srt --output-file=yolo`

only accept 16-bit wav file as input: `ffmpeg -v warning -stats -i input.mp3 -ar 16000 -ac 1 -c:a pcm_s16le output.wav`

if microphone access: need SDL2
download from https://github.com/libsdl-org/SDL/releases

copy file `lib/x64/SDL2.dll` to `build/bin` then ???
