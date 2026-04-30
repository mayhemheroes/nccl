# Copyright 2026 ForAllSecure
#
# Multi-stage build using OSS-Fuzz base builder.
# Stage 1: Build the fuzz target using the OSS-Fuzz toolchain.
# Stage 2: Runtime image containing only the fuzzer binary.

FROM --platform=linux/amd64 gcr.io/oss-fuzz-base/base-builder AS builder

# Install build dependencies (same as OSS-Fuzz project Dockerfile)
RUN apt-get update && apt-get install -y make autoconf automake libtool patchelf

# Install CUDA toolkit
RUN mkdir $SRC/cuda-install && \
    cd $SRC/cuda-install && \
    wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin && \
    mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600 && \
    wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-keyring_1.1-1_all.deb && \
    dpkg -i cuda-keyring_1.1-1_all.deb && \
    apt-get update && \
    apt-get -y install cuda-toolkit-12-9

# Clone the nccl source
RUN git clone --depth 1 https://github.com/NVIDIA/nccl $SRC/nccl

# Copy the OSS-Fuzz harness and build script
COPY fuzz_xml.cpp $SRC/
COPY build.sh $SRC/

WORKDIR $SRC/nccl

# Compile
RUN compile

# Stage 2: Runtime image
FROM --platform=linux/amd64 ubuntu:22.04

RUN apt-get update && apt-get install -y libstdc++6 && rm -rf /var/lib/apt/lists/*

COPY --from=builder /out/fuzz_xml /fuzz_xml
COPY --from=builder /out/libcudart.so.12 /libcudart.so.12
COPY --from=builder /out/libcudart.so /libcudart.so

ENV LD_LIBRARY_PATH=/

ENTRYPOINT ["/fuzz_xml"]
