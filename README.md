docker-kdc
==========

Docker container for Kerberos KDC.

Usable on plain Linux as well as on OSX (via boot2docker).


#Usage

`./kdc build`

`./kdc start`

`$(./kdc shellinit)`

Render a ticket for `tillt`:

`kinit tillt/kdc.example.com@EXAMPLE.COM`

Password: `matilda`

Check the ticket:

`klist`

[...]

`./kdc stop`

#Reference

./kdc start|stop|build|clean|shellinit

##build

Builds the docker image.

##start

Starts the container in detached mode while also producing a kerberos configuration file (`krb5.conf`) locally.

##stop

Stops the container and deletes `krb5.conf`.

##clean

Removes the docker image.

##shellinit

Renders the environment variables needed for using the KDC.
