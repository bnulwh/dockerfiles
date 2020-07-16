FROM kong:0.14.1-centos
FROM 5ibnu/centos:7.4
LABEL maintainer="Kong Core Team <team-core@konghq.com>"

ENV KONG_VERSION=0.14.1

RUN set -ex && \
          yum install -y wget https://bintray.com/kong/kong-community-edition-rpm/download_file?file_path=centos/7/kong-community-edition-$KONG_VERSION.el7.noarch.rpm &&    \
          yum clean all

COPY --from=0 /docker-entrypoint.sh  /docker-entrypoint.sh 

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 8000 8001 8443 8444

STOPSIGNAL SIGTERM

CMD ["kong" "docker-start"]
