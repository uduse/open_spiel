FROM ubuntu:20.04 as base

ENV DEBIAN_FRONTEND=noninteractive 
ENV OPEN_SPIEL_ENABLE_JAX=ON

RUN apt update
RUN dpkg --add-architecture i386 && apt update
RUN apt-get -y install \
    clang \
    curl \
    git \
    python3 \
    python3-dev \
    python3-pip \
    python3-setuptools \
    python3-wheel \
    sudo \
    gdb
RUN echo 'alias python=python3' >> ~/.bashrc
RUN mkdir repo
WORKDIR /repo

RUN sudo pip3 install --upgrade pip
RUN sudo pip3 install matplotlib
RUN sudo pip3 install tensorflow

# install
COPY . .
RUN apt-get -y install tzdata
RUN ./install.sh
RUN pip3 install --upgrade setuptools testresources
RUN pip3 install --upgrade -r requirements.txt
RUN pip3 install --upgrade cmake

# build and test
RUN mkdir -p build
WORKDIR /repo/build
RUN cmake -DPython3_EXECUTABLE=`which python3` -DCMAKE_CXX_COMPILER=`which clang++` ../open_spiel
RUN make --j$(nproc)
ENV PYTHONPATH=${PYTHONPATH}:/repo
ENV PYTHONPATH=${PYTHONPATH}:/repo/build/python
RUN ctest --j$(nproc)

# build shared lib
WORKDIR /repo/build
RUN BUILD_SHARED_LIB=ON CXX=clang++ cmake -DPython3_EXECUTABLE=$(which python3) -DCMAKE_CXX_COMPILER=${CXX} ../open_spiel
RUN make -j$(nproc) open_spiel
ENV LD_LIBRARY_PATH="/repo/build"
WORKDIR /repo/