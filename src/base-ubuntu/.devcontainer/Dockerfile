# [Choice] Ubuntu version (use jammy on local arm64/Apple Silicon): jammy, focal
ARG VARIANT="questing"

FROM buildpack-deps:${VARIANT}-curl

# RE-DECLARE HERE to make it available to RUN commands
ARG VARIANT
ARG GCCVER="15"
ARG PERF_VER="6.17"

#LABEL dev.containers.features="common"

# [Optional] Uncomment this section to install additional packages.
RUN apt-get update && apt-get -y install ubuntu-dbgsym-keyring
# To see which repos are activated: RUN cat /etc/apt/sources.list.d/ubuntu.sources
RUN echo "deb http://ddebs.ubuntu.com ${VARIANT} main restricted universe multiverse" | tee -a /etc/apt/sources.list.d/ddebs.list
RUN echo "deb http://ddebs.ubuntu.com ${VARIANT}-updates main restricted universe multiverse" | tee -a /etc/apt/sources.list.d/ddebs.list
#RUN echo "deb http://ddebs.ubuntu.com ${VARIANT}-proposed main restricted universe multiverse" | tee -a /etc/apt/sources.list.d/ddebs.list

RUN  apt-get update && export DEBIAN_FRONTEND=noninteractive && apt-get -y upgrade \
  && apt-get -y install --no-install-recommends git gcc-${GCCVER} g++-${GCCVER} mold cmake automake make ninja-build ccache rsync gdb pkg-config zip unzip tar libtool dotnet-host-8.0 dotnet-sdk-8.0 gpg pass inotify-tools sudo linux-libc-dev\
  # avhengigheter for heaptrack
  libdw-dev libunwind-dev zlib1g zlib1g-dev libboost-iostreams-dev libboost-program-options-dev libboost-system-dev libboost-filesystem-dev libboost-container-dev\
  # avhengigheter for perf
  patch flex bison libssl-dev elfutils libaudit-dev libelf-dev libcap-dev wget xz-utils libzstd-dev binutils-dev libiberty-dev libslang2-dev libcap-dev systemtap-sdt-dev python3 python3-dev python3-distutils-extra\ 
  libtraceevent-dev libbabeltrace-dev\
  libc6-dbg libgcc-s1-dbgsym libstdc++6-dbgsym\
  # gcovr coverage
  pipx\
  && echo ubuntu ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/ubuntu \
  && chmod 0440 /etc/sudoers.d/ubuntu

RUN  update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-${GCCVER} 20 \
  && update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-${GCCVER} 20 \
  && update-alternatives --install /usr/bin/gcov gcov /usr/bin/gcov-${GCCVER} 20 \
  && update-alternatives --install /usr/bin/cc cc /usr/bin/gcc 30 \
  && update-alternatives --set cc /usr/bin/gcc \
  && update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++ 30 \
  && update-alternatives --set c++ /usr/bin/g++

COPY ./dockerindocker.sh /home/ubuntu/dockerindocker.sh
RUN chmod +x /home/ubuntu/dockerindocker.sh 
RUN DOCKERDASHCOMPOSEVERSION=v2 USERNAME=ubuntu MOBY=false /home/ubuntu/dockerindocker.sh 

WORKDIR /home/ubuntu

# Bygg perf
RUN wget https://mirrors.edge.kernel.org/pub/linux/kernel/tools/perf/v$PERF_VER.0/perf-$PERF_VER.0.tar.xz\
  && tar xf perf-$PERF_VER.0.tar.xz\
  && mv perf-$PERF_VER.0 perf_build\
  && cd perf_build\
# Måtte fjerne -Werror en del Makefiles før bygging fordi vi bygger med nyere GCC som har fått flere warnings-as-errors.
  && find . -type f | xargs sed -i 's/-Werror//g'\
# Lurt å bygge statisk binærfil for å kunne kopiere rundt uten ekstra avhengigheter
#  && sed -i 's/# Define LDFLAGS=-static to build a static binary./LDFLAGS=-static/g' tools/perf/Makefile.perf\ 
# Når bygging er ferdig skal perf være klar i /home/ubuntu/per_build/tools/perf/perf
  && export LIBDW_DIR=/usr/include/elfutils && make -C tools/perf install

WORKDIR /home/ubuntu
# Bygg heaptrack
RUN git clone https://github.com/KDE/heaptrack.git && cd heaptrack && mkdir build && cd build && cmake -DCMAKE_BUILD_TYPE=Release .. && make -j$(nproc)
# Når bygging er ferdig skal heaptrack være klar i /home/ubuntu/heaptrack/build/bin/heaptrack

USER ubuntu

RUN pipx ensurepath\ 
 && pipx install gcovr
