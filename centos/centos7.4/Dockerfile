FROM centos:7.4.1708 as base_build
# install develop tools ,build tools
RUN set -ex && \
           yum makecache && \
           yum groupinstall -y 'Development Tools' -y && \
           yum install -y gcc gcc-c++ cmake vim which tree expat-devel glib2-devel &&\
        rm -fr /var/cache/yum

# compile bash
RUN set -ex && \
          curl -o /tmp/bash-5.0.tar.gz -fSL http://ftp.gnu.org/gnu/bash/bash-5.0.tar.gz && \
          cd /tmp && \
         tar zxvf  bash-5.0.tar.gz && \
          cd bash-5.0 && ./configure && make && make install && \
         rm -fr /tmp/* && \
          cp -f /usr/local/bin/bash /usr/bin/ && \
          cp -f /usr/local/bin/bash /bin/ && \
          which bash && \
          whereis bash && \
          bash -version

# compile libdbus
RUN set -ex && \
          curl -o /tmp/dbus-1.12.18.tar.gz -fSL https://dbus.freedesktop.org/releases/dbus/dbus-1.12.18.tar.gz && \
          cd /tmp && tar zxvf dbus-1.12.18.tar.gz && \
          cd dbus-1.12.18 && \
          ./configure --prefix=/usr                        \
            --sysconfdir=/etc                    \
            --localstatedir=/var                 \
            --enable-user-session                \
            --disable-doxygen-docs               \
            --disable-xml-docs                   \
            --disable-static                     \
            --with-systemduserunitdir=no         \
            --with-systemdsystemunitdir=no       \
            --docdir=/usr/share/doc/dbus-1.12.18 \
            --with-console-auth-dir=/run/console \
            --with-system-pid-file=/run/dbus/pid \
            --with-system-socket=/run/dbus/system_bus_socket && make && make install && \
#            mv -v /usr/lib/libdbus-1.so.* /lib && \
        ln -sf /usr/lib/libdbus-1.so.3.19.12 /usr/lib/libdbus-1.so && \
        ln -sf /usr/lib/libdbus-1.so.3.19.12 /usr/lib/libdbus-1.so.3 && \
        rm -f /usr/lib64/libdbus-1.so.* && \
        ldconfig && \
        chown -v root:dbus /usr/libexec/dbus-daemon-launch-helper && \
        chmod -v      4750       /usr/libexec/dbus-daemon-launch-helper && \
        dbus-uuidgen --ensure && \
        rm -fr /tmp/* && \
        ls -lh /usr/lib 

# compile libdbus-glib
RUN  set -ex && \
        curl -o /tmp/dbus-glib-0.102.tar.gz -fSL https://dbus.freedesktop.org/releases/dbus-glib/dbus-glib-0.102.tar.gz && \
        cd /tmp && tar  zxvf dbus-glib-0.102.tar.gz && cd dbus-glib-0.102 && \
        export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/lib/pkgconfig && \
        ./configure --prefix=/usr    --sysconfdir=/etc    --disable-static && \
        make && make install && cd /tmp && \
        rm -fr /tmp/* && \
        ls -lh /usr/lib

# merge image files first
FROM centos:7.4.1708 as release_build
RUN set -ex && \
           yum makecache && \
           yum upgrade -y && \
           rpm -qa && \
           rpm -e --nodeps dbus-glib-0.100-7.el7.x86_64 && \
           rpm -qa |grep dbus && \
           rm -fr /var/cache/yum
COPY --from=base_build /usr/local/bin/bash /usr/bin/bash
COPY --from=base_build /usr/local/bin/bash /bin/bash
COPY --from=base_build /usr/lib/libdbus-1.so.3.19.12 /usr/lib/libdbus-1.so.3.19.12
COPY --from=base_build /usr/lib/libdbus-1.la /usr/lib/libdbus-1.la
COPY --from=base_build /usr/lib/libdbus-glib-1.so.2.2.2 /usr/lib/libdbus-glib-1.so.2.2.2
COPY --from=base_build /usr/lib/libdbus-glib-1.la /usr/lib/libdbus-glib-1.la
COPY --from=base_build /usr/lib/dbus-1.0 /usr/lib/dbus-1.0
COPY --from=base_build /usr/lib/pkgconfig /usr/lib/pkgconfig
RUN set -ex && cd /usr/lib && \
        ln -sf libdbus-1.so.3.19.12  libdbus-1.so && \
        ln -sf libdbus-1.so.3.19.12  libdbus-1.so.3 && \
        ln -sf libdbus-glib-1.so.2.2.2  libdbus-glib-1.so.2 && \
        ln -sf libdbus-glib-1.so.2.2.2  libdbus-glib-1.so && \
        ldconfig && \
        ls -lh /usr/lib && \
        ls -lh /usr/lib64

#merge files last
FROM scratch
COPY --from=release_build / /
CMD [ "/bin/bash" ]





