<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Alejandro AR
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Vikunja

This is an [Ansible](https://www.ansible.com/) role which installs [Vikunja](https://vikunja.io/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Vikunja is a self-hosted to-do application.

See the project's [documentation](https://vikunja.io/docs/) to learn what Vikunja does and why it might be useful to you.

## Prerequisites

To run a Vikunja instance it is necessary to prepare a database. You can use a [MySQL](https://www.mysql.com/) compatible database server, [Postgres](https://www.postgresql.org/), or [SQLite](https://www.sqlite.org/). The SQLite database file will be automatically created by the service if it is enabled.

If you are looking for Ansible roles for a MySQL compatible server or Postgres, you can check out [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Vikunja with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# vikunja                                                              #
#                                                                      #
########################################################################

vikunja_enabled: true

########################################################################
#                                                                      #
# /vikunja                                                             #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Vikunja you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
vikunja_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Vikunja under a subpath (by configuring the `vikunja_path_prefix` variable) does not seem to be possible due to Vikunja's technical limitations.

### Set a random string for JWT tokens verification

You also need to set a random string used for verifying issued JWT tokens. To do so, add the following configuration to your `vars.yml` file. The value can be generated with `pwgen -s 64 1` or in another way.

```yaml
vikunja_environment_variables_service_secret: YOUR_SECRET_KEY_HERE
```

### Configuring database

#### Specify database

It is necessary to select database used by Vikunja from a MySQL compatible database, Postgres, and SQLite.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
vikunja_database_type: postgres
```

Set `mysql` to use a MySQL compatible database and `sqlite` to use SQLite, respectively. The SQLite database is stored in the directory specified with `vikunja_database_path`.

For other settings, check variables such as `vikunja_database_postgres_*` and `vikunja_database_mysql_*` on [`defaults/main.yml`](../defaults/main.yml).

#### Configuring connection to the database server (optional)

By default the role is configured to establish the connection to the database server via a Unix socket. You can mount the Unix socket by adding the following configuration to your `vars.yml` file:

```yaml
# Specify the path to the MySQL compatible server's Unix socket path on the host (bind-mount source)
vikunja_database_mysql_socket_path_host: ""

# Specify the path to the Postgres Unix socket path on the host (bind-mount source)
vikunja_database_postgres_socket_path_host: ""
```

Setting it enables to connect to the database server via Unix socket mounted in the container.

If TCP connection is preferred, connection via the Unix socket can be disabled by adding the following configuration to your `vars.yml` file:

```yaml
# Disable the connection to the MySQL compatible server via a Unix socket
vikunja_database_mysql_socket_enabled: false

# Disable the connection to the Postgres server via a Unix socket
vikunja_database_postgres_socket_enabled: false
```

### Configuring a Redis database (optional)

You can optionally enable a [Redis](https://redis.io/) database for the Vikunja instance. [Valkey](https://valkey.io/) can also be used instead.

To enable the Redis database for Vikunja, add the following configuration to your `vars.yml` file. Note that the role is by default configured to establish connection with the Redis database via the Unix socket.

```yaml
# Specify the path to the Redis Unix socket path on the host (bind-mount source)
vikunja_redis_socket_path_host: ""

vikunja_redis_database: 0
```

If TCP connection is preferred, connection via the Unix socket can be disabled by adding the following configuration to your `vars.yml` file:

```yaml
# Disable the connection to Redis via a Unix socket
vikunja_redis_socket_enabled: false

vikunja_redis_hostname: YOUR_REDIS_SERVER_HOSTNAME_HERE
```

Make sure to replace `YOUR_REDIS_SERVER_HOSTNAME_HERE` with your own value.

If you are looking for an Ansible role for Redis, you can check out [ansible-role-redis](https://github.com/mother-of-all-self-hosting/ansible-role-redis) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team. The role for Valkey ([ansible-role-valkey](https://github.com/mother-of-all-self-hosting/ansible-role-valkey)) is available as well.

### Configuring the mailer (optional)

You can configure a SMTP mailer to enable it for signing up and resetting password. If it is disabled, all users are enabled right away and password reset will not be possible.

To configure it, add the following configuration to your `vars.yml` file as below (adapt to your needs):

```yaml
# Set to `true` to enable mailer
vikunja_mailer_enabled: true

# Specify SMTP server hostname
vikunja_environment_variables_smtp_host: ""

# Specify SMTP server port number
vikunja_environment_variables_smtp_port: 587

# Specify SMTP server username
vikunja_environment_variables_smtp_user: ""

# Specify SMTP server password
vikunja_environment_variables_smtp_password: ""

# Specify the email address that emails will be sent from
vikunja_environment_variables_smtp_from: ""

# Specify the SMTP Auth Type
# Valid values: plain, login, cram-md5
vikunja_environment_variables_smtp_authtype: plain

# Set to `true` to skip verification of the TLS certificate on the server
vikunja_environment_variables_skiptlsverify: false

# Set to `true` to use SSL instead of STARTTLS
vikunja_environment_variables_smtp_forcessl: false
```

>[!WARNING]
> Without setting an authentication method such as DKIM, SPF, and DMARC for your hostname, emails are most likely to be quarantined as spam at recipient's mail servers. The worst scenario is that your server's IP address or hostname will be included in the spam list such as the one managed by [Spamhaus](https://www.spamhaus.org/). If you have set up a mail server with the [MASH project's exim-relay Ansible role](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay), you can enable DKIM signing with it. Refer [its documentation](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay/blob/main/docs/configuring-exim-relay.md#enable-dkim-support-optional) for details.

### Enabling user registration

By default the role is configured to disable user registration. You can enable it by adding the following configuration to your `vars.yml` file.

```yaml
vikunja_environment_variables_service_enableregistration: true
```

Alternatively, you can also create users by running the command to run [`user create`](https://vikunja.io/docs/cli/#user-create) inside the container. See below in [this section](#creating-users) for the usage.

### Configuring rate limit

You can enable the rate limit by adding the following configuration to your `vars.yml` file:

```yaml
vikunja_environment_variables_ratelimit_enabled: true
```

### Integrating with Prometheus (optional)

Vikunja can natively expose metrics to Prometheus.

If you are looking for an integration, you can check out the MASH playbook. Refer to [this section of the documentation on the playbook](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/services/vikunja.md#integrating-with-prometheus-optional) for more information.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `vikunja_environment_variables_additional_variables` variable

Refer to [the official documentation](https://vikunja.io/docs/config-options/) for a complete list of Vikunja's config options that you can put in `vikunja_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Vikunja becomes available at the specified hostname like `https://example.com`.

To get started, create a user first and open the URL with a web browser to log in to the instance. You can create one on the web UI if `vikunja_environment_variables_service_enableregistration` is set to `true`.

Alternatively, you can run the command below to create users.

### Creating users

#### Creating a user manually

You can create a user by running the command below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=create-user-vikunja -e username=USERNAME_HERE -e password=PASSWORD_HERE -e email=EMAIL_ADDRESS_HERE
```

#### Creating users automatically

It is also possible to create muitiple users specified with `vikunja_users_custom` on your `vars.yml` file by running the command below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=ensure-vikunja-users-created
```

Those users can be specified like below:

```yaml
vikunja_users_custom:
  - username: user
    initial_email: user@example.com
    initial_password: password
  - username: user
    initial_email: user@example.com
    initial_password: password
```

### Running the CLI command

It is possible to run commands on the command line inside the container by running the `cli-vikunja` tag, setting the `command` extra variable.

For example, you can run the command `version` by running the playbook with the tag as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=cli-vikunja -e command='version'
```

Refer to [this page](https://vikunja.io/docs/cli/) for the list of available commands.

### Typesense integration for enhanced search capabilities

Vikunja supports [Typesense](https://typesense.org/), which allows fast fulltext search with fuzzy matching support. To enable it, the following configuration to your `vars.yml` file:

```yaml
vikunja_environment_variables_typesense_enabled: true
vikunja_environment_variables_typesense_url: TYPESENSE_INSTANCE_URL_HERE
vikunja_environment_variables_typesense_apikey: TYPESENSE_ADMIN_API_KEY_HERE
```

Make sure to replace `TYPESENSE_INSTANCE_URL_HERE` and `TYPESENSE_ADMIN_API_KEY_HERE` with your own values.

After adding the configuration and restarting the service, it is necessary to run `vikunja index` to have Typesense index tasks in the Vikunja instance. You can invoke it by running the playbook below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=cli-vikunja -e command='index'
```

Refer to [this page](https://vikunja.io/docs/typesense/) as well.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu vikunja` (or how you/your playbook named the service, e.g. `mash-vikunja`).

#### Increase logging verbosity

If you want to increase the verbosity, add the following configuration to your `vars.yml` file:

```yaml
vikunja_environment_variables_log_level: DEBUG
```
