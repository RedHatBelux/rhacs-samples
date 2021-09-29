# Dockerfile to build our custom image based on httpd which will contain netcat
FROM httpd:2.4
COPY ./public-html/ /usr/local/apache2/htdocs/
COPY ./conf/ /usr/local/apache2/conf/
RUN apt update
RUN apt -y install netcat
RUN chown -R daemon:daemon /usr/local/apache2/logs
RUN chmod 777 /usr/local/apache2/logs