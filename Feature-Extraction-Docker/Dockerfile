FROM --platform=linux/amd64 nvidia/cuda:11.3.1-cudnn8-devel-ubuntu20.04

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
    # Install Dependencies for OpenCV
        build-essential \
        yasm \
        pkg-config \
        libtbb2 \
        libtbb-dev \
        libjpeg-dev \
        libpng-dev \
        libtiff-dev \
        libpq-dev \
        libxine2-dev \
        libglew-dev \
        libtiff5-dev \
        zlib1g-dev \
        libavcodec-dev \
        libavformat-dev \
        libavutil-dev \
        libpostproc-dev \
        libswscale-dev \
        libeigen3-dev \
        libgtk2.0-dev \
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

# Build OpenCV from source
# Source: https://github.com/JulianAssmann/opencv-cuda-docker/blob/master/ubuntu-20.04/opencv-4.5/cuda-11.1/Dockerfile
RUN bash /espnet/tools/activate_python.sh && cd /opt/ &&\
    # Download and unzip OpenCV and opencv_contrib and delete zip files
    wget --quiet https://github.com/opencv/opencv/archive/$OPENCV_VERSION.zip &&\
    unzip -qq ${OPENCV_VERSION}.zip &&\
    rm ${OPENCV_VERSION}.zip &&\
    wget --quiet https://github.com/opencv/opencv_contrib/archive/$OPENCV_VERSION.zip &&\
    unzip -qq ${OPENCV_VERSION}.zip &&\
    rm ${OPENCV_VERSION}.zip &&\
    # Create build folder and switch to it
    mkdir /opt/opencv-${OPENCV_VERSION}/build && cd /opt/opencv-${OPENCV_VERSION}/build &&\
    # Cmake configure
    cmake \
        -DOPENCV_EXTRA_MODULES_PATH=/opt/opencv_contrib-${OPENCV_VERSION}/modules \
        -DBUILD_opencv_python2=OFF \
        -DBUILD_opencv_python3=ON \
        -DWITH_CUDA=ON \
        -DCUDA_ARCH_BIN=3.5,5.0,6.0,6.1,7.0,7.5,8.0,8.6  \
        -DCMAKE_BUILD_TYPE=RELEASE \
        -DCUDNN_VERSION=8 \
        -DPYTHON3_LIBRARY=/opt/conda/lib/python3.8 \
        -DPYTHON3_INCLUDE_DIR=/opt/conda/include/python3.8 \
        -DPYTHON3_EXECUTABLE=/opt/conda/bin/python \
        -DPYTHON3_PACKAGES_PATH=/opt/conda/lib/python3.8/site-packages \
        .. &&\
    # Make
    make -j "$(($(($((`free -g | grep '^Mem:' | grep -o '[^ ]*$'`/2)) < $(nproc) ? $((`free -g | grep '^Mem:' | grep -o '[^ ]*$'`/2)) : $(nproc)))>1 ? $(($((`free -g | grep '^Mem:' | grep -o '[^ ]*$'`/2)) < $(nproc) ? $((`free -g | grep '^Mem:' | grep -o '[^ ]*$'`/2)) : $(nproc))) : 1))" && \
    # Install to /usr/local/lib
    make install && \
    ldconfig && \
    # Remove OpenCV sources and build folder
    rm -rf /opt/opencv-${OPENCV_VERSION} && rm -rf /opt/opencv_contrib-${OPENCV_VERSION}


# Install Tensorflow from source
RUN apt-get update && apt install -y apt-transport-https curl gnupg && \
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
    python -m pip install --no-cache-dir \
        p-tqdm \
        mediapipe==0.10.* \
        ffmpeg-python