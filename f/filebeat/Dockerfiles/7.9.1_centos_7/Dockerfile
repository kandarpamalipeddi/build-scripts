# NOTE: This is based on the original work at https://github.com/elastic/dockerfiles/

FROM golang:1.13 as builder

RUN echo $GOPATH && mkdir -p $GOPATH/src/github.com/elastic && \
    cd $GOPATH/src/github.com/elastic && \
    git clone https://github.com/elastic/beats.git && \
    cd beats && git checkout v7.9.1 && \
    wget https://raw.githubusercontent.com/ppc64le/build-scripts/master/f/filebeat/filebeat_7.9.1.patch
    git apply filebeat_7.9.1.patch && \
    cd filebeat && \
    make 

FROM centos:7

RUN yum -y --setopt=tsflags=nodocs update && \
    yum clean all

LABEL org.label-schema.build-date="2020-09-01T19:58:34Z" \
	org.label-schema.schema-version="1.0" \
	org.label-schema.vendor="Elastic" \
	org.label-schema.license="Elastic License" \
	org.label-schema.name="filebeat" \
	org.label-schema.version="7.9.1" \
	org.label-schema.url="https://www.elastic.co/products/beats/filebeat" \
	org.label-schema.vcs-url="github.com/elastic/beats/v7" \
	org.label-schema.vcs-ref="ad823eca4cc74439d1a44351c596c12ab51054f5" \
	license="Elastic License description=Filebeat sends log files to Logstash or directly to Elasticsearch."

ENV ELASTIC_CONTAINER "true"
ENV PATH=/usr/share/filebeat:$PATH

COPY --from=builder /go/src/github.com/elastic/beats/filebeat/filebeat /usr/share/filebeat/
COPY --from=builder /go/src/github.com/elastic/beats/filebeat/filebeat.yml /usr/share/filebeat/
COPY --from=builder /go/src/github.com/elastic/beats/filebeat/modules.d /usr/share/filebeat/modules.d
COPY --from=builder /go/src/github.com/elastic/beats/filebeat/module /usr/share/filebeat/module
COPY docker-entrypoint /usr/local/bin/docker-entrypoint
RUN chmod 755 /usr/local/bin/docker-entrypoint

RUN groupadd --gid 1000 filebeat

RUN mkdir  /usr/share/filebeat/data /usr/share/filebeat/logs && \
    chown -R root:root /usr/share/filebeat && \
    find /usr/share/filebeat -type d -exec chmod 0750 {} \; && \
    find /usr/share/filebeat -type f -exec chmod 0640 {} \; && \
    chmod 0750 /usr/share/filebeat/filebeat && \
    chmod 0770 /usr/share/filebeat/modules.d && \   
    chmod 0770 /usr/share/filebeat/data /usr/share/filebeat/logs

RUN useradd -M --uid 1000 --gid 1000 --groups 0 --home /usr/share/filebeat filebeat

USER filebeat

EXPOSE 5044

WORKDIR /usr/share/filebeat
ENTRYPOINT ["/usr/local/bin/docker-entrypoint"]
CMD ["-environment", "container"]
