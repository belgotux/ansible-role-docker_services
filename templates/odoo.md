# ODOO template
This template has a structured Compose and .env file, variables will automatically provisioned the files based on `docker_services_list`.

Here is an example of ODOO instance; make sure to use your vault for your passwords : 
```yml
docker_services_list:
  - name: 'my_instance_1'
    service: 'odoo'
    template: 'odoo'
    routers_host: 'my_instance_1.yourdomain.tld'
    db_password: 'xxx'
```

As you can see, we set the usual `docker_services` role parameters (name, service, template) but also some specific glpi parameters. See the [Compose template](./odoo.yml.j2).
Few documentation, [here a good one](https://oec.sh/guides/odoo-docker-hub#environment-variables).

## Special action for the first run of Odoo

Odoo is special, the script in the docker image don't initialise the database if itis empty like other apps...
And you can't have the workers running at this moment. the odoo docker need to be down for the initialisation
You need to :
1. down the stack
2. up the db
3. doing a init command with un one shot run

```sh
docker compose down -v
docker compose up -d db
docker compose run --rm odoo odoo -d <database> -i base --stop-after-init
docker compose up -d odoo && docker compose logs -f
```

## Variables list

| Variable | Required | Default | Description |
|---|---|---|---|
| `name` | Yes |  | `POSTGRES_DB` | Instance name, also used as the database name |
| `routers_host` | Yes |  | Nom de domaine (FQDN) pour le routage Traefik (ex: `erp.example.com`). |
| `db_password` | Yes |  | PostgreSQL password (shared by Odoo and the database) |
| `odoo_version` | No | 20.0 | Odoo image version |
| `postgres_version` | No | 16-alpine | PostgreSQL image tag |
| `odoo_workers` | No | 4 | Number of Odoo workers |
| `odoo_cron_threads` | No | 2 | Number of threads for scheduled actions |
| `odoo_db_maxconn` | No | 48 | Maximum number of persistent database connections per Odoo worker. |
| `odoo_limit_time_cpu` | No | 600 | CPU time limit in seconds for a single Odoo request. |
| `odoo_limit_time_real` | No | 1200 | Real (wall-clock) time limit in seconds for a single Odoo request. |
| `odoo_log_level` | No | warn | Odoo logging level (`debug`, `info`, `warn`, `error`, `critical`). |
| `odoo_limits_memory` | No | 3G | Memory limit for the Odoo container. |
| `odoo_limits_cpus` | No | 2.0 | CPU limit for the Odoo container (in number of cores). |
| `db_limits_memory` | No | 2G | Memory limit for the PostgreSQL container. |
| `db_limits_cpus` | No | 2.0 | CPU limit for the PostgreSQL container (in number of cores). |
| `workers` | No | 4 | Number of Odoo workers (used to calculate PostgreSQL `max_connections`). |
| `db_maxconn` | No | `odoo_workers`*`odoo_db_maxconn` | Maximum database connections per worker (used to calculate PostgreSQL `max_connections`). |
