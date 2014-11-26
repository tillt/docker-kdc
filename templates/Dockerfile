FROM ubuntu:14.04

RUN apt-get -y update

ADD krb5.conf /etc/krb5.conf

RUN apt-get -y install heimdal-kdc
RUN apt-get -y install libsasl2-modules-gssapi-heimdal

EXPOSE	88

# Create keytab folder.
RUN mkdir /etc/docker-kdc

# Add a kerberos principal.
RUN /bin/echo -e '\n\n\n\n\n\nPASSWORD\nPASSWORD\n' | kadmin -l add PRINCIPAL/HOST_NAME.DOMAIN_NAME

# Export keytab.
RUN kadmin -l ext_keytab -k /etc/docker-kdc/krb5.keytab PRINCIPAL/HOST_NAME.DOMAIN_NAME

# KDC daemon startup.
ENTRYPOINT ["/usr/lib/heimdal-servers/kdc", "--config-file=/etc/heimdal-kdc/kdc.conf", "-P 88"]
