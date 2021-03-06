# Ubuntu base
#
# Updates packages, install python, add some fixes and configure rsyslog

FROM ubuntu:trusty-20160624
MAINTAINER Jirayut Nimsaeng <w [at] winginfotech.net>

# 1) Configure timezone, locale and fix environment
# 2) Make sure the package repository is up to date
# 3) Install python since it will be requirement for Ansible Docker connection
ARG APT_CACHER_NG
ENV LANG=en_US.UTF-8 LANGUAGE=en_US.UTF-8 LC_ALL=en_US.UTF-8 HOME=/root \
    DEBIAN_FRONTEND=noninteractive FAKE_CHROOT=1 INITRD=No
RUN echo "Asia/Bangkok" > /etc/timezone && \
    locale-gen en_US.UTF-8 && \
    dpkg-reconfigure -f noninteractive locales && \
    dpkg-reconfigure -f noninteractive tzdata && \
    update-locale LANG=en_US.UTF-8 && \
    [ -n "$APT_CACHER_NG" ] && \
      echo "Acquire::http::Proxy \"$APT_CACHER_NG\";" \
      > /etc/apt/apt.conf.d/11proxy || true; \
    apt-get update && \
    apt-get -y dist-upgrade && \
    apt-get -y install python && \
    apt-get -y autoremove && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /etc/apt/apt.conf.d/11proxy

# 1) Some fixes from https://journal.paul.querna.org/articles/2013/10/15/
#    docker-ubuntu-on-rackspace/
# 2) Config logrotate and rsyslog
# PS) For ubuntu:14.04 image the rsyslog rotate needs to change postrotate
#     command from upstart to init.d
COPY build-files /build-files
RUN mv /usr/bin/ischroot /usr/bin/ischroot.original && \
    mv /build-files/ischroot /usr/bin/ischroot && \
    sed -i '/^weekly/a \dateext\ndateformat .%Y%m%d' \
      /etc/logrotate.conf && \
    sed -i 's/\trotate .*/\trotate 365/g' \
      /etc/logrotate.d/rsyslog && \
    sed -i 's/reload rsyslog/service rsyslog rotate/g' \
      /etc/logrotate.d/rsyslog && \
    sed -i 's/--pidfile \$PIDFILE --exec \$DAEMON$/--pidfile \$PIDFILE/g' \
      /etc/init.d/rsyslog && \
    sed -i '/^\($ModLoad imklog\|$KLogPermitNonKernelFacility\)/ s/^#*/#/' \
      /etc/rsyslog.conf && \
    sed -i '/^#\(cron\|mail\|user\)/ s/^#*//' \
      /etc/rsyslog.d/50-default.conf && \
    sed -i '/^\(news\|daemon\|\tnews\|\t\*\.=\)/ s/^#*/#/' \
      /etc/rsyslog.d/50-default.conf && \
    service rsyslog start && \
    service rsyslog stop && \
    rm -rf /build-files
