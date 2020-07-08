#ARG IMAGE_NAME
FROM 5ibnu/cuda:10.0-base-centos7.4
LABEL maintainer "NVIDIA CORPORATION <sw-cuda-installer@nvidia.com>"

RUN yum install -y \
        cuda-libraries-$CUDA_PKG_VERSION \
cuda-nvtx-$CUDA_PKG_VERSION \
    && \
    rm -rf /var/cache/yum/*