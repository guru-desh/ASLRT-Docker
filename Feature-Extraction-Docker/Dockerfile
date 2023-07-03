FROM --platform=linux/amd64 nvidia/cuda:11.3.0-cudnn8-devel-ubuntu20.04

LABEL maintainer="Gururaj Deshpande <gurudesh@gatech.edu>"

ARG DEBIAN_FRONTEND=noninteractive
ARG OPENCV_VERSION=4.5.0

# Install dependencies
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
        build-essential \
        cmake \
        git \
        libprotobuf-dev \
        protobuf-compiler \
        python3 \
        python3-pip \
        wget \
        libgoogle-glog-dev \
        libgflags-dev \
        libyaml-dev \
        unzip \ 
        libopencv-dev \
        software-properties-common \
        libboost-all-dev \
        libhdf5-dev \
        ffmpeg \
        libatlas-base-dev \
    && rm -rf /var/lib/apt/lists/* && \
    ldconfig /usr/local/cuda/lib64

# Install Miniconda
ENV CONDA_DIR /opt/conda
RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm ~/miniconda.sh
ENV PATH=$CONDA_DIR/bin:$PATH

# Initialize conda
RUN conda init bash && \
    echo "conda activate base" >> ~/.bashrc
RUN bash -c "source ${HOME}/.bashrc" 

# Set CUDA environment variables
ENV CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda \
    Protobuf_INCLUDE_DIR=/usr/include \
    PATH=/usr/local/cuda/bin/:$PATH \
    LD_LIBRARY_PATH=/usr/local/cuda/lib64/:$LD_LIBRARY_PATH \
    HDF5_ROOT=/usr/lib/x86_64-linux-gnu/hdf5/serial/ \
    ATLAS_ROOT=/usr/lib/atlas-base/

# Build OpenPose from source
RUN cd / && git clone https://github.com/CMU-Perceptual-Computing-Lab/openpose && \
    cd openpose/ && \
    git submodule update --init --recursive --remote && \
    mkdir build && \
    cd build && \
    cmake ../ -G "Unix Makefiles" -DBUILD_PYTHON=ON && \
    make -j "$(($(($((`free -g | grep '^Mem:' | grep -o '[^ ]*$'`/2)) < $(nproc) ? $((`free -g | grep '^Mem:' | grep -o '[^ ]*$'`/2)) : $(nproc)))>1 ? $(($((`free -g | grep '^Mem:' | grep -o '[^ ]*$'`/2)) < $(nproc) ? $((`free -g | grep '^Mem:' | grep -o '[^ ]*$'`/2)) : $(nproc))) : 1))"

ENV TORCH_CUDA_ARCH_LIST="3.5;5.0;6.0;6.1;7.0;7.5;8.0;8.6+PTX"

# Install AlphaPose
RUN cd ~ && \
    conda install pytorch==1.12.1 torchvision==0.13.1 torchaudio==0.12.1 cudatoolkit=11.3 -c pytorch && \
    git clone https://github.com/MVIG-SJTU/AlphaPose.git && \
    cd AlphaPose && \
    python -m pip install --no-cache-dir cython natsort halpecocotools pycocotools && \
    python setup.py build develop --user

# Install Tensorflow from source
RUN apt-get update && apt install apt-transport-https curl gnupg && \
    curl -fsSL https://bazel.build/bazel-release.pub.gpg | gpg --dearmor >bazel-archive-keyring.gpg && \
    mv bazel-archive-keyring.gpg /usr/share/keyrings && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/bazel-archive-keyring.gpg] https://storage.googleapis.com/bazel-apt stable jdk1.8" | tee /etc/apt/sources.list.d/bazel.list && \
    apt-get update && \
    apt-get install -y bazel-5.0.0 && \
    ln -s /usr/bin/bazel-5.0.0 /usr/bin/bazel && \
    apt-get install -y openjdk-11-jdk && \
    apt-get update && \
    rm -rf /var/lib/apt/lists/*
ENV TF_NEED_CUDA=1 \
    TF_CUDA_VERSION=11.3 \
    CUDA_TOOLKIT_PATH=/usr/local/cuda \
    TF_CUDNN_VERSION=8 \
    CUDNN_INSTALL_PATH=/usr \
    TF_CUDA_COMPUTE_CAPABILITIES=3.5,5.0,6.0,6.1,7.0,7.5,8.0,8.6 \
    CC_OPT_FLAGS="--config=cuda" \
    PYTHON_BIN_PATH="/opt/conda/bin/python"

RUN rm -rf /tensorflow && git clone https://github.com/tensorflow/tensorflow.git && \
    cd tensorflow && \
    git checkout r2.9 && \
    ./configure && \
    bazel clean --expunge && \
    bazel build --config=cuda --local_ram_resources="$(free -m | grep '^Mem:' | grep -o '[^ ]*$')" //tensorflow/tools/pip_package:build_pip_package

RUN cd /tensorflow && \ 
    bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg && \
    python3 -m pip install /tmp/tensorflow_pkg/tensorflow-2.9*.whl

# Install Python dependencies
RUN conda install -y -c conda-forge \
        pip \
        matplotlib \
        numpy \
        pandas \
        scipy \
        scikit-learn \
        scikit-image \
        seaborn \
        tqdm \
        pyarrow \
        ipykernel \
        ipywidgets \
        jupyter \
        plotly \
        tf-bodypix \
    && conda clean -ya && \
    python -m pip install --no-cache-dir opencv-python-headless \
        p-tqdm \
        mediapipe==0.10.* \
        ffmpeg-python