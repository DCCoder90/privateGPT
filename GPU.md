From [author](https://github.com/imartinez/privateGPT/pull/521#issuecomment-1586093625):

We must set the flag to turn on / of GPU acceleration. With the help of @DanielusG I manage to run it on my Windows PC, didn't test it on Linux and MacOS but I gather some experiences I can list here. Note this worked for llama-cpp-python==0.1.61

# GPU acceleration
Most importantly the GPU_IS_ENABLED variable must be set to true. Add this to HuggingfaceEmbeddings:

```
embeddings_kwargs = {'device': 'cuda'} if gpu_is_enabled else {}
```
## GPU (Windows)
Set OS_RUNNING_ENVIRONMENT=windows inside .env file
```
pip3 install -r requirements_windows.txt
```
Install Visual Studio 2019 - 2022 Code C++ compiler on Windows 10/11:

1. Install Visual Studio.
2. Make sure the following components are selected:
   * Universal Windows Platform development
   * C++ ```CMak``` tools for Windows
3. Download the ```MinGW``` installer from the [MinGW website](https://sourceforge.net/projects/mingw/).
4. Run the installer and select the ```gcc``` component.

You can use the included installer batch file to install the required dependencies for GPU acceleration, or:

1. Find your card driver here [NVIDIA Driver Downloads](https://www.nvidia.com/download/index.aspx)

2. Install [NVidia CUDA 11.8](https://developer.nvidia.com/cuda-11-8-0-download-archive?target_os=Windows&target_arch=x86_64)

3. Install ```llama-cpp-python``` package with ```cuBLAS``` enabled. Run the code below in the directory you want to build the package in.

Powershell:
```powershell
$Env:CMAKE_ARGS="-DLLAMA_CUBLAS=on"; $Env:FORCE_CMAKE=1; pip3 install llama-cpp-python --force-reinstall --upgrade --no-cache-dir
```
Bash:
```bash
CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip3 install llama-cpp-python --force-reinstall --upgrade --no-cache-dir
```
4. Enable GPU acceleration in ```.env``` file by setting ```GPU_IS_ENABLED``` to ```true```

5. Run ```ingest.py``` and ```privateGPT.py``` as usual

If the above doesn't work for you, you will have to manually build llama-cpp-python library with CMake:

1. Get repo ```git clone https://github.com/abetlen/llama-cpp-python.git```
    * switch to tag this application is using from ```requirements-*.txt``` file:
    * uninstall your local llama-cpp-python: ```pip3 uninstall llama-cpp-python```
2. Open ```llama-cpp-python/vendor/llama.cpp/CMakeList.txt``` in text editor and add
```set(LLAMA_CUBLAS 1)``` to the line ```178``` before ```if (LLAMA_CUBLAS)``` line.
3. Install [CMake](https://cmake.org/download/)
4. Go to ```cd llama-cpp-python``` and perform the actions:
    * perform ```git submodule update --init --recursive```
    * ```mkdir build``` and ```cd build```
5. Build llama-cpp-python yourself:
```
cmake -G "Visual Studio 16 2019" -A x64 -D CUDAToolkit_ROOT="C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v11.8" ..
``` 
6. Position CLI to this project and install llama from the folder you build, let's say ```pip3 install ../llama-cpp-python/```

Next is somewhat @DanielusG already tested on Linux I guess:

## GPU (Linux):
Set ```OS_RUNNING_ENVIRONMENT=linux``` inside ```.env``` file

If you have an Nvidia GPU, you can speed things up by installing the ```llama-cpp-python``` version with CUDA
by setting these flags: ```export LLAMA_CUBLAS=1```

(some libraries might be different per OS, that's why I separated requirements files)
```
pip3 install -r requirements_linux.txt
```
First, you have to uninstall the old torch installation and install CUDA one:
Install a proper torch version:
```
pip3 uninstall torch
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```
Now, set environment variables and source them:
```
vim ~/.bashrc
export LLAMA_CUBLAS=1
export LLAMA_CLBLAST=1 
export CMAKE_ARGS=-DLLAMA_CUBLAS=on
export FORCE_CMAKE=1
source ~/.bashrc
```
## LLAMA
llama.cpp doesn't work easily on Windows with GPU,, so you should try with a Linux distro
Installing torch with CUDA will only speed up the vector search, not the writing by llama.cpp.

You should install the latest Cuda toolkit:

```
conda install -c conda-forge cudatoolkitpip uninstall llama-cpp-python
```
if you're already in conda env you can uninstall llama-cpp-python like this:
```
pip3 uninstall llama-cpp-python
```
Install llama:
```
CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip install --upgrade --force-reinstall llama-cpp-python==0.1.61 --no-cache-dir
```
Modify LLM code to accept ```n_gpu_layers```:
```
llm = LlamaCpp(model_path=model_path, ..., n_gpu_layers=20)
```
Change environment variable model:
```
MODEL_TYPE=llamacpp
MODEL_ID_OR_PATH=models/ggml-vic13b-q5_1.bin
```