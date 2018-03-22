FROM postgres:10

RUN export DEBIAN_FRONTEND=noninteractive \
    && echo 'APT::Install-Recommends "0";\nAPT::Install-Suggests "0";' > /etc/apt/apt.conf.d/01norecommend \
    && apt-get update -y \
    && apt-get upgrade -y \
    && apt-get install -y git curl jq python-psycopg2 python-yaml python-requests python-six python-pysocks \
        python-dateutil python-pip python-prettytable python-wheel python-psutil python locales \
        openssh-client gcc build-essential cmake \

    ## Make sure we have a en_US.UTF-8 locale available
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8 \

    && pip install setuptools pip --upgrade \

    # build nss_wrapper
    && git clone git://git.samba.org/nss_wrapper.git /tmp/nss_wrapper \
    && mkdir /tmp/nss_wrapper/build \
    && cd /tmp/nss_wrapper/build \
    && cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DLIB_SUFFIX=64 .. \
    && make \
    && make install \
    && rm -rf /tmp/nss_wrapper

EXPOSE 5432 8008
ENV LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    NSS_WRAPPER_PASSWD=/tmp/passwd \
    NSS_WRAPPER_GROUP=/tmp/group

COPY patroni-1.4.3.tar.gz entrypoint.sh nss-wrap.sh /
RUN pip install /patroni-1.4.3.tar.gz[kubernetes] \

    # Setup nss wrapper files
    && for path in "$NSS_WRAPPER_PASSWD" "$NSS_WRAPPER_GROUP"; do \
    touch $path && chmod 666 $path ; done \

    # Clean up
    && apt-get remove -y git python-pip python-setuptools git gcc build-essential cmake \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* /root/.cache /patroni-1.4.3.tar.gz

ENTRYPOINT ["/nss-wrap.sh"]
CMD ["/entrypoint.sh"]
