# fluentd/Dockerfile
FROM fluent/fluentd:v1.7-1
USER root
COPY ./fluent.conf /fluentd/etc
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-document", "--version", "3.5.2"]
USER fluent