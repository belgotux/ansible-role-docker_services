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
- `docker_services_docker_conf` docker conf repository (default `/etc/docker`)
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

### Example with config files in the "files" folder
Create a folder with the **name** of your docker compose into `files` like this : 
```
roles/docker_services
├── README.md
├── defaults
│   └── main.yml
├── files
│   └── nginx_prout
│       └── nginx.conf
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   ├── deploy_docker_compose.yml
│   └── main.yml
├── templates
│   └── docker-compose.yml.j2
```
No need modification is the service, folder presence is automatically check!

### Example with variables only
```
  - name: 'nginx_prout'
    service: 'nginx'
    image: 'nginx:1.27'
    mem_limit: '300m'
    networks: ["proxy-net"]
    volumes: ["/files:/usr/share/nginx/html:ro", "$PWD/nginx.conf:/etc/nginx/conf.d/default.conf:ro"]
    environments: ["TZ=Europe/Brussels"]
    labels:
      - traefik.enable=true
      - traefik.http.routers.nginx.entrypoints=websecure
      - traefik.http.routers.nginx.tls.certresolver=le
      - "traefik.http.routers.nginx.rule=Host(`nginx.yourdomain.tld`)"
      - traefik.http.routers.nginx.middlewares=secure-headers@file
    extra: |
      networks:
        proxy-net:
          external: true
```

### Example with files only
```
Here, we using yaml files to configure the services, with environment file or override file
roles/docker_services
├── README.md
├── defaults
│   └── main.yml
├── files
│   └── mysrv
│       └── nginx.conf
├── handlers
│   └── main.yml
├── tasks
│   ├── deploy_docker_compose.yml
│   └── main.yml
├── templates
│   ├── mysrv.yml.j2
│   ├── mysrv.env.j2
│   └── mysrv.override.yml.j2
```

License
-------

[GPL v3](https://www.gnu.org/licenses/gpl-3.0.en.html)

Author Information
------------------

Belgotux
[MonLinux](https://www.monlinux.net)
