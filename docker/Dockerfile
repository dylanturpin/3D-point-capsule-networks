FROM nvidia/cudagl:10.1-devel-ubuntu18.04

# apt installable dependencies
RUN DEBIAN_FRONTEND=noninteractive apt-get update -y &&\
  DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  build-essential \
  cmake \
  curl \
  ca-certificates \
  ffmpeg \
  git \
  glew-utils \
  libboost-dev \
  libboost-thread-dev \
  libcudnn7 \
  libglew-dev \
  libeigen3-dev \
  libegl1-mesa-dev \
  libglfw3-dev \
  libgmp-dev \
  libhdf5-dev \
  libmpfr-dev \
  libosmesa6-dev \
  libsm6 \
  libtbb-dev \
  openssh-client \
  psmisc \
  rsync \
  software-properties-common \
  unzip \
  wget \
  vim

# update to gcc 7 (or torch-nndistance build fails)
#RUN DEBIAN_FRONTEND=noninteractive add-apt-repository -y ppa:ubuntu-toolchain-r/test &&\
  #DEBIAN_FRONTEND=noninteractive apt-get update -y &&\
  #DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  #gcc-7 g++-7 &&\
  #mv /usr/bin/gcc-7 /usr/bin/gcc

RUN DEBIAN_FRONTEND=noninteractive add-apt-repository -y ppa:openscad/releases &&\
  DEBIAN_FRONTEND=noninteractive apt-get update -y &&\
  DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  openscad

# install anaconda
RUN wget https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh -O ./anaconda.sh &&\
  /bin/bash ./anaconda.sh -b -p /opt/conda &&\
  /opt/conda/bin/conda init bash &&\
  rm ./anaconda.sh

# create new conda env
RUN /opt/conda/bin/conda create -y -n py37 python=3.7

# install pytorch
RUN /opt/conda/bin/conda run -n py37 pip install torch==1.5.0+cu101 torchvision==0.6.0+cu101 -f https://download.pytorch.org/whl/torch_stable.html

# install wandb client from fork with pyyaml fix
RUN mkdir /pkgs &&\
  cd /pkgs &&\
  git clone https://github.com/dylanturpin/client &&\
  cd /pkgs/client &&\
  /opt/conda/bin/conda run -n py37 pip install -e .

# install some pip packages for capsnet
RUN /opt/conda/bin/conda run -n py37 pip install h5py matplotlib

# build open3d with headless rendering
# first need to update cmake
RUN DEBIAN_FRONTEND=noninteractive apt-get update -y &&\
  DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  apt-transport-https ca-certificates gnupg software-properties-common wget &&\
  wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | gpg --dearmor - | tee /etc/apt/trusted.gpg.d/kitware.gpg >/dev/null &&\
  apt-add-repository 'deb https://apt.kitware.com/ubuntu/ bionic main' &&\
  DEBIAN_FRONTEND=noninteractive apt-get update -y &&\
  DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  cmake

RUN cd /pkgs &&\
  cd /pkgs &&\
  git clone --recursive https://github.com/intel-isl/Open3D &&\
  mkdir /pkgs/Open3D/build &&\
  cd /pkgs/Open3D/build &&\
  /opt/conda/bin/conda run -n py37 cmake -DENABLE_HEADLESS_RENDERING=ON \
                 -DENABLE_GUI=OFF \
                 -DBUILD_GLEW=ON \
                 -DBUILD_GLFW=ON \
                 .. &&\
  /opt/conda/bin/conda run -n py37 make -j16 &&\
  /opt/conda/bin/conda run -n py37 make install-pip-package

# install chamfer distance package
#RUN cd /pkgs &&\
  #git clone https://github.com/dylanturpin/3D-point-capsule-networks &&\
  #cd /pkgs/3D-point-capsule-networks/models/nndistance &&\
  #/opt/conda/bin/conda run -n py37 python build.py install
