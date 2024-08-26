ARG UBUNTU_VERSION=22.04
ARG CUDA_MAJOR_VERSION=11.8.0
ARG CUDNN_MAJOR_VERSION=8
FROM nvidia/cuda:${CUDA_MAJOR_VERSION}-cudnn${CUDNN_MAJOR_VERSION}-runtime-ubuntu${UBUNTU_VERSION} AS base

# install dependencies
ENV DEBIAN_FRONTEND=noninteractive TZ=Europe/Amsterdam
USER root
RUN apt-get update && apt-get install -y --no-install-recommends \
        openssh-server curl wget cmake libglib2.0-dev \
         gcc clang htop xz-utils ca-certificates \
        python3-pip python3-dev libopencv-dev python3-opencv \
        libqt5concurrent5 libqt5core5a libqt5gui5 libqt5widgets5 \
        man apt-transport-https sudo git subversion \
        g++ meson ninja-build pv bzip2 zip unzip dcmtk libboost-all-dev \
        libgomp1 libjpeg-turbo8 libssl-dev zlib1g-dev libncurses5-dev libncursesw5-dev \
        libreadline-dev libsqlite3-dev libgdbm-dev libdb5.3-dev libbz2-dev \
        libexpat1-dev liblzma-dev tk-dev gcovr libffi-dev uuid-dev \
        libgtk2.0-dev libgsf-1-dev libtiff5-dev libopenslide-dev \
        libgl1-mesa-glx libgirepository1.0-dev libexif-dev librsvg2-dev fftw3-dev orc-0.4-dev \
    && rm -rf /var/lib/apt/lists/*
RUN apt-get update && apt-get install --only-upgrade libpmix2

# install Python
ARG PYTHON_VERSION=3.11.3
ARG BUILD_JOBS=16
RUN cd /tmp && \
    wget "https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tar.xz" && \
    tar xfv Python*.xz && \
    cd Python-3*/ && \
    ./configure --enable-shared LDFLAGS="-fprofile-arcs" && \
    make -j${BUILD_JOBS} install && \
    cd ~ && \
    rm -rf /tmp/Python-3* && \
    ldconfig

# install ASAP
ARG ASAP_URL=https://github.com/computationalpathologygroup/ASAP/releases/download/ASAP-2.1-(Nightly)/ASAP-2.1-Ubuntu2204.deb
RUN curl -L ${ASAP_URL} -o /tmp/ASAP.deb && apt-get install --assume-yes /tmp/ASAP.deb && \
    SITE_PACKAGES=`python3 -c "import sysconfig; print(sysconfig.get_paths()['purelib'])"` && \
    printf "/opt/ASAP/bin/\n" > "${SITE_PACKAGES}/asap.pth" && apt-get clean

# install miniconda
RUN mkdir -p /root/miniconda3
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /root/miniconda3/miniconda.sh
RUN bash /root/miniconda3/miniconda.sh -b -u -p /root/miniconda3
RUN rm -rf /root/miniconda3/miniconda.sh
ENV PATH="/root/miniconda3/bin:$PATH"

WORKDIR /home/user/

# clone dinov2 repo
RUN git clone https://github.com/DIAGNijmegen/dinov2.git
WORKDIR /home/user/dinov2
RUN git switch aws

# configure conda environment
COPY conda.yaml /tmp/.
RUN conda env create -f /tmp/conda.yaml -n dinov2
RUN ln -sf /root/miniconda3/bin/python /usr/bin/python3 && \
    ln -sf /root/miniconda3/bin/python /usr/bin/python

# activate conda environment
RUN echo "source /root/miniconda3/etc/profile.d/conda.sh && conda activate dinov2" >> ~/.bashrc

# add dinov2 folder to python path
ENV PYTHONPATH="/home/user/dinov2:$PYTHONPATH"
ENV CUDA_LAUNCH_BLOCKING=1
ENV TORCH_USE_CUDA_DSA=1

ENTRYPOINT ["/bin/bash", "-c", "source /root/miniconda3/etc/profile.d/conda.sh && conda activate dinov2 && exec \"$@\"", "--"]