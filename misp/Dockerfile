FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive  

RUN apt-get update && \
    apt-get dist-upgrade -y && apt-get upgrade && apt-get autoremove -y && apt-get clean && \
    apt-get install -y software-properties-common && \
    apt-get install -y postfix && \
    apt-get install -y mysql-client curl gcc git gnupg-agent \
    make openssl redis-server sudo vim zip locales wget iproute2 supervisor cron iputils-ping

# install python
RUN add-apt-repository ppa:deadsnakes/ppa
RUN apt-get update && apt-get -y install python3.9 python3-pip
RUN pip3 install --upgrade pip
RUN pip install pymisp


# set language
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8

# add user misp
RUN useradd misp && usermod -aG sudo misp

# Install script
COPY --chown=misp:misp ./installation/INSTALL_NODB.sh* ./
RUN chmod +x INSTALL_NODB.sh
RUN echo "ALL ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers


USER misp

RUN bash INSTALL_NODB.sh -A -u
RUN touch /var/tmp/first-configuration

USER root

RUN pip3 list -o | cut -f1 -d' ' | tr " " "\n" | awk '{if(NR>=3)print}' | cut -d' ' -f1 | xargs -n1 pip3 install -U ; exit 0 #Hack error code

# Supervisord Setup
RUN ( \
    echo '[supervisord]'; \
    echo 'nodaemon = true'; \
    echo ''; \
    echo '[program:postfix]'; \
    echo 'process_name = master'; \
    echo 'directory = /etc/postfix'; \
    echo 'command = /usr/sbin/postfix -c /etc/postfix start'; \
    echo 'startsecs = 0'; \
    echo 'autorestart = false'; \
    echo ''; \
    echo '[program:redis-server]'; \
    echo 'command=redis-server /etc/redis/redis.conf'; \
    echo ''; \
    echo '[program:apache2]'; \
    echo 'command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2 -D FOREGROUND"'; \
    echo ''; \
    echo '[program:resque]'; \
    echo 'command=/bin/bash /var/www/MISP/app/Console/worker/start.sh'; \
    echo 'user = www-data'; \
    echo 'startsecs = 0'; \
    echo 'autorestart = false'; \
    echo ''; \
    echo '[program:misp-modules]'; \
    echo 'command=/bin/bash -c "/var/www/MISP/venv/bin/misp-modules -l 127.0.0.1 -s"'; \
    echo 'user = www-data'; \
    echo 'startsecs = 0'; \
    echo 'autorestart = false'; \
    echo ''; \
    echo '[program:configuration]'; \
    echo 'command=/bin/bash -c "python3 /configuration.py"'; \
    echo 'user = www-data'; \
    echo 'startsecs = 20'; \
    echo 'autorestart = false'; \
    ) >> /etc/supervisor/conf.d/supervisord.conf

# Add run script
# Trigger to perform first boot operations
ADD installation/run.sh /run.sh
RUN mv /etc/apache2/sites-available/misp-ssl.conf /etc/apache2/sites-available/misp-ssl.conf.bak
ADD installation/misp-ssl.conf /etc/apache2/sites-available/misp-ssl.conf
RUN chmod 0755 /run.sh && touch /.firstboot.tmp
# Make a backup of /var/www/MISP to restore it to the local moint point at first boot
WORKDIR /var/www/MISP
RUN tar czpf /root/MISP.tgz .

# Install MISP Modules
#RUN sudo apt-get install python3-dev python3-pip libpq5 libjpeg-dev tesseract-ocr libpoppler-cpp-dev imagemagick virtualenv libopencv-dev zbar-tools libzbar0 libzbar-dev libfuzzy-dev build-essential -y
#RUN sudo -u www-data virtualenv -p python3 /var/www/MISP/venv
#RUN cd /usr/local/src/
#RUN sudo chown -R www-data: .
#RUN cd /var/www/MISP/venv/
#RUN sudo pip install -I -r requirements.txt
#RUN sudo pip install .


# Configuration folder
ADD configuration/ /
RUN chmod 0755 /configuration.py

VOLUME /var/www/MISP
EXPOSE 80
ENTRYPOINT ["/run.sh"]
