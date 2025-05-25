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

# Docker project dynamic vars (uses `docker_project_name` prefix, adapt if overridden)

# Main service additional docker-compose options (ex: cpu_shares, deploy, ...)
pihole_service_additional_options: |
  # See https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
  #cap_add:
  #  - NET_ADMIN # Required if you are using Pi-hole as your DHCP server, else not needed
  #  - SYS_TIME  # Required if you are using Pi-hole as your NTP client to be able to set the host's system time
  #  - SYS_NICE  # Optional, if Pi-hole should get some more processing time

# Traefik options
pihole_traefik_loadbalancer_server_port: 8081
pihole_traefik_entrypoints: http,https
pihole_traefik_middlewares:
  - "internal-access@file"


# Pi-hole docker-compose variables

# pihole/pihole image version
pihole_version: latest

# Default pihole user id
pihole_uid: "{{ ansible_user_uid }}"
# Default pihole group id
pihole_gid: "{{ ansible_user_gid }}"

# Docker default network MAC address
# Useful when using macvlan network driver (configure network in `pihole_compose_additional_options`).
# When not set, host networking mode will be used.
pihole_mac_address:

# Container hostname
pihole_container_hostname: pi.hole


# Pi-hole project variables

# Admin password
pihole_webpassword: ""

# Local DNS hosts
pihole_dns_hosts: []
#  - 192.168.0.123 example.lan

# Local DNS CNAME records
pihole_dns_cnameRecords: []
#  - test.example.lan,example.lan
#  - tmp.example.lan,example.lan,3600

# Initial adlists
pihole_adlists_init:
  - https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
```

```yaml
# Pihole environment configuration
# See https://docs.pi-hole.net/docker/configuration/#configuring-ftl-via-the-environment
pihole_env_config:

  ## Ports to be used by the webserver.
  FTLCONF_webserver_port: "{{ pihole_traefik_enable | default(false) | ternary(
    pihole_traefik_loadbalancer_server_port | default(8081), 
    '80o,443os,[::]:80o,[::]:443os') 
  }}"

  ## Upstream DNS server(s)
  #FTLCONF_dns_upstreams: "{{ pihole_dns_quad9 }}"

  ## Never forward non-FQDNs
  #FTLCONF_dns_domainNeeded: true # Pi-hole default: false

  ## DNS conditional forwarding
  #   format:  <enabled>,<ip-address>[/<prefix-len>],<server>[#<port>],<domain>
  #   example: true,192.168.0.0/24,192.168.0.1#53,lan
  #FTLCONF_dns_revServers: "false"

  ## Pi-hole network interface
  #FTLCONF_dns_interface: ""

  ## Pi-hole interface listening mode
  # "LOCAL":  Allow only local requests.
  # "SINGLE": Permit all origins, accept only on the specified interface.
  # "BIND":   Force FTL to really bind only the interfaces it is listening on.
  # "ALL":    Permit all origins, accept on all interfaces.
  # "NONE":   Do not add any configuration concerning the listening mode.
  #FTLCONF_dns_listeningMode: "LOCAL"

  ## Blocked queries will be answered with the "unspecified address" (0.0.0.0 or ::).
  #FTLCONF_dns_blocking_mode: "NULL"

  ## FTL dns.rateLimit configuration
  #FTLCONF_dns_rateLimit_count: 1000
  #FTLCONF_dns_rateLimit_interval: 60

  ## FTL database configuration
  #FTLCONF_database_maxDBdays: 91

  ## Hourly PTR requests for server hostnames
  # "IPV4_ONLY": Resolve only IPv4 addresses. (default)
  # "ALL":       Resolve all addresses.
  # "UNKNOWN":   Only resolve unknown hostnames. Already existing hostnames are never refreshed.
  # "NONE":      Don't do any hourly PTR lookups. Llook host names up exactly once.
  #FTLCONF_resolver_refreshNames: "IPV4_ONLY"
```

Dependencies
------------

This role depends on :
- [djuuu.docker_project](https://github.com/Djuuu/ansible-role-docker-project)

Some variables allow integration with:
- [djuuu.traefik_docker](https://github.com/Djuuu/ansible-role-traefik-docker)

Example Playbooks
-----------------

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

```yaml
- hosts: all
  gather_facts: false

  tasks:
    - name: Configure Pi-hole local DNS records
      ansible.builtin.include_role:
        name: djuuu.pihole_docker
        tasks_from: configure-dns
```

License
-------

Beerware License
