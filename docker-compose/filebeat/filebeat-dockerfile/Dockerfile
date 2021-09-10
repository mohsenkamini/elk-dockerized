FROM elastic/filebeat:7.14.0
USER root
RUN usermod -a -G adm filebeat
VOLUME /var/lib/docker/containers/ /var/mydocker.logs/
RUN chmod 750 /var/mydocker.logs/ -R
USER filebeat
