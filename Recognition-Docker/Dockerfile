FROM --platform=linux/amd64 ubuntu:20.04

LABEL maintainer="Gururaj Deshpande <gurudesh@gatech.edu>"

ARG DEBIAN_FRONTEND=noninteractive

# Install dependencies
RUN rm -f /etc/apt/sources.list.d/cuda.list && \
    rm -f /etc/apt/sources.list.d/nvidia-ml.list && \
    apt-get update && apt-get upgrade -y &&\
    # Install Overall Dependencies
    apt-get install -y --no-install-recommends \
        g++ \
        make \
        cmake \
        automake \
        autoconf \
        bzip2 \
        unzip \
        wget \
        sox \
        libtool \
        subversion \
        zlib1g-dev \
        gfortran \
        ca-certificates \
        patch \
        ffmpeg \
        libsndfile1-dev \
        flac \
        curl \
        nkf \
        libfreetype6-dev \
        software-properties-common \
        libopencv-dev \
    # Install Dependencies for HTK
    && apt-get update && apt-get upgrade -y && apt-get install -y libc6-dev-i386 \
        ksh \
        bc \
        libx11-dev \
        # Since Guru had a Windows Computer, he needs to convert the Windows Line Endings to Unix Line Endings
        dos2unix \ 
    && apt-get clean && rm -rf /var/lib/apt/lists/* && \
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

# Install HTK
COPY ./htk ./htk
COPY ./gt2k ./gt2k
COPY ./prepare ./htk/prepare
# Deleted LetterLevel and WordLevel folders because they are not used in this project
WORKDIR /htk
# This prepare script is where htk is built from scratch
RUN dos2unix prepare && chmod +x prepare && ./prepare

# Install Pytorch
RUN conda install -y pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia

# Install Tensorflow
RUN conda install -y -c conda-forge cudatoolkit=11.8.0 && \
    python3 -m pip install nvidia-cudnn-cu11==8.6.0.163 && \
    mkdir -p $CONDA_PREFIX/etc/conda/activate.d && \
    echo 'CUDNN_PATH=$(dirname $(python -c "import nvidia.cudnn;print(nvidia.cudnn.__file__)"))' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh && \
    echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CONDA_PREFIX/lib/:$CUDNN_PATH/lib' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh && \
    python3 -m pip install --upgrade pip && \
    pip install tensorflow==2.12.*

# Install other Python dependencies
RUN conda install -y -c huggingface \
    transformers \
    scipy \
    tensorboard \
    ipykernel \
    ipywidgets \
    jupyter \
    matplotlib \
    pandas \
    scikit-learn \
    seaborn \
    tqdm \
    plotly \
    pyarrow && \
    conda clean -ya && \
    python3 -m pip install opencv-python-headless \
        p-tqdm

