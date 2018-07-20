# Ansible Role: Install akeneo

[![Build Status](https://travis-ci.org/tschifftner/ansible-role-akeneo.svg?branch=master)](https://travis-ci.org/tschifftner/ansible-role-akeneo)

Installs akeneo from archive on Debian/Ubuntu linux servers.

## Requirements

ansible 2.0+

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```
akeneo_project_path: /var/www/akeneo/live
akeneo_install_path: '{{ akeneo_project_path }}/releases/automatic'
akeneo_unarchive_path: '{{ akeneo_install_path }}/pim-community-standard'
akeneo_force_install: false
akeneo_archive_url: https://download.akeneo.com/pim-community-standard-v2.3-latest.tar.gz #for minimal version

akeneo_database_driver: pdo_mysql
akeneo_database_host: localhost
akeneo_database_port: null
akeneo_database_name: akeneo_pim
akeneo_database_user: akeneo_pim
akeneo_database_password: akeneo_pim
akeneo_locale: de
akeneo_secret: X5J6Wk3XDU9xPOuFf6n2chPd
akeneo_product_index_name: akeneo_pim_product
akeneo_product_model_index_name: product_model_index_name
akeneo_product_and_product_model_index_name: akeneo_pim_product_and_product_model
akeneo_index_hosts: 'localhost:9200'

akeneo_owner: www-data
akeneo_group: www-data
```

## Dependencies

None.

## Requirements

```
- src: tschifftner.nodejs
- src: tschifftner.yarn
- src: tschifftner.elasticsearch
- src: tschifftner.nginx
- src: tschifftner.php7-fpm
- src: tschifftner.composer
- src: tschifftner.mysql
- src: tschifftner.akeneo
```

## Installation

```
$ ansible-galaxy install \ 
    tschifftner.akeneo \ 
    tschifftner.nodejs \
    tschifftner.yarn \
    tschifftner.elasticsearch \
    tschifftner.nginx \
    tschifftner.php7-fpm \
    tschifftner.composer \
    tschifftner.mysql
```

## Example Playbook

Webserver setup must be done prior to running this role. 

```
- hosts: localhost
  remote_user: root

  vars:
    # AKENEO
    akeneo_project_path: /var/www/akeneo/live
    akeneo_database_host: localhost
    akeneo_database_name: akeneo_pim
    akeneo_database_user: akeneo_pim
    akeneo_database_password: akeneo_pim

    # NGINX
    nginx_vhosts:
      akeneo-live:
        - server_name: localhost
          listen: 80
          root: '{{ akeneo_project_path }}/releases/current/web/'
          access_log: '{{ akeneo_project_path }}/logs/nginx-access.log'
          error_log: '{{ akeneo_project_path }}/logs/nginx-error.log'
          extra_parameters: |

            set $socket unix:/run/akeneo-live.socket;

            location / {
                try_files $uri /app.php$is_args$args;
            }

            location ~ (app|app_dev|config)\.php$ {
                try_files $uri =404;
                fastcgi_pass    $socket;
                fastcgi_buffers 1024 4k;

                fastcgi_param  PHP_FLAG  "session.auto_start=off \n suhosin.session.cryptua=off";
                fastcgi_read_timeout 600s;
                fastcgi_connect_timeout 600s;

                fastcgi_index  index.php;
                fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                include        fastcgi_params;
            }

            location ~ \.php$ {
                deny all;
            }

            location ~ /\. {
                log_not_found off;
                deny all;
            }
    nginx_dhparam_size: 32

    # PHP
    php7_fpm_version: '7.1'
    php7_fpm_ini_memory_limit: 1024M
    php7_fpm_ini_max_execution_time: 300
    php7_fpm_ini_max_input_time: 60
    php7_fpm_ini_max_input_vars: 1000
    php7_fpm_ini_realpath_cache_size: 64K
    php7_fpm_ini_realpath_cache_ttl: 120
    php7_fpm_ini_upload_max_filesize: 64M
    php7_cli_ini_memory_limit: 1024M
    php7_cli_ini_max_execution_time: 300
    php7_cli_ini_max_input_time: 60
    php7_cli_ini_max_input_vars: 1000
    php7_cli_ini_realpath_cache_size: 64K
    php7_cli_ini_realpath_cache_ttl: 120
    php7_cli_ini_upload_max_filesize: 64M
    php7_fpm_pools:
      - name: 'akeneo_live'
        user: 'www-data'
        group: 'www-data'
        prefix: '{{ akeneo_project_path }}'
        logs_path: '/var/log/akeneo/live/'
        project_paths:
          - '{{ akeneo_project_path }}/releases'
          - '{{ akeneo_project_path }}/shared'
          - '{{ akeneo_project_path }}/logs'
        socket_file: '/run/akeneo-live.socket'

    # MYSQL
    mysql_admin_user: 'admin'
    mysql_admin_password: 'travis'
    mysql_debug: true

    mysql_users:
      - name: '{{ akeneo_database_user }}'
        hosts:
          - '{{ akeneo_database_host }}'
          - 127.0.0.1
        password: '{{ akeneo_database_user }}'
        priv: '{{ akeneo_database_name }}.*:ALL'

    mysql_databases:
      - name: '{{ akeneo_database_name }}'
        collation: utf8_general_ci
        encoding: utf8

  pre_tasks:
    - name: Upgrade system
      apt:
        upgrade: safe
        update_cache: yes
        cache_valid_time: 86400

  roles:
    - tschifftner.nodejs
    - tschifftner.yarn
    - tschifftner.mysql
    - tschifftner.elasticsearch
    - tschifftner.php7-fpm
    - tschifftner.nginx
    - tschifftner.composer
    - tschifftner.akeneo
```

## Supported OS

 - Debian 9 (Stretch)
 - Debian 8 (Jessie)
 - Ubuntu 18.04 (Bionic Beaver)
 - Ubuntu 16.04 (Xenial Xerus)
 
## Required ansible version

Ansible 2.5+

## License

[MIT License](http://choosealicense.com/licenses/mit/)

## Author Information

 - [Tobias Schifftner](https://twitter.com/tschifftner), [ambimax® GmbH](https://www.ambimax.de)