sa-container-bootstrap
======================

[![Build Status](https://travis-ci.org/softasap/sa-container-bootstrap.svg?branch=master)](https://travis-ci.org/softasap/sa-container-bootstrap)
[![License: MIT][softasap-license-image] ][softasap-license-url]
[![Ansible-Container friendly][ansible-container-image] ][ansible-container-url]
[![Packer.io friendly friendly][packer-io-image] ][packer-io-url]


[ansible-container-image]: https://img.shields.io/badge/ansible--container-ready-brightgreen.svg
[ansible-container-url]: http://bit.ly/ansible-container
[softasap-license-image]: https://img.shields.io/badge/License-MIT-yellow.svg
[softasap-license-url]: https://opensource.org/licenses/MIT
[packer-io-image]: https://img.shields.io/badge/packer--io-ready-brightgreen.svg
[packer-io-url]: http://packer.io


Helper role to be executed with `ansible-container` or Hashicorp `packer` aiming to pre-configure guest OS for better image
Ubuntu part is based on Phusion BaseImage idea. See original license.

Third party ideas and libraries used
 - https://github.com/renatosilva/easyoptions
 - https://github.com/phusion/baseimage-docker


| Distribution |   BASE IMAGE  | +SSHD       | +CRON       | + syslog ng |
| ------------ |:-------------:|:-----------:|:-----------:|:-----------:|
| Alpine 3.4 | :white_check_mark: | :white_check_mark:| :white_check_mark:| :no_entry:|
| Alpine 3.5 | :white_check_mark: | :white_check_mark:| :white_check_mark:| :no_entry:|
| Alpine 3.6 | :white_check_mark: | :white_check_mark:| :white_check_mark:| :no_entry:|
| Alpine 3.7 | :white_check_mark: | :white_check_mark:| :white_check_mark:| :no_entry:|
| debian-jessie | :white_check_mark: | :white_check_mark:| :white_check_mark:| :white_check_mark:|
| debian-stretch | :white_check_mark: | :white_check_mark:| :white_check_mark:| :white_check_mark:|
| ubuntu-xenial | :white_check_mark: | :white_check_mark:| :white_check_mark:| :white_check_mark:|



Vars
----

Role can be configured using following parameters
```YAML

# GENERAL CONFIGURATION

# Desired init system:  phusion docker image compatible, using dumb-init
container_init: "phusion-init" # "dumb-init" "tini-init"

container_svc: "runit" # "supervisord"

# Directory where you can place executable files to be executed at startup
container_init_directory: /etc/my_init.d

# Install cron inside container (using runit service starter)
option_container_cron: true
# Install ssh service inside container (using runit service starter)
option_container_sshd: true
#... and enable service startup
option_container_sshd_enabled: true
#... using this key pair to connect inside instance:
container_ssh_private_key: "{{role_dir}}/files/keys/insecure_key"
container_ssh_public_key: "{{role_dir}}/files/keys/insecure_key.pub"
# Install syslog service inside container and redirect output to docker logs (using runit service starter)
option_container_syslog_ng: true


# phusion-init related configuration
# used by phusion-init, should point real installed python
container_python_interpeter: "/usr/bin/python3"


# dumb-init related configuration

dumb_init_version: "1.2.0"
```  

More insides on services
-------------------------

Container uses runit http://smarden.org/runit/ , which acts similar to classic upstart. It targets almost 100% of systems usually used as base for docker images.

Few examples on starting services with runit: place executable .runit files under `/etc/service/SERVICE_NAME/run`

```bash
#!/bin/sh
set -e
. /etc/memcached.conf
exec chpst -u memcache /usr/bin/memcached $MEMCACHED_OPTS >>/var/log/memcached.log 2>&1
```

```
#!/bin/bash
set -e
exec nginx -c /etc/nginx/nginx.conf
```

```

#!/bin/sh
set -e

RUNDIR=/var/run/redis
PIDFILE=$RUNDIR/redis.pid

mkdir -p $RUNDIR
touch $PIDFILE
chown redis:redis $RUNDIR $PIDFILE
chmod 755 $RUNDIR

exec chpst -u redis /usr/bin/redis-server /etc/redis/redis.conf
```

Running processes on startup
----------------------------

Placing sh files under `container_init_directory` ( default `/etc/my_init.d` ) will ensure that they are executed on startup.
If you decide to skip installation of the runit, this is the only way to get smth executed on boot.


Additional ways of setting environment inside container
-------------------------------------------------------

In addition to passing environment externally, you can put environment files under `/etc/container_environment` directory,
using following convention: file is named by the name of var you want to set, like ENVVARNAME and it's contents is value of the variable you want to set.


Container init specifics
------------------------

As was mentioned, role supports three initialization options: `phusion-init`, `dumb-init`, `supervisor-init`.
While `phusion-init` provides the same approach as we see on Phusion docker image, `dumb-init` and `supervisor-init` can be used in simpler
services.

`dumb-init` uses simpler init system: https://github.com/Yelp/dumb-init

`supervisor-init` known, but more heavy init system, often used with python projects.

Code in action
--------------

See box-example for the standalone working example. It will configure application
image that will display 'OK' on connect - check it out:

[![](https://github.com/play-with-docker/stacks/raw/cff22438cb4195ace27f9b15784bbb497047afa7/assets/images/button.png)](http://play-with-docker.com?stack=https://raw.githubusercontent.com/softasap/sa-container-bootstrap/master/box-example/ubuntu-xenial/docker-compose-try.yml)

More temporary hints on ansible container troubleshouting, if any on https://gist.github.com/Voronenko/77fc4743ef7e70d74ee74b7ee62fd7e5  ()


Copyright and license
---------------------

Code is dual licensed under the [BSD 3 clause] (https://opensource.org/licenses/BSD-3-Clause) and the [MIT License] (http://opensource.org/licenses/MIT). Choose the one that suits you best.


Reach us:

Subscribe for roles updates at [FB] (https://www.facebook.com/SoftAsap/)

Join gitter discussion channel at [Gitter](https://gitter.im/softasap)

Discover other roles at  http://www.softasap.com/roles/registry_generated.html

visit our blog at http://www.softasap.com/blog/archive.html
