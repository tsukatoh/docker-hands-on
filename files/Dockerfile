FROM ubuntu:16.04

RUN apt-get update && \
    apt-get install -y nginx

COPY ./html/* /var/www/html/

EXPOSE 80

CMD ["/usr/sbin/nginx", "-g", "daemon off;"]