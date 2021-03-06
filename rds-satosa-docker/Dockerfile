FROM debian:jessie

RUN apt-get update && apt-get install -y --no-install-recommends \
        git \
        python3-dev \
        build-essential \
        libffi-dev \
        libssl-dev \
        xmlsec1 \
        libyaml-dev \
        wget \
        ca-certificates

# Use the head of the master branch of the official SATOSA repository.
ENV SATOSA_SRC_URL=https://github.com/IdentityPython/SATOSA/archive/master.tar.gz

# Use the head of the master branch of the official SATOSA microservices repository.
ENV SATOSA_MICROSERVICES_SRC_URL=https://github.com/IdentityPython/satosa_microservices/archive/master.tar.gz

WORKDIR /tmp

# Download the latest pip and use it to install SATOSA and then
# upgrade pySAML.
RUN wget https://bootstrap.pypa.io/get-pip.py \
    && python3 get-pip.py \
    && rm -f get-pip.py \
    && wget -O satosa.tar.gz ${SATOSA_SRC_URL} \
    && mkdir -p /tmp/satosa \
    && tar -zxf satosa.tar.gz -C /tmp/satosa --strip-components=1 \
    && rm -f satosa.tar.gz \
    && pip install ./satosa \
    && pip install ldap3 \
    && pip install --upgrade pysaml2 \
    && rm -rf satosa

# Until a rew release of pysaml2 is available with better MDQ
# support and the ability to support signed signed responses or
# assertions we need to build a newer version from the repository.
# Until the necessary pull requests are accepted we need to merge
# them directly.
RUN git clone https://github.com/rohe/pysaml2.git \
    && cd pysaml2 \
    && git -c "user.name=nobody" -c "user.email=nobody@localhost" pull --rebase origin pull/483/head \
    && git -c "user.name=nobody" -c "user.email=nobody@localhost" pull --rebase origin pull/485/head \
    && pip install --upgrade ./ \
    && cd .. \
    && rm -rf pysaml2

# Download the SATOSA microservices.
RUN wget -O satosa_microservices.tar.gz ${SATOSA_MICROSERVICES_SRC_URL} \
    && mkdir -p /opt/satosa_microservices \
    && tar -zxf satosa_microservices.tar.gz -C /opt/satosa_microservices --strip-components=1 \
    && rm -f satosa_microservices.tar.gz

# Set PYTHONPATH appropriately for the SATOSA microservices.
ENV PYTHONPATH=/opt/satosa_microservices/src/satosa/micro_services

COPY start.sh /usr/local/sbin/satosa-start.sh
COPY proxy_conf.yaml /etc/satosa/proxy_conf.yaml
COPY internal_attributes.yaml /etc/satosa/internal_attributes.yaml
COPY attributemaps /etc/satosa/attributemaps
COPY plugins /etc/satosa/plugins
COPY rds-metadata-signer.crt /etc/satosa/rds-metadata-signer.crt

ENTRYPOINT ["/usr/local/sbin/satosa-start.sh"]
