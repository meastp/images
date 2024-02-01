# [Choice] Ubuntu version (use jammy on local arm64/Apple Silicon): jammy, focal
ARG VARIANT="lunar"
FROM buildpack-deps:${VARIANT}-curl

#LABEL dev.containers.features="common"

# [Optional] Uncomment this section to install additional packages.
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
  && apt-get -y install --no-install-recommends git gcc-13 g++-13 mold cmake automake make ninja-build ccache rsync gdb pkg-config zip unzip tar dotnet-host-7.0 dotnet-sdk-7.0 gpg pass inotify-tools sudo\
  && echo ubuntu ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/ubuntu \
  && chmod 0440 /etc/sudoers.d/ubuntu

RUN  update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-13 20 \
  && update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-13 20 \
  && update-alternatives --install /usr/bin/cc cc /usr/bin/gcc 30 \
  && update-alternatives --set cc /usr/bin/gcc \
  && update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++ 30 \
  && update-alternatives --set c++ /usr/bin/g++

# Installer Microsoft ODBC
# Konfigurere sertifikat/pakker: https://learn.microsoft.com/en-us/linux/packages
# https://learn.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-ver16

RUN curl -sSL -O  https://packages.microsoft.com/config/ubuntu/23.04/packages-microsoft-prod.deb \
  && dpkg -i packages-microsoft-prod.deb \
  && rm packages-microsoft-prod.deb \
  && apt-get update \
  && ACCEPT_EULA=Y apt-get -y install --no-install-recommends msodbcsql18 unixodbc-dev
