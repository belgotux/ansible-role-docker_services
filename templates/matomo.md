# Matomo template
This template has a structured Compose and .env file, variables will automatically provisioned the files based on `docker_services_list`.

Here is an example of a Matomo instance; make sure to use your vault for your passwords :

```yml
docker_services_list:
  - name: 'mymatomo'
    service: 'mymatomo'
    template: 'matomo'
    routers_host: 'matomo.example.com'
    mariadb_password: 'xxx'
```

Example with availables options : 
```yml
docker_services_list:
  - name: 'mymatomo'
    service: 'mymatomo'
    template: 'matomo'
    routers_host: 'matomo.example.com'
    mariadb_password: 'xxx'
    matomo_version: 5
    mariadb_version: 11.4
    mariadb_limits_memory: '1G'
    mariadb_limits_cpus: '2.0'
    mariadb_max_allowed_packet: '64MB'
    matomo_limits_memory: '1G'
    matomo_limits_cpus: '2.0'
```

As you can see, we set the usual `docker_services` role parameters (name, service, template) but also some specific Matomo parameters. See the [Compose template](./matomo.yml.j2).

1. set the database password in the ansible vault
2. adjust the resource limits (memory / CPU) of the database and Matomo containers if needed
3. All the .env variables are generated from these variables
4. the Compose template automatically configures the reverse proxy URL (Traefik labels with the `secure-headers@file` middleware), the timezone (`Europe/Paris`) and the healthchecks. Matomo only starts once the database is healthy.

Note : the `MariaDB` root password is left empty on purpose (`MARIADB_ALLOW_EMPTY_ROOT_PASSWORD=1`). The database is only reachable from the internal `backend` network and is not exposed outside of the Compose project, so only the `matomo` user password matters.

Note : SQL files placed in the `init_sql` directory next to the Compose file are executed by MariaDB on the first start only (empty `dbdata` volume), through `/docker-entrypoint-initdb.d/`. Practical to import data.

## Variables list

| Variable | Required | Default | Description |
|---|---|---|---|
| `name` | Yes | | Name of the instance. Used as the MariaDB database name (`MARIADB_DATABASE`) and as the Traefik router name. |
| `service` | No | `app` | Name of the Matomo service in the Compose file. |
| `routers_host` | Yes | | FQDN of the instance. Used for the Traefik `Host()` rule. |
| `matomo_version` | No | `5` | Tag of the `matomo` image (e.g. `5`). Exposed as `MATOMO_VERSION` in the `.env` file. |
| `mariadb_version` | No | `11.4` | Tag of the `mariadb` image (e.g. `11.4`). Exposed as `MARIADB_VERSION` in the `.env` file. |
| `mariadb_password` | Yes | | Password of the Matomo database user (`matomo`). Store it in the vault. |
| `mariadb_limits_memory` | No | `1G` | Memory limit of the MariaDB container. |
| `mariadb_limits_cpus` | No | `2.0` | CPU limit of the MariaDB container. |
| `mariadb_max_allowed_packet` | No | `64MB` | MariaDB `--max-allowed-packet` value. Increase it if you import large datasets or hit "packet too large" errors. |
| `matomo_limits_memory` | No | `1G` | Memory limit of the Matomo container. |
| `matomo_limits_cpus` | No | `2.0` | CPU limit of the Matomo container. |