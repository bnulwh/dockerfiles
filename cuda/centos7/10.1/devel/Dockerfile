# ARG IMAGE_NAME
FROM 5ibnu/cuda:10.1-runtime-centos7.4
LABEL maintainer "NVIDIA CORPORATION <sw-cuda-installer@nvidia.com>"

RUN set -ex && yum install -y \
        cuda-nvml-dev-$CUDA_PKG_VERSION \
        cuda-command-line-tools-$CUDA_PKG_VERSION \
cuda-libraries-dev-$CUDA_PKG_VERSION \
        cuda-minimal-build-$CUDA_PKG_VERSION \
        libcublas-devel-10.2.1.243-1 \
        && \
        rpm -e --nodeps kernel-headers  && \
        rpm -import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org && \
        rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm && \
        yum -y --enablerepo=elrepo-kernel install kernel-ml-headers.x86_64 && \
        yum clean all && \
        rm -rf /var/cache/yum/* && \
        rpm -qa

ENV LIBRARY_PATH /usr/local/cuda/lib64/stubs