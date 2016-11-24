docker-kdc
==========

Docker container generator for a Heimdal Kerberos 5 KDC.

The intension here is to ease the first steps with Kerberos while also allowing a customized, automated setup for development or test integration. Usable on plain Linux as well as on OSX.

---

#Dependencies
- Docker
- jq 1.4

####Linux specific dependency
- Heimdal Kerberos 5

####OSX specific dependency
- boot2docker

---

#Usage

###Check your configuration
The default configuration is likely to be fine for your first steps, validate it using the `config` command.
```
./kdc config
```

You will receive a list of relevant configuration information. The defaults are derived from your hosts' configuration to allow for a quick test setup.

**Example output: `./kdc config`**
```
System
  fqdn:      hostname.domain.name
KDC
  nat:       127.0.0.1
  port:      48088
Kerberos
  domain:    domain.name
  realm:     DOMAIN.NAME
  principal: tillt/hostname.domain.name@DOMAIN.NAME, password: matilda
```


###Build the docker image
```
./kdc build
```

This will render the image which is based on plain ubuntu 14.04. Additionally the packages `heimdal-kdc` as well as `libsasl2-modules-gssapi-heimdal` are installed. The latter is useful only if you extend this container image by further applications making use of Kerberos authentication via SASL2's GSSAPI.


###Run the container
```
./kdc start
```

On OSX, this step starts by setting up the VM (via boot2docker). Then, on all host systems, the container is started in detached mode, allowing you to keep on working with this shell without having to fork another process. The container name is directly derived from the hostname supplied via the configuration (see [Configuration](#configuration)).

###Watch the KDC server log file
```
docker exec -it kdc tail -f /var/log/heimdal-kdc.log
```

###Run a quick test
```
./kdc test
```
On OSX, this first checks if the VM is active. Then, on all hosts systems, a network connection to the KDC is attempted.

###Prepare the environment
```
$(./kdc shellinit)
```

A Kerberos client needs access to a configuration file. To prevent having to edit the system wide configuration file (`/etc/krb5.conf`) a local, minimal version is rendered and supplied once the container has gotten started. Additionally, the keytab also gets exported and hence needs to be accessible for clients making use of password-less authentication. To make use of the files, environment variables that are interpreted by Kerberos clients are prepared.

###Render a ticket supplying the principal password
```
kinit tillt/hostname.example.com@EXAMPLE.COM
```

Password: `matilda`

####Check the ticket
```
klist
```

On OSX you could also use the Ticket Viewer to check the details of the issued ticket (`open "/System/Library/CoreServices/Ticket Viewer.app"`).

**Example output: `klist`**

```
Credentials cache: API:42926CE1-63E2-4C66-B2D7-00B2F198182F
        Principal: tillt/hostname.example.com@EXAMPLE.COM

  Issued                Expires               Principal
Nov 26 11:06:25 2014  Nov 26 21:06:25 2014  krbtgt/EXAMPLE.COM@EXAMPLE.COM
```

####Remove the ticket
```
kdestroy
```

###Check the content of the keytab
```
ktutil --keytab=krb5.keytab list
```

**Example output: `ktutil --keytab=krb5.keytab list`**
```
krb5.keytab:

Vno  Type                     Principal                              Aliases
  1  aes256-cts-hmac-sha1-96  tillt/hostname.example.com@EXAMPLE.COM
  1  des3-cbc-sha1            tillt/hostname.example.com@EXAMPLE.COM
  1  arcfour-hmac-md5         tillt/hostname.example.com@EXAMPLE.COM
```

###Render a ticket using keytab based authentication
```
kinit -kt krb5.keytab tillt/hostname.example.com@EXAMPLE.COM
```

###Check the ticket
```
klist
```

[...]


###Stop the container
```
./kdc stop
```

This will stop the KDC server, stop and remove the container and additionally remove the temporary keytab and configuration files.


###Customize your configuration
You may use environment variables and/or a JSON configuration file for customizing the setup. The default filename for the JSON file is `kdc.json` but may be configured by the environment variable  KDC_CONFIG.

The default configuration is most likely good enough for your first experiments.

####Kerberos principal
| env. variable | config node | default   |
|---------------|-------------|-----------|
| KDC_PRINCIPAL | id          | `tillt`   |

**Note**: using a configuration file allows setting up multiple principals (via **principals[ ].id**).

####Kerberos password
| env. variable | config node | default   |
|---------------|-------------|-----------|
| KDC_PASSWORD  | password    | `matilda` |

**Note**: using a configuration file allows setting up multiple passwords (via **principals[ ].password**).

####Kerberos client
| env. variable | config node | default                 |
|---------------|-------------|-------------------------|
| KDC_CLIENT    | n/a         | oufput of `hostname -s` |

**Note**: when no principals are defined via configuration file, KDC_CLIENT is used to create a full service principal (schema: KDC_PRINCIPAL **/** KDC_CLIENT **.** KDC_DOMAIN_NAME **@** KDC_REALM_NAME ).

####KDC hostname
| env. variable | config node | default   |
|---------------|-------------|-----------|
| KDC_HOST_NAME | n/a         | `kdc`     |

####External KDC IP
| env. variable | config node | default     |
|---------------|-------------|-------------|
| KDC_NATHOST   | nat         | `127.0.0.1` |

**Note**: this value gets overridden by the kdc script on OSX to allow for connecting to the boot2docker VM. You shouldn't really need to override this in any case.

####External KDC port
| env. variable | config node | default   |
|---------------|-------------|-----------|
| KDC_PORT      | port        | `48088`   |

####Kerberos domain name
| env. variable   | config node | default                                  |
|-----------------|-------------|------------------------------------------|
| KDC_DOMAIN_NAME | domain      | hostname cut off output of `hostname -f` |

####Kerberos realm name
| env. variable  | config node | default                              |
|----------------|-------------|--------------------------------------|
| KDC_REALM_NAME | realm       | capitalized value of KDC_DOMAIN_NAME |

**Note**: it is common practice to simply use the domain-name but all capitalized for this.

####Configuration filename
| env. variable  | config node | default       |
|----------------|-------------|---------------|
| KDC_CONFIG     | n/a         | `kdc.json`    |

**templates/kdc.json**
```
{
  "principals": [
    {
      "id": "tillt/host.example.com@EXAMPLE.COM",
      "password": "herbert"
    },
    {
      "id": "tillt@EXAMPLE.COM",
      "password": "herbert"
    }
  ],
  "domain": "example.com",
  "realm": "EXAMPLE.COM",
  "ip": "127.0.0.1",
  "port": 48088
}
```

---

#Reference

```
./kdc start|stop|build|clean|config|shellinit
```

##build
Builds the docker image.

##start
Starts the container in detached mode while also producing a Kerberos configuration file (`krb5.conf`) as well as a Kerberos keytab (`krb5.keytab`) locally. 

Note that the keytab is only readable/usable by the current user unless you change its access rights which is not recommended for production environments.

##stop
Stops the container and deletes `krb5.conf` as well as `krb5.keytab`.

##clean
Removes the docker image.

##config
Shows relevant configuration information.

##test
Checks if the KDC is reachable and accepting connections.

##shellinit
Renders the environment variables needed for using the KDC. KRB5_CONFIG points towards the temporary configuration file. KRB5_KTNAME points towards the temporary keytab file.

---

#TODO

- strip down base image to squeeze out some space
- refactor code into something less convoluted
- allow for an admin server, not just the KDC


---

#Credits

This script was inspired by some work of a co-worker of mine, Matthias Veit. Matthias did the hard work of finding out how to properly route docker ports on boot2docker hosts.

---

#Author

* [Till Toenshoff](https://github.com/tillt) ([@ttoenshoff](https://twitter.com/ttoenshoff))
