Docker-services
===============

Role to deploy docker-compose, with override and .env files.
Template for most of docker-compose file but you can add your specific template too.

Requirements
------------

- Docker installed (with [my role](https://galaxy.ansible.com/belgotux/docker) if needed [github](https://github.com/belgotux/ansible-role-docker))
- Ansible collection community.docker ([documentation](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_compose_module.html)) : `ansible-galaxy collection install community.docker`

Role Variables
--------------
The role can work as it with the [default configuration](defaults/main.yml).

## Docker configuration

### Mandatory
- `docker_services_docker_path` docker conf repository (default `/etc/docker`)
- `docker_services_list` list of services, see the example below (default deploy a whoami)

### Optionnal
- `docker_services_docker_user` docher user (default `{{ remote_user }}`)
- `docker_services_docker_group` docker group (default `docker`)

Dependencies
------------
Installed by the role :
- pip3
- pip lib for docker
  - `docker`

Example Playbook
----------------

## simple
```
- hosts: '{{ myhost | default('all') }}'
  remote_user: root

  roles:
    - role: belgotux.docker_services
  variables:
    docker_services_list:
      - name: 'whoami'
        service: 'whoami'
        image: 'containous/whoami'
        mem_limit: '6m'
        environments: ["TZ=Europe/Paris"]
        ports: ["8080:80"]
```

## with traefik
You can find my Traefik role on [ansible galaxy](https://galaxy.ansible.com/belgotux/docker_traefik) or [github](https://github.com/belgotux/ansible-role-docker_traefik)
```
docker_services_list:
  - name: 'whoami'
    service: 'whoami'
    image: 'containous/whoami'
    mem_limit: '6m'
    networks: ["proxy-net"]
    volumes: ["$PWD/config:/config", "$PWD/modules:/lib/modules"]
    caps: ["NET_ADMIN", "SYS_MODULE"]
    environments: ["TZ=Europe/Paris"]
    labels:
      - traefik.enable=true
      - traefik.http.routers.whoami.entrypoints=websecure
      - traefik.http.routers.whoami.tls.certresolver=le
      - "traefik.http.routers.whoami.rule=Host(`whoami.yourdomain.tld`)"
      - traefik.http.routers.whoami.middlewares=secure-headers@file
    extra: |
      networks:
        proxy-net:
          external: true
```

License
-------

[GPL v3](https://www.gnu.org/licenses/gpl-3.0.en.html)

Author Information
------------------

Belgotux
[MonLinux](https://www.monlinux.net)
