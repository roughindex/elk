FROM registry.access.redhat.com/ubi7/ubi-minimal:latest

RUN printf "[elasticsearch]\nname=Elasticsearch repository for 7.x packages\nbaseurl=https://artifacts.elastic.co/packages/7.x/yum\ngpgcheck=1\ngpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch\nenabled=0\nautorefresh=1" > /etc/yum.repos.d/elasticsearch.repo && \
    rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch && \
    microdnf install shadow-utils && \
    microdnf install --enablerepo=elasticsearch logstash metricbeat && \
    microdnf remove shadow-utils && \
    microdnf clean all && \
    rpm --rebuilddb && \
    metricbeat modules enable docker && \
    printf "output.logstash:\n    hosts: ["logstash:5044"]" >> /etc/metricbeat/metricbeat.yml && \
    mkdir /config && \
    chown -R logstash /config /usr/share/logstash && \
    rm -rf /usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/rubyzip-1.3.0/test

COPY logstash*.conf /config/

EXPOSE 9650
EXPOSE 5044

USER logstash

CMD ["/usr/share/logstash/bin/logstash", "--path.config", "/config", "--pipeline.workers", "1", "--pipeline.ordered", "false", "--log.level", "info", "--http.host", "0.0.0.0", "--http.port", "9650"]
