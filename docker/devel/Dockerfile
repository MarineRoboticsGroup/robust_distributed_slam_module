FROM ubuntu:18.04
LABEL Maintainer="Pierre-Yves Lajoie <lajoie.py@gmail.com>"
LABEL robust-multi-robot-slam-module.version="0.1"

# Install common dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    dpkg \ 
    git \
    pkg-config \
    python \
    python-dev \
    python-numpy \
    software-properties-common \
    && rm -rf /var/lib/apt/lists/*

# Install GTSAM
RUN apt-get update && apt-get install -y \
    libboost-all-dev \
    cmake \
    libtbb-dev \
    && rm -rf /var/lib/apt/lists/*

RUN git clone https://bitbucket.org/gtborg/gtsam && \
    cd gtsam && \
    git checkout develop && \
    mkdir build && \
    cd build && \
    cmake .. && \
    make -j2 && \
    make install

# Install Eigen3 (packaged in GTSAM)
RUN cd gtsam/gtsam/3rdparty/Eigen && \
    mkdir build && \
    cd build && \
    cmake .. &&\
    make install

# Install Buzz
RUN git clone https://github.com/MISTLab/Buzz.git /home/docker/buzz
RUN cd /home/docker/buzz &&\
    mkdir build &&\
    cd build &&\
    cmake -DCMAKE_BUILD_TYPE=Debug ../src &&\
    make &&\
    make install &&\
    ldconfig

# Install and setup some editing and debugging tools
RUN apt-get update &&\
    apt-get install -y gedit nano strace valgrind wget

RUN wget -O vscode.deb https://go.microsoft.com/fwlink/?LinkID=760868 &&\
    apt install -y ./vscode.deb &&\
    apt install -y libasound2

RUN mkdir s/code &&\
    code --install-extension ms-vscode.cpptools --user-data-dir /home/docker/code

# Settings for buzz editing
RUN touch /home/docker/code/User/settings.json &&\
    echo "{ \"files.associations\": { \"*.bzz\":\"python\" } }" > /home/docker/code/User/settings.json

# Add dummy argument to force rebuild starting from that point
ARG CODE_UPDATE=unknown

# Clone simulation
RUN git clone https://github.com/lajoiepy/robust_distributed_slam_module.git /home/docker/robust_distributed_slam_module &&\
    cd /home/docker/robust_distributed_slam_module &&\
    git submodule update --init --recursive

# Install Robust Distributed Mapper
RUN cd /home/docker/robust_distributed_slam_module/buzz_slam/robust_distributed_mapper/cpp &&\
    mkdir build &&\
    cd build &&\
    cmake -DCMAKE_BUILD_TYPE=Debug ../ &&\
    make &&\
    make install

# Install Buzz SLAM
RUN cd /home/docker/robust_distributed_slam_module/buzz_slam &&\
    mkdir build &&\
    cd build &&\
    cmake -DCMAKE_BUILD_TYPE=Debug -DARGOS_SIMULATION=OFF ../src &&\
    make &&\
    make install

# Compile buzz script
RUN cd /home/docker/robust_distributed_slam_module/scripts &&\
    bzzc dataset_test.bzz

# Put your own git credentials to commit edited code
RUN git config --global user.email "lajoie.py@gmail.com" &&\
    git config --global user.name "Pierre-Yves Lajoie"

# Entry point to start the module
WORKDIR /home/docker/robust_distributed_slam_module/scripts