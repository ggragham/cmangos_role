CMaNGOS Role
============

Ansible role for CMaNGOS deployment on Linux.

The role is based on the [ThoriumLXC](https://thoriumlxc.github.io/) project and adapts its deployment model to a simplified Ansible-based workflow with a reduced set of required variables.

Table of Contents
-----------------

- [Features](#features)
- [Requirements](#requirements)
- [Role Variables](#role-variables)
- [Installation](#installation)
- [Usage](#usage)
- [Account Creation](#account-creation)
- [License](#license)
- [Author Information](#author-information)

Features
--------

- simplified CMaNGOS deployment through Ansible using [ThoriumLXC container images](https://hub.docker.com/r/thoriumlxc/cmangos-tbcdb)
- reduced number of required variables for initial setup
- expansion selection: `classic`, `tbc`, `wotlk`
- deployment with or without bots
- fixed Docker Compose config mounting, including bot configs
- fixed `ahbot` config
- fixed `aiplayerbot` configs for `tbc` and `wotlk`
- `realmd.realmlist` correction during deployment
- default account disabling during deployment
- optional phpMyAdmin deployment
- optional Nginx proxying for `realmd` and `mangosd`
- phpMyAdmin protection with Nginx Basic Auth
- dedicated account creation action
- SELinux compatibility

Requirements
------------

- Ansible on the control node
- reachable RHEL- or Debian-based target host with Docker
- SSH access and required privileges on the target host

Role Variables
--------------

```yml
# Core settings
CMANGOS_DOMAIN: 127.0.0.1  # Realm address for clients; can be an FQDN or an IP address.
CMANGOS_EXPANSION: tbc  # Game expansion: classic, tbc, or wotlk.
CMANGOS_ASSETS_SRC: ./client-data  # Path to extracted WoW client-data, or keep ./client-data next to the playbook.

CMANGOS_HOST_UID: 1000  # UID of the primary user on the host; usually 1000.
CMANGOS_HOST_GID: 1000  # GID of the primary user on the host; usually 1000.

CMANGOS_DB_USERNAME: mangos_username  # Database application username.
CMANGOS_DB_PASSWORD: mangos_password  # Database application password; recommended to change.
CMANGOS_DB_ROOT_PASSWORD: mangos_root_password  # Database root password; recommended to change.

# Bot settings
CMANGOS_ENABLE_BOTS: true  # Enable "AI" player bots.
CMANGOS_AIPLAYERBOT_MIN_RANDOM_BOTS: 900  # Minimum random bot count.
CMANGOS_AIPLAYERBOT_MAX_RANDOM_BOTS: 1000  # Maximum random bot count.

# Advanced settings

## Container features
CMANGOS_DOCKER_TAG: 2025.05.11  # Container image version tag from ThoriumLXC images (docker.io/thoriumlxc/).
CMANGOS_INSTALL_PHPMYADMIN: false  # Enable phpMyAdmin for database administration; exposed on port 8080 when enabled.

## Nginx settings
CMANGOS_INSTALL_NGINX: true  # Enable Nginx reverse proxy service.
CMANGOS_NGINX_DOCKER_TAG: latest  # Nginx container image tag.
CMANGOS_NGINX_USE_SSL: false  # Enable SSL/TLS for Nginx.
CMANGOS_NGINX_PHPMYADMIN_BASIC_AUTH_USER: admin  # Username for Nginx Basic Auth protecting phpMyAdmin.
CMANGOS_NGINX_PHPMYADMIN_BASIC_AUTH_PASSWORD: change_me  # Password for Nginx Basic Auth protecting phpMyAdmin; recommended to change.
CMANGOS_NGINX_SSL_CERT_PATH: ./cert/default.crt  # SSL certificate path.
CMANGOS_NGINX_SSL_KEY_PATH: ./cert/default.key  # SSL private key path.
CMANGOS_NGINX_SSL_DH_PATH: ./cert/dhparam.pem  # Diffie-Hellman parameters file path.

## Realm settings
CMANGOS_MANGOSD_SERVER_NAME: CMaNGOS  # Realm name shown in the client.
CMANGOS_MANGOSD_GAME_TYPE: 1  # Realm game type; see mangosd.conf for valid values.
CMANGOS_MANGOSD_REALM_ZONE: 1  # Realm region/zone ID; see mangosd.conf for valid values.
CMANGOS_MANGOSD_MOTD: Welcome to the Continued Massive Network Game Object Server.  # Message of the day.

## Rates
CMANGOS_MANGOSD_XP_RATE: 1  # Experience rate multiplier. Examples: 1 = normal, 2 = double rate, 0.5 = half rate.
CMANGOS_MANGOSD_MONEY_RATE: 1  # Money drop rate multiplier. Examples: 1 = normal, 2 = double rate, 0.5 = half rate.
```

Installation
------------

Create `requirements.yml`:

```yaml
- name: cmangos_role
  src: git+https://github.com/ggragham/cmangos_role.git
  scm: git
  version: master
```

Install the role:

```bash
ansible-galaxy install -r requirements.yml -p roles
```

Usage
-----

Create `vars.yml` and copy the required variables from the [Role Variables](#role-variables) section into it.

Create an Ansible inventory describing the target host. See the [Ansible inventory documentation](https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html) for supported formats and examples.

Create `playbook.yml`:

```yaml
- hosts: servers
  vars_files:
    - vars.yml
  roles:
    - role: cmangos_role
```

Run deployment:

```bash
ansible-playbook -i <inventory> playbook.yml
```

Account Creation
----------------

The role provides a dedicated account creation action.

Interactive mode:

```bash
ansible-playbook -i inventory playbook.yml -e action=create_account
```

The playbook will prompt for:

- account name
- account password
- GM level

Non-interactive mode:

```bash
ansible-playbook -i inventory playbook.yml \
  -e action=create_account \
  -e account_name=admin \
  -e account_password=very_strong_password \
  -e account_gm_level=3
```

License
-------

BSD

Author Information
------------------

[Grell Gragham](https://github.com/ggragham)
