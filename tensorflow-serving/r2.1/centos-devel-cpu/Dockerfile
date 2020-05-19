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
    yum install  gmp  gmp-devel  mpfr  mpfr-devel  libmpc  libmpc-devel -y && \
    curl -fSsL -o /tmp/gcc-7.5.0.tar.gz https://bigsearcher.com/mirrors/gcc/releases/gcc-7.5.0/gcc-7.5.0.tar.gz && \
	tar -C /tmp -zxvf /tmp/gcc-7.5.0.tar.gz && \
	mkdir -p /tmp/gcc-7.5.0/build && \
	cd /tmp/gcc-7.5.0/build && \
	../configure --prefix=/usr/local/gcc --enable-checking=release --enable-languages=c,c++ --disable-multilib && \
	make -j 4 &&  \
	make install && \
	find / -name "libstdc++.so*" && \
	gcc --version && \
	mv /usr/bin/gcc /usr/bin/gcc_old && \
	ln -s /usr/local/gcc/bin/gcc /usr/bin/gcc && \
	mv /usr/bin/g++ /usr/bin/g++_old && \
	ln -s /usr/local/gcc/bin/g++ /usr/bin/g++ && \
	gcc --version && \
	g++ --version && \
	rm -fr /tmp/gcc-7.5.0 && \
	rm -f /tmp/gcc-7.5.0.tar.gz && \
	rm -rf /var/cache/yum
	
RUN set -ex && \    
	curl -fSsL -O https://bootstrap.pypa.io/get-pip.py && \
    python get-pip.py && \
    rm get-pip.py && \
	pip --no-cache-dir install --upgrade pip && \
	python -V && \
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
    rm -f /lib64/libstdc++.so.6 && \
    rm -f /usr/lib64/libstdc++.so.6 && \
    ln -sf /usr/local/gcc/lib64/libstdc++.so.6 /usr/lib64/libstdc++.so.6 && \
    ln -sf /usr/local/gcc/lib64/libstdc++.so.6 /lib64/libstdc++.so.6 && \
    strings /lib64/libstdc++.so.6 |grep CXXABI|sort|uniq && \
    strings /lib64/libstdc++.so.6 |grep '^GLIBCXX'|sort|uniq


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
