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

# Install AlphaPose
RUN cd ~ && \
    conda install pytorch torchvision torchaudio pytorch-cuda=11.3 -c pytorch -c nvidia && \
    git clone https://github.com/MVIG-SJTU/AlphaPose.git && \
    cd AlphaPose && \
    python -m pip install cython && \
    python setup.py build develop

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
    && conda clean -ya && \
    python -m pip install opencv-python-headless \
        p-tqdm \
        mediapipe==0.10.* \
        ffmpeg-python