# HPC IDUN Cluster

## How to build Pytorch 1.3.0 for the IDUN cluster

[This ](https://medium.com/repro-repo/build-pytorch-from-source-on-ubuntu-18-04-1c5556ca8fbf)blog post noticed remarkable performance gains from building pytorch from source.

To build it from source on the IDUN cluster was kind of a hassle, but this script should do it:



```text
module load GCC/8.2.0-2.31.1
module load CUDA/10.1.105
module load Anaconda3/2018.12
module load cuDNN/7.4.2.24
module load NCCL/2.4.2

conda create --name py37 python=3.7
# Got error message when running activate for the first time.
# Had to do what it said, and log onto the node again
conda activate py37

# pytorch dependencies
conda install numpy ninja pyyaml mkl mkl-include setuptools cmake cffi typing --yes
conda install -c pytorch magma-cuda90 --yes
conda install opencv --yes


export CUDNN_INCLUDE_DIR="/share/apps/software/Compiler/GCC-CUDA/8.2.0-2.31.1-10.1.105/cuDNN/7.4.2.24/include/"
export CUDNN_LIB_DIR="/share/apps/software/Compiler/GCC-CUDA/8.2.0-2.31.1-10.1.105/cuDNN/7.4.2.24/lib64/"
export NCCL_INCLUDE_DIR="/share/apps/software/Compiler/GCC-CUDA/8.2.0-2.31.1-10.1.105/NCCL/2.4.2/include/"
export NCCL_LIBRARY="/share/apps/software/Compiler/GCC-CUDA/8.2.0-2.31.1-10.1.105/NCCL/2.4.2/lib/"
export NCCL_LIB_DIR="/share/apps/software/Compiler/GCC-CUDA/8.2.0-2.31.1-10.1.105/NCCL/2.4.2/lib/"
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/share/apps/software/Compiler/GCC-CUDA/8.2.0-2.31.1-10.1.105/NCCL/2.4.2/lib/"
export USE_CUDA=1 USE_CUDNN=1 USE_MKLDNN=1 USE_MKLDNN_CBLAS=1 USE_OPENCV=1

git clone --recursive https://github.com/pytorch/pytorch pytorch_source
cd pytorch_source
# Checkout to version 1.3.0
git checkout tags/v1.3.1
git submodule sync
git submodule update --init --recursive


# install pytorch
python setup.py install

cd ..
git clone https://github.com/pytorch/vision
cd vision
git checkout tags/v0.4.2

python setup.py install
```

