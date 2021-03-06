## Stage: Build driver
ARG D_BASE_IMAGE=ubuntu:20.04
FROM $D_BASE_IMAGE AS build

# Build Args - pass with --build-arg flag during build
ARG D_OFED_VERSION="5.0-2.1.8.0"
ARG D_OS="ubuntu20.04"
ARG D_ARCH="x86_64"
ARG D_OFED_PATH="MLNX_OFED_LINUX-${D_OFED_VERSION}-${D_OS}-${D_ARCH}"

ARG D_OFED_TARBALL_NAME="MLNX_OFED_LINUX-${D_OFED_VERSION}-${D_OS}-${D_ARCH}.tgz"
ARG D_OFED_URL_PATH="https://www.mellanox.com/downloads/ofed/MLNX_OFED-${D_OFED_VERSION}/${D_OFED_TARBALL_NAME}"

# Internal arguments
ARG D_WITHOUT_FLAGS="--without-rshim-dkms --without-iser-dkms --without-isert-dkms --without-srp-dkms --without-kernel-mft-dkms --without-mlnx-rdma-rxe-dkms"

# Download and extract tarball
WORKDIR /root
ADD ${D_OFED_URL_PATH} ./
RUN tar -xf ${D_OFED_TARBALL_NAME}
RUN apt-get -yq update; apt-get -yq install perl

# Install OFED
RUN /bin/bash -c '/root/MLNX_OFED_LINUX-${D_OFED_VERSION}-${D_OS}-${D_ARCH}/mlnxofedinstall --without-fw-update --kernel-only --force ${D_WITHOUT_FLAGS}'

# Post Install steps

# dont load kernel (builtin) esp4/6 modules related to innova
RUN sed -i '/ESP_OFFLOAD_LOAD=yes/c\ESP_OFFLOAD_LOAD=no' /etc/infiniband/openib.conf

# Put a post start hook in place for restoring network configurations of mellanox net devs
RUN cp /root/MLNX_OFED_LINUX-${D_OFED_VERSION}-${D_OS}-${D_ARCH}/docs/scripts/openibd-post-start-configure-interfaces/post-start-hook.sh /etc/infiniband/post-start-hook.sh && \
    chmod +x /etc/infiniband/post-start-hook.sh

## Stage: Build container
FROM build

WORKDIR /
ADD ./entrypoint.sh /root/entrypoint.sh

ENTRYPOINT ["/root/entrypoint.sh"]
