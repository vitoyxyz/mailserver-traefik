# Mailserver with Traefik v2 proxy

## Introduction

Set-up a [MailServer](https://github.com/vitoyxyz/docker-mailserver) with Traefik v2 reverse proxy, using Docker Compose.

**Prerequisites**

- Ubuntu 20.04 server(or any distro you want)
- [Docker](https://docs.docker.com/engine/install/) & [Docker-Compose](https://docs.docker.com/compose/install/) installed
- [Traefik](https://doc.traefik.io/traefik/getting-started/install-traefik/) installed
- Domain name

## Setting up stack

### Clone the repository

```shell
git clone https://github.com/vitoyxyz/mailserver-traefik && cd mailserver-traefik
```

## You need edit

### 1. In .env file replace with your hostname, domain, container name

```yml
# .env

HOSTNAME=mail
DOMAINNAME=domain.com
CONTAINER_NAME=mailserver
```

### 2. Traefik labels, with your domain

```yml
# docker-compose.yml

whoami: ...
  - traefik.http.routers.mail.rule=Host(`mail.domain.com`)
```

### 3. You need to provide the path to Traefik's acme.json to the container for SSL

```yml
# docker-compose.yml

services:
  mailserver: ...
    - /path/to/trafik/acme.json:/etc/letsencrypt/acme.json:ro
    ...
```

### 4. You edit the network name to your traefik network name

```yml
networks:
    proxy:  #this
        external: true
...
services:
    mailserver:
        ...
        networks:
            - proxy #this
```

### Running the stuck

First the the correct shell script version that is used for managing the mail server/container

```BASH
# if you're using :edge as the image tag
wget https://raw.githubusercontent.com/docker-mailserver/docker-mailserver/master/setup.sh
# if you're using :latest (= :9.0.1) as the image tag
wget https://raw.githubusercontent.com/docker-mailserver/docker-mailserver/v9.0.1/setup.sh

chmod a+x ./setup.sh

# and make yourself familiar with the script
./setup.sh help
```

If you'd like to use SELinux, add `-Z` to the variable `SELINUX_LABEL` in `.env`. If you want the volume bind mount to be shared among other containers switch `-Z` to `-z`

```BASH
docker-compose up -d

# for SELinux, use -Z
./setup.sh [-Z] email add <user@domain> [<password>]
./setup.sh [-Z] alias add postmaster@<domain> <user@domain>
./setup.sh [-Z] config dkim
```

If you're seeing error messages about unchecked error, please **verify that you're using the right version of `setup.sh`**. Refer to the [Get the tools](https://github.com/docker-mailserver/docker-mailserver#get-the-tools) section and / or execute `./setup.sh help` and read the `VERSION` section.

In case you're using LDAP, the setup looks a bit different as you do not add user accounts directly. Postfix doesn't know your domain(s) and you need to provide it when configuring DKIM:

```BASH
./setup.sh config dkim domain '<domain.com>[,<domain2.tld>]'
```

If you want to see detailed usage information, run `./setup.sh config dkim help`.

### Miscellaneous

#### DNS - DKIM

When keys are generated, you can configure your DNS server by just pasting the content of `config/opendkim/keys/domain.com/mail.txt` to [set up DKIM](https://mxtoolbox.com/dmarc/dkim/setup/how-to-setup-dkim). See the [documentation](https://docker-mailserver.github.io/docker-mailserver/edge/config/best-practices/dkim/) for more details.

#### Restart the container and start it again(For Postfix you to create atleast one email account first to start working correctly)

```BASH
docker-compose down

docker-compose up -d
```
