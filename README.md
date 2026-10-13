Docker-services
===============

This role deploys Docker Compose services, with override and `.env` files. It provides a generic Compose template and supports **your** specific templates.

Requirements
------------

- Docker installed (with [my role](https://galaxy.ansible.com/belgotux/docker) if needed [github](https://github.com/belgotux/ansible-role-docker))
- The `community.docker` Ansible collection. Install it with `ansible-galaxy collection install community.docker`, see the [module documentation](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_compose_module.html)

Dependencies
------------
The role installs the following dependencies:
- pip3
- pip lib for docker
- docker

Example Playbook
----------------

## Simple playbook example
```yml
- hosts: "{{ myhost | default('all') }}"
  remote_user: root

  roles:
    - role: belgotux.docker_services
  vars:
    docker_services_list:
      - name: 'whoami'
        service: 'whoami'
        image: 'containous/whoami'
        mem_limit: '6m'
        environments: ["TZ=Europe/Paris"]
        ports: ["8080:80"]
```

## Example with Traefik
You can find my Traefik role on [Ansible Galaxy](https://galaxy.ansible.com/belgotux/docker_traefik) or [GitHub](https://github.com/belgotux/ansible-role-docker_traefik).
```yml
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

## Using configuration files in the "templates" folder
Create a folder with the **name** of your docker compose into `files`:
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
No service-specific configuration is required. The role automatically checks for a matching directory under `files` and copies its contents.

### Files structure
This example uses yaml files to configure the services, with environment file or override file
```
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

Add the service name to the list:
```yml
docker_services_list:
  - name: 'mysrv'
```

### Using one template and multiple instances
You can deploy multiple instances of the same application, each with its own Compose and environment files. The template filenames must match each instance's `name`.

Here is an Odoo example.

#### The docker_services_list
```yml
docker_services_list:
  - name: 'my_instance_1'
    service: 'odoo'
    template: 'odoo'
    routers_host: 'my_instance_1.yourdomain.tld'
    db_password: '{{ ansible_vault.db_password }}'
```

#### The Docker Compose YAML file
Typical service with an app and a database `template/odoo.yml.j2` :
```yml
networks:
  backend:
  proxy-net:
    external: true

volumes:
  data:
  db-data:

services:
  odoo:
    image: odoo:${ODOO_VERSION}
    depends_on:
      db:
        condition: service_healthy
    ports:
      - "${ODOO_PORT}:8069"
      - "${ODOO_GEVENT_PORT}:8072"
    volumes:
      - data:/var/lib/odoo
    environment:
      - HOST=${ODOO_DB_HOST}
      - USER=${ODOO_DB_USER}
      - PASSWORD=${ODOO_DB_PASSWORD}
    command:
      - odoo
      - --proxy-mode
      - --no-database-list
      - --workers=3
      - --http-interface=0.0.0.0
      - --limit-time-cpu=600
      - --limit-time-real=1200
      - --log-level="warn"
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8069/web/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s
    networks:
      - backend
      - proxy-net

  db:
    image: postgres:${POSTGRES_VERSION}
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - db-data:/var/lib/postgresql/data/pgdata
    restart: unless-stopped
```

#### The environment file for an instance
Your can have multiple environment files, one for eatch instance.
Here an example with `my_instance_1.env.j2`. It **MUST** match the name in the `docker_services_list`.

See you can use variables like item.name to catch the `docker_services_list.my_instance_1.name` value.
You can add **your** custom variable too, like `item.db_password` here.
```
# Odoo
ODOO_VERSION=20.0
ODOO_PORT=8069
ODOO_GEVENT_PORT=8072
ODOO_DB_HOST=db
ODOO_DB_USER=odoo
ODOO_DB_PASSWORD={{ item.db_password }}
ODOO_DB_NAME={{ item.name }}
ODOO_WORKERS=4
ODOO_MAX_CRON_THREADS=2
ODOO_LIST_DB=False
ODOO_PROXY_MODE=True

# PostgreSQL
POSTGRES_VERSION=17-alpine
POSTGRES_USER=odoo
POSTGRES_PASSWORD={{ item.db_password }}
POSTGRES_DB={{ item.name }}
```

#### Another example with custom variables
When using Traefik, you may need to configure a router name and rule. You can use custom variables for these values.

The following example configures labels for 2 matomo instances:

1. In the service list:
```yml
docker_services_list:
  - name: 'matomo-instance1'
    template: 'matomo'
    routers_rule: 'Host(`instance1.yourdomain.tld`)'
  - name: 'matomo-instance2'
    template: 'matomo'
    routers_rule: 'Host(`instance2.yourdomain.tld`)'
```

2. In your compose file `matomo.yml.j2` your can use this 
```yml
labels:
  - traefik.enable=true
  - traefik.http.routers.{{ item.name | default ('matomo') }}.entrypoints=websecure
  - traefik.http.routers.{{ item.name | default ('matomo') }}.tls=true
  - traefik.http.routers.{{ item.name | default ('matomo') }}.rule={{ item.routers_rule }}
  - traefik.http.routers.{{ item.name | default ('matomo') }}.middlewares=secure-headers@file
```

Role Variables
--------------
The role can run with the [default configuration](defaults/main.yml); with an empty service list, it deploys nothing.

### Mandatory service variables

Each entry in `docker_services_list` must define `name`. When using the generic Compose template, it must also define `service`. A service-specific template named `<name>.yml.j2` does not require `service`.

| Variable | Description |
| --- | --- |
| `docker_services_list.name` | Service/project name, determines its directory and the names of any service-specific files |
| `docker_services_list.service` | Compose service key used by the generic template, required only when no service-specific Compose template is used. |

### Optional role variables

| Variable | Default | Description |
| --- | --- | --- |
| `docker_services_docker_conf` | `/etc/docker` | Base directory where each service's project directory is created |
| `docker_services_docker_group` | `docker` | Group that can access to docker daemon |
| `docker_services_docker_user` | `{{ remote_user }}` | Force a user on the files (default is the user executing the playbook) |
| `docker_services_list` |  | Services to deploy |

### Optional service variables

These fields are properties of each `docker_services_list` entry, not top-level role variables. They apply to the generic Compose template; service-specific templates may use their own fields.

| Variable | Description |
| --- | --- |
| `image` | Container image |
| `mem_limit` | Container memory limit |
| `volumes` | Volume entries |
| `networks` | Networks attached |
| `caps` | Linux capabilities emitted as Compose `cap_add` |
| `ports` | Ports |
| `environments` | Environment entries |
| `restart` | Restart policy (defaults `unless-stopped`) |
| `labels` | Compose labels |
| `extra` | Additional Compose YAML inserted into the generated service definition |

If using custom compose file :
| Variable | Description |
| `template` | Docker compose template; determines the name of the file to use as template. |

All optional service fields are omitted when not supplied, except `restart`, which defaults to `unless-stopped`.

License
-------

[GPL v3](https://www.gnu.org/licenses/gpl-3.0.en.html)

Author Information
------------------

Belgotux
[MonLinux](https://www.monlinux.net)
