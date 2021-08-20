FROM registry.access.redhat.com/ubi7/ubi-minimal:latest

RUN printf "[elasticsearch]\nname=Elasticsearch repository for 7.x packages\nbaseurl=https://artifacts.elastic.co/packages/7.x/yum\ngpgcheck=1\ngpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch\nenabled=0\nautorefresh=1" > /etc/yum.repos.d/elasticsearch.repo && \
    microdnf update && \
    # shadow-utils are temporarily installed to provide `useradd`
    microdnf install shadow-utils && \
    rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch && \
    microdnf install --enablerepo=elasticsearch kibana filebeat metricbeat && \
    useradd -u 1001 -r -g 0 -d /usr/share/kibana -s /sbin/nologin -c "Kibana User" default && \
    microdnf remove shadow-utils && \
    microdnf clean all && \
    rpm --rebuilddb 

ENV PATH /usr/share/kibana/bin:$PATH
RUN chown -R 1001:0 /usr/share/kibana && \
    chown -R 1001:0 /var/log/kibana && \
    chown -R 1001:0 /var/lib/kibana && \
    chown -R 1001:0 /etc/sysconfig/kibana && \
    chown -R 1001:0 /etc/kibana && \
    sed -i "/elasticsearch\.hosts/a elasticsearch.hosts: [\"http://elastic:9200\"]" /etc/kibana/kibana.yml && \
    sed -i '/server\.host/a server.host: "0.0.0.0"' /etc/kibana/kibana.yml && \
    sed -i "s/localhost:9200/elastic:9200/" /etc/filebeat/filebeat.yml && \
    sed -i "s/localhost:9200/elastic:9200/" /etc/metricbeat/metricbeat.yml && \
    chown -R 1001:0 /var/log && \
    chown -R 1001:0 /var/lib && \
    chown -R 1001:0 /etc/metricbeat && \
    chown -R 1001:0 /etc/filebeat

WORKDIR /usr/share/kibana

EXPOSE 5601
USER 1001

#The start script waits for ElasticSearch to start, sets up Beats schemas, then starts Kibana
COPY start .

CMD ./start