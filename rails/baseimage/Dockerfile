FROM phusion/baseimage:master

CMD ["/sbin/my_init"]

# PREPARE
ADD . /tmp/build

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential git sudo tzdata gnupg2 nginx \
    libpq-dev

RUN cp /tmp/build/configs/dpkg-nodocs.conf /etc/dpkg/dpkg.cfg.d/01_nodoc && \
    cp /tmp/build/configs/dpkg-only-english-locale.conf /etc/dpkg/dpkg.cfg.d/01_only_english_locale && \
    echo Asia/Shanghai > /etc/timezone && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# INSTALL RUBY
RUN gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB && \
    curl -sSL https://get.rvm.io | bash -s stable && \
    /bin/bash -l -c 'source /etc/profile.d/rvm.sh' && \
    /bin/bash -l -c 'rvm requirements' && \
    /bin/bash -l -c 'rvm install 3.0.0' && \
    /bin/bash -l -c 'rvm use 3.0.0 --default' && \
    /bin/bash -l -c 'rvm rubygems current' && \
    echo "source /etc/profile.d/rvm.sh" >> /etc/profile && \
    mkdir -p /var/run/puma && \
    chown -R 9999:9999 /var/run/puma

# INSTALL NODEJS
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - && \
    apt-get install -y nodejs && \
    npm i -g yarn

# ADD USER
RUN addgroup --gid 9999 app && \
    adduser --uid 9999 --gid 9999 --disabled-password --gecos "Application" app && \
    usermod -L app && \
    passwd -u app && \
    usermod -a -G docker_env app && \
    usermod -a -G rvm app

# CONFIG SSH
RUN rm -rf /etc/service/sshd/down && \
    sed -i "s/#ClientAliveInterval 0/ClientAliveInterval 60/g" /etc/ssh/sshd_config && \
    cp -R /tmp/build/configs/ssh /home/app/.ssh && \
    chmod 700 /home/app/.ssh && \
    chmod 644 /home/app/.ssh/authorized_keys && \
    chmod 600 /home/app/.ssh/id_rsa.pub && \
    chmod 600 /home/app/.ssh/id_rsa && \
    chown -R 9999:9999 /home/app/.ssh

# CONFIG NGINX
RUN mkdir /etc/service/nginx && \
    mkdir /etc/service/nginx-log-forwarder && \
    touch /etc/service/nginx/down && \
    touch /etc/service/nginx-log-forwarder/down && \
    cp /tmp/build/runit/nginx /etc/service/nginx/run && \
    cp /tmp/build/runit/nginx-log-forwarder /etc/service/nginx-log-forwarder/run && \
    chmod +x /etc/service/nginx/run && \
    chmod +x /etc/service/nginx-log-forwarder/run && \
    cp /tmp/build/configs/nginx/nginx_default.conf /etc/nginx/sites-available/default

EXPOSE 80 443

RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
