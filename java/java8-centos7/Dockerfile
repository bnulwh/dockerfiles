FROM 5ibnu/centos:7.4
maintainer Liu wenhui <bnulwh@gmail.com>
RUN set -ex && \
    yum makecache && \
    yum install -y java-1.8.0-openjdk-devel git && \
    java -version && \
    git --version && \
    yum clean all && \
    rm -fr /var/cache/yum
ENV JAVA_HOME=/usr/lib/jvm/java-openjdk
ENV CLASSPATH=.;${JAVA_HOME}/lib/dt.jar;${JAVA_HOME}/lib/tools.jar
ENV PATH=$PATH:${JAVA_HOME}/bin
