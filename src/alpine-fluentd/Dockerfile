FROM stonevil/alpine-ruby
MAINTAINER stonevil@gmail.com

ENV FLUENTD_VERSION=${FLUENTD_VERSION:-0.14.0}

LABEL com.stonevil.fluentd.version=${FLUENTD_VERSION} \
  com.stonevil.k8s.ready=true

RUN ${APK_INSTALL_CMD} build-base ruby-dev ruby-io-console ruby-irb tzdata && \
  ${GEM_UPGRADE_CMD} && \
  ${GEM_INSTALL_CMD} fluentd -v $FLUENTD_VERSION && \
  ${GEM_INSTALL_CMD} fluent-plugin-elasticsearch fluent-plugin-kubernetes_metadata_filter && \
  adduser -D -H -h /data/fluentd -s /bin/bash fluentd && \
  ${APK_REMOVE_CMD} build-base ruby-dev && \
  ${APK_CACHE_RM}

ADD _rootfs /
ONBUILD ADD _rootfs_vendor /

EXPOSE 24224

VOLUME ["/data/fluentd"]

ENV FLUENTD_K8S_ELASTIC_INIT=${FLUENTD_K8S_ELASTIC_INIT:-false}

ENV ELASTIC_HOST=${ELASTIC_HOST:-127.0.0.1}
ENV ELASTIC_PORT=${ELASTIC_PORT:-9200}
ENV K8S_URL=${K8S_URL:-http://127.0.0.1:8080}

CMD mkdir -p /data/fluentd/logs /etc/fluentd/plugins; chown -fR fluentd:fluentd /data/fluentd /etc/fluentd/plugins && \
  (ulimit -n 65536; if [ `ulimit -n` -lt "65536" ] ; then (echo "Fluentd recommends 64k open files per process. you have "`ulimit -n` && echo "the docker deamon should be run with increased file descriptors to increase those available in the container" && echo " try \`ulimit -n 65536\`"); fi) && \
  if ${FLUENTD_K8S_ELASTIC_INIT}; then \
  (sed -i -e 's|_ELASTICHOST_|'"${ELASTIC_HOST}"'|g' -e 's|_ELASTICPORT_|'"${ELASTIC_PORT}"'|g' -e 's|_K8SURL_|'"${K8S_URL}"'|g' /etc/fluentd/fluent-k8s-elastic.conf && gosu fluentd fluentd --config /etc/fluentd/fluent-k8s-elastic.conf --plugin /etc/fluentd/plugins); else \
  (gosu fluentd fluentd --config /etc/fluentd/fluent.conf --plugin /etc/fluentd/plugins); fi
