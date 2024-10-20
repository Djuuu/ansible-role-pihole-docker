Ansible Role: Pi-hole-docker
============================

Install Pi-hole Docker Compose project.

- https://pi-hole.net/
- https://github.com/pi-hole/pi-hole

Requirements
------------

Requires the following to be installed:
- docker
- docker compose

Role Variables
--------------

Common system variables:

```yaml
timezone: UTC
```

Common Docker projects variables:

```yaml
# Base directory for Docker projects
docker_projects_path: # /var/apps
```

Available role variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# Docker project variables

pihole_project_name: pihole

# Docker project dynamic vars (uses `docker_project_name` prefix, adapt if overriden)

# Traefik options
pihole_traefik_loadbalancer_server_port: 8081
pihole_traefik_entrypoints: http,https
pihole_traefik_middlewares:
  - "internal-access@file"
  - "allow-frames@file"
  - "{{ docker_project_slug }}-cors@docker"


# Pi-hole docker-compose variables

# Docker default network MAC address
# Useful when using macvlan network driver (configure network in `pihole_compose_additional_options`).
# When not set, host networking mode will be used.
pihole_mac_address:

# Container hostname
pihole_container_hostname: pi.hole


# Pi-hole project variables

## Recommended Variables

# Admin password (WEBPASSWORD  environment variable)
pihole_webpassword: ""

## Optional Variables

# Upstream DNS server(s) (PIHOLE_DNS_ environment variable)
# see available DNS vars in vars/main.yml
pihole_dns: "{{ pihole_dns_quad9 }}"

# DNSSEC support
pihole_dnssec: false

# Never forward reverse lookups for private ranges (DNS_BOGUS_PRIV environment variable)
pihole_dns_bogus_priv: true
# Never forward non-FQDNs (DNS_FQDN_REQUIRED environment variable)
pihole_dns_fqdn_required: true

# Enable DNS conditional forwarding for device name resolution (REV_SERVER environment variable)
pihole_rev_server: false
# If conditional forwarding is enabled, set the domain of the local network router (REV_SERVER_DOMAIN environment variable)
pihole_rev_server_domain:
# If conditional forwarding is enabled, set the IP of the local network router (REV_SERVER_TARGET environment variable)
pihole_rev_server_target:
# If conditional forwarding is enabled, set the reverse DNS zone (REV_SERVER_CIDR environment variable)
pihole_rev_server_cidr:

## Advanced Variables

# Pi-hole interface (INTERFACE environment variable)
pihole_interface:

# DNSMASQ_LISTENING environment variable
# 'local' listens on all local subnets,
# 'all' permits listening on internet origin subnets in addition to local,
# 'single' listens only on the interface specified.
pihole_dnsmasq_listening: local

# CORS_HOSTS environment variable
pihole_cors_hosts: []

# FTL
pihole_ftlconf_rate_limit: 1000/60 # FTLCONF_RATE_LIMIT environment variable
pihole_ftlconf_maxdbdays:  365     # FTLCONF_MAXDBDAYS environment variable
```

Additional config files
-----------------------

Additional files located in `config/pihole/{{ inventory_hostname }}/` will be copied in the project main directory:

- `config/pihole/all/etc-dnsmasq.d/`
- `config/pihole/all/etc-pihole/`
- `config/pihole/{{ inventory_hostname }}/etc-dnsmasq.d/`
- `config/pihole/{{ inventory_hostname }}/etc-pihole/`

Ex:
- `config/pihole/`
  - `all/`
    - `etc-dnsmasq.d/`
      - `05-pihole-custom-cname.conf` (local CNAME records)
    - `etc-pihole/`
      - `adlists.list` (initial ad lists, not used by Pi-hole anymore)
      - `custom.list` (local DNS configuration)
  - `my-host/`
    - `etc-dnsmasq.d/`
      - `04-docker-resolving.conf` (local Dnsmasq configuration)

Dependencies
------------

This role depends on :
- [djuuu.docker_project](https://github.com/Djuuu/ansible-role-docker-project)

Some variables allow integration with:
- [djuuu.traefik_docker](https://github.com/Djuuu/ansible-role-traefik-docker)

Example Playbook
----------------

```yaml
- hosts: all
  gather_facts: true
  gather_subset:
    - "!all"
    - "!min"
    - user_id

  roles:
    - djuuu.pihole_docker
```

License
-------

Beerware License
