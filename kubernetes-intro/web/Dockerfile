# FROM alexdo/python-2.4-alpine
FROM docker.io/bitnami/nginx:1.21
ENV NGINX_ENABLE_CUSTOM_PORTS yes
ENV NGINX_HTTP_PORT_NUMBER 8000
EXPOSE 8000
COPY ./homework.html /app/

