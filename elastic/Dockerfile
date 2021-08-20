FROM registry.access.redhat.com/ubi7/ubi-minimal:latest

RUN printf "[elasticsearch]\nname=Elasticsearch repository for 7.x packages\nbaseurl=https://artifacts.elastic.co/packages/7.x/yum\ngpgcheck=1\ngpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch\nenabled=0\nautorefresh=1" > /etc/yum.repos.d/elasticsearch.repo && \
    microdnf update && \
    # shadow-utils are temporarily installed to provide `useradd`
    microdnf install shadow-utils && \
    # install elasticsearch and beats from elastic.co repos
    rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch && \
    microdnf install --enablerepo=elasticsearch elasticsearch metricbeat && \
    # default user added after installing elastic search in order to set home folder created during install
    useradd -u 1001 -r -g 0 -d /usr/share/elasticsearch -s /sbin/nologin -c "ElasticSearch User" default && \
    # cleanup - remove shadow-utils and clear all install caches 
    microdnf remove shadow-utils && \
    microdnf clean all && \
    rpm --rebuilddb && \
    # Configure metricbeat to watch performance statistics and push logs to logstash
    metricbeat modules enable system && \
    printf "output.logstash:\n    hosts: ["logstash:5044"]" >> /etc/metricbeat/metricbeat.yml && \
    sed -i 's/#name:/name: "elastic"/' /etc/metricbeat/metricbeat.yml  && \
    sed -i 's/output.elasticsearch/#output.elasticsearch/' /etc/metricbeat/metricbeat.yml  && \
    sed -i 's/hosts: \["localhost:9200"\]/#hosts: ["localhost:9200"]/' /etc/metricbeat/metricbeat.yml  && \
    echo "#\! /bin/sh" > /metrics && \
    echo "sleep 120" >> /metrics && \
    echo "/usr/share/metricbeat/bin/metricbeat -c /etc/metricbeat/metricbeat.yml --path.home /usr/share/metricbeat --path.config /etc/metricbeat --path.data /var/lib/metricbeat --path.logs /var/log/metricbeat" >> /metrics && \
    chmod +x /metrics && \
    chown 1001:0 /metrics && \
    # Create start command. Metric capture in the background. ElasticSeach as the foreground task
    echo "#\! /bin/sh" > /start && \
    echo "/metrics &" >> /start && \
    echo "/usr/share/elasticsearch/bin/elasticsearch" >> /start && \
    chmod +x /start && \
    chown 1001:0 /start && \
    # Set elasticsearch and beats permissions and volumes
    chown -R 1001:0 /var/log/metricbeat && \
    chown -R 1001:0 /var/lib/metricbeat && \
    chown -R 1001:0 /etc/metricbeat && \
    # elasticsearch data is persistent outside the container
    mkdir -p /vol/log/elasticsearch && \
    mkdir -p /vol/lib/elasticsearch && \
    chown -R 1001:0 /vol/log/elasticsearch && \
    chown -R 1001:0 /vol/lib/elasticsearch && \
    chown -R 1001:0 /etc/sysconfig/elasticsearch && \
    chown -R 1001:0 /etc/elasticsearch && \
    # Configure elasticsearch
    sed -i '/network\.host/a network.host: 0.0.0.0' /etc/elasticsearch/elasticsearch.yml && \
    sed -i 's/var/vol/g' /etc/elasticsearch/elasticsearch.yml && \
    sed -i 's/var/vol/g' /etc/elasticsearch/jvm.options && \
    sed -i '/0\.0\.0\.0/a transport.host: localhost' /etc/elasticsearch/elasticsearch.yml


VOLUME /vol/log/elasticsearch
VOLUME /vol/lib/elasticsearch

ENV PATH /usr/share/elasticsearch/bin:$PATH
WORKDIR /usr/share/elasticsearch

EXPOSE 9200
USER 1001
CMD /start