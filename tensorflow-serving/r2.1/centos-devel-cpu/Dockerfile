FROM centos:7.4.1708 as base_build

RUN set -ex && \
    yum makecache && \
	yum install -y deltarpm && \
	yum groupinstall -y 'Development Tools' && \
    yum install -y  \
        automake \
        build-essential \
        ca-certificates \
        curl \
        git \
        libcurl-devel \
        freetype-devel \
        libpng12-devel \
        libtool \
        cppzmq-devel czmq-devel \
        mlocate \
        java-1.8.0-openjdk \
        java-1.8.0-openjdk-headless \
        pkg-config \
        python-devel \
        swig \
        unzip \
        wget \
        zip patch which gcc gcc-c++ make cmake\
        zlib-devel \
        && \
    rm -rf /var/cache/yum

RUN set -ex && \
    yum makecache && \
	yum install -y python36 python3-devel python3-pip && \
	curl -fSsL -O https://bootstrap.pypa.io/get-pip.py && \
    python get-pip.py && \
	python3 get-pip.py && \
    rm get-pip.py \
	pip3 --no-cache-dir install --upgrade pip && \
	python -V && \
	python3 -V && \
	pip3 -V && \
	pip -V && \
	rm -rf /var/cache/yum
	

RUN pip --no-cache-dir install \
    future>=0.17.1 \
    grpcio \
    h5py \
    keras_applications>=1.0.8 \
    keras_preprocessing>=1.1.0 \
    mock \
    numpy \
    requests

ENV BAZEL_VERSION 0.24.1
WORKDIR /
RUN mkdir /bazel && \
    cd /bazel && \
    curl -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36" -fSsL -O https://github.com/bazelbuild/bazel/releases/download/$BAZEL_VERSION/bazel-$BAZEL_VERSION-installer-linux-x86_64.sh && \
    curl -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36" -fSsL -o /bazel/LICENSE.txt https://raw.githubusercontent.com/bazelbuild/bazel/master/LICENSE && \
    chmod +x bazel-*.sh && \
    ./bazel-$BAZEL_VERSION-installer-linux-x86_64.sh && \
    cd / && \
    rm -f /bazel/bazel-$BAZEL_VERSION-installer-linux-x86_64.sh

RUN set -ex && \
    curl -o /tmp/serving-2.1.0.tar.gz -fSsL https://codeload.github.com/tensorflow/serving/tar.gz/2.1.0 && \
	tar -C / -zxvf /tmp/serving-2.1.0.tar.gz && \
	mv /serving-2.1.0 /tensorflow-serving && \
	ls -lh /tensorflow-serving && \
	rm -f /tmp/serving-2.1.0.tar.gz
	
# Download TF Serving sources (optionally at specific commit).
WORKDIR /tensorflow-serving

FROM base_build as binary_build

# Build, and install TensorFlow Serving
ARG TF_SERVING_BUILD_OPTIONS="--config=nativeopt"
RUN echo "Building with build options: ${TF_SERVING_BUILD_OPTIONS}"
ARG TF_SERVING_BAZEL_OPTIONS=""
RUN echo "Building with Bazel options: ${TF_SERVING_BAZEL_OPTIONS}"

RUN bazel build --color=yes --curses=yes \
    ${TF_SERVING_BAZEL_OPTIONS} \
    --verbose_failures \
    --output_filter=DONT_MATCH_ANYTHING \
    ${TF_SERVING_BUILD_OPTIONS} \
    tensorflow_serving/model_servers:tensorflow_model_server && \
    cp bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server \
    /usr/local/bin/

# Build and install TensorFlow Serving API
RUN bazel build --color=yes --curses=yes \
    ${TF_SERVING_BAZEL_OPTIONS} \
    --verbose_failures \
    --output_filter=DONT_MATCH_ANYTHING \
    ${TF_SERVING_BUILD_OPTIONS} \
    tensorflow_serving/tools/pip_package:build_pip_package && \
    bazel-bin/tensorflow_serving/tools/pip_package/build_pip_package \
    /tmp/pip && \
    pip --no-cache-dir install --upgrade \
    /tmp/pip/tensorflow_serving_api-*.whl && \
    rm -rf /tmp/pip

FROM binary_build as clean_build
# Clean up Bazel cache when done.
RUN bazel clean --expunge --color=yes && \
    rm -rf /root/.cache
CMD ["/bin/bash"]
