# Test 6: build OpenCV with CUDA on Windows

prepare at least 10gb disk space

tested version: opencv v4.8
```
git clone
	--single-branch
	--branch=4.x
	--depth=1
	--recurse-submodules
	--shallow-submodules
	https://github.com/opencv/opencv-python
```
edit file `pyproject.toml`: replace all lines `numpy` with only 1 line `numpy`

need Visual Studio console + prepare a fresh python env with `pip install numpy`
```batchfile
SET CFLAGS=/MP
SET CXXFLAGS=/MP
SET CUDA_NVCC_FLAGS=--threads=0
SET ENABLE_CONTRIB=1
SET ENABLE_ROLLING=1
SET CMAKE_ARGS=-D CMAKE_BUILD_TYPE=Release -D OPENCV_ENABLE_NONFREE=ON -D BUILD_EXAMPLES=OFF -D WITH_GSTREAMER=OFF -D WITH_CUDA=ON -D OPENCV_DNN_CUDA=ON -D CUDA_ARCH_BIN=█.█ -D CUDA_ARCH_PTX=█.█
```
*N.B.*:
- in this case do not surround env var with quotes
- in `setup.py` already enforce many flags
- (intel cpu) enable `-D WITH_OPENMP=ON -D WITH_IPP=ON -D WITH_TBB=ON` (together or separatedly) make wheel unusable with error `ImportError: DLL load failed while importing cv2: The specified module could not be found.`

then build with `pip wheel . --wheel-dir=dist --verbose` (may take >1h, much longer if multiple cuda arch)

if re-build then delete folder `_skbuild/` at each time

if error `Compiler doesn't support baseline optimization flags` then add `-D CV_DISABLE_OPTIMIZATION=ON -D CPU_BASELINE="" -D CPU_DISPATCH=""`

wheel file in `dist\opencv_contrib_python_rolling-4.8.███-cp3██-cp3██-win_amd64.whl`

test python code
```python
import cv2
print(cv2.getBuildInformation())
print(cv2.cuda.getCudaEnabledDeviceCount())

vod = cv2.VideoCapture("test.mp4")
frame_device = cv2.cuda_GpuMat()
ret, frame = vod.read()
while ret:
	frame_device.upload(frame)
	ret, frame = vod.read()
```
