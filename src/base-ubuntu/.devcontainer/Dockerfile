# [Choice] Ubuntu version (use jammy on local arm64/Apple Silicon): jammy, focal
ARG VARIANT="lunar"
FROM buildpack-deps:${VARIANT}-curl

#LABEL dev.containers.features="common"

# [Optional] Uncomment this section to install additional packages.
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
  && apt-get -y install --no-install-recommends git gcc-13 g++-13 mold cmake automake make ninja-build ccache rsync gdb pkg-config zip unzip tar dotnet-host-7.0 dotnet-sdk-7.0 gpg pass inotify-tools sudo\
  # avhengigheter for heaptrack
  libdw-dev libunwind-dev zlib1g zlib1g-dev libboost-iostreams-dev libboost-program-options-dev libboost-system-dev libboost-filesystem-dev libboost-container-dev\
  # avhengigheter for perf
  flex bison libssl-dev libelf-dev python3 wget xz-utils\ # flere utilities, men bygget feiler da binutils-dev libzstd-dev libcap-dev\
  && echo ubuntu ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/ubuntu \
  && chmod 0440 /etc/sudoers.d/ubuntu

RUN  update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-13 20 \
  && update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-13 20 \
  && update-alternatives --install /usr/bin/cc cc /usr/bin/gcc 30 \
  && update-alternatives --set cc /usr/bin/gcc \
  && update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++ 30 \
  && update-alternatives --set c++ /usr/bin/g++

COPY ./dockerindocker.sh /home/ubuntu/dockerindocker.sh
RUN chmod +x /home/ubuntu/dockerindocker.sh 
RUN DOCKERDASHCOMPOSEVERSION=v2 USERNAME=ubuntu MOBY=false /home/ubuntu/dockerindocker.sh 


WORKDIR /home/ubuntu
# Bygg perf
ARG PERF_VER="5.15"
RUN wget https://mirrors.edge.kernel.org/pub/linux/kernel/tools/perf/v5.15.0/perf-$PERF_VER.0.tar.xz\
  && tar xf perf-$PERF_VER.0.tar.xz\
  && mv perf-$PERF_VER.0 perf_build\
  && cd perf_build\
# Måtte fjerne -Werror en del Makefiles før bygging fordi vi bygger med nyere GCC som har fått flere warnings-as-errors.
  && find . -type f | xargs sed -i 's/-Werror//g'\
# Når bygging er ferdig skal perf være klar i /home/ubuntu/per_build/tools/perf/perf
  && make -C tools/perf install

WORKDIR /home/ubuntu
# Bygg heaptrack
RUN git clone https://github.com/KDE/heaptrack.git && cd heaptrack && mkdir build && cd build && cmake -DCMAKE_BUILD_TYPE=Release .. && make -j$(nproc)
# Når bygging er ferdig skal perf være klar i /home/ubuntu/heaptrack/build/bin/heaptrack
