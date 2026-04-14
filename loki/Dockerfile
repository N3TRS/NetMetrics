FROM grafana/loki:latest
COPY loki/loki-config.yml /etc/loki/local-config.yaml
CMD ["-config.file=/etc/loki/local-config.yaml"]