FROM ubuntu:14.04
MAINTAINER Till Toenshoff <@ttoenshoff>

ARG DEBIAN_FRONTEND=noninteractive

ADD krb5.conf /etc/krb5.conf

RUN echo package heimdal/realm string REALM | debconf-set-selections
RUN apt-get -y update && \
    apt-get -y install heimdal-kdc libsasl2-modules-gssapi-heimdal

EXPOSE	88

# Create keytab folder.
RUN mkdir /etc/docker-kdc

# Add kerberos principal/s.
PRINCIPALS

# Export keytab.
EXPORT_KEYTAB

# KDC daemon startup.
ENTRYPOINT ["/usr/lib/heimdal-servers/kdc", "--config-file=/etc/heimdal-kdc/kdc.conf", "-P 88"]
