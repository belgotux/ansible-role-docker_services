# GLPI template
This template has a structured Compose and .env file, variables will automatically provisioned the files based on `docker_services_list`.

Here is an example of GLPI instance; make sure to use your vault for your passwords : 
```yml
- name: 'myglpi'
    service: 'myglpi'
    template: 'glpi'
    routers_host: 'myglpi'
    glpi_version: 11
    mariadb_version: 12.3
    db_password: 'xxx'
    db_root_password: 'yyy'
    glpi_market_key: 'zzz'
    glpi_lang: 'fr_BE'
    glpi_plugins:
      - samlsso
```

As you can see, we set the usual `docker_services` role parameters (name, service, template) but also some specific glpi parameters. See the [Compose template](./glpi.yml.j2).

1. set password in the ansible vault
2. set the language and plugins
3. All the .env variables are generated from these variables
4. the Compose template automatically configures the reverse proxy URL, language, timezone using a compagnion Docker container to set all of this

Note : for `GLPI_TRUSTED_PROXIES`, default docker networks are `172.16.0.0/12,10.0.0.0/8` but you can override it with `trusted_proxies_ips: x.x.x.x/x` if you change them.

## Variables list

| Variable | Required | Default | Description |
|---|---|---|---|
| `routers_host` | Yes | | FQDN of the instance. Used for the Traefik `Host()` rule and to set `url_base` and `url_base_api` in GLPI. |
| `glpi_version` | Yes | 11 | Tag of the `glpi/glpi` image (e.g. `11`). Exposed as `GLPI_VERSION` in the `.env` file. |
| `glpi_lang` | No | `en_US` | Default language of the instance (e.g. `fr_BE`, `fr_FR`). |
| `glpi_market_key` | No | | GLPI Network registration key. If defined, it is registered in GLPI by the `init` container. Store it in the vault. |
| `glpi_plugins` | No | `[]` | List of marketplace plugins to download, install and activate automatically (e.g. `samlsso formcreator`). |
| `mariadb_version` | Yes | 12.3 | Tag of the `mariadb` image (e.g. `12.3`). Exposed as `MARIADB_VERSION` in the `.env` file. |
| `db_password` | Yes | | Password of the GLPI database user. Store it in the vault. |
| `db_root_password` | Yes | | Password of the MariaDB root user. Store it in the vault. |