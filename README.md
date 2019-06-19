Cloudweeb Polemarch
=========

your description

Requirements
------------

  - MariaDB-devel
  - MariaDB-shared

Role Variables
--------------

```YAML
polemarch_user: polemarch
polemarch_group: '{{ polemarch_user }}'

polemarch_main_dir: /opt/polemarch

polemarch_base_packages:
  - polemarch
  - mysqlclient

polemarch_db_name: polemarch_db
polemarch_db_user: polemarch_user
polemarch_db_pass: password

polemarch_system_dir:
  - '{{ polemarch_main_dir }}/logs'
  - '{{ polemarch_main_dir }}/pid'
  - '{{ polemarch_main_dir }}/projects'
  - '{{ polemarch_main_dir }}/hooks'
  - /etc/polemarch

polemarch_configuration_maps:
  - '{{ polemarch_main_config }}'
  - '{{ polemarch_database_config }}'
  - '{{ polemarch_database_options_config }}'
  - '{{ polemarch_uwsgi_config }}'
  - '{{ polemarch_worker_config }}'
  - '{{ polemarch_cache_config }}'
  - '{{ polemarch_locks_config }}'
  - '{{ polemarch_rpc_config }}'

polemarch_main_config:
  section: main
  options:
    projects_dir: /opt/polemarch/projects
    hooks_dir: /opt/polemarch/hooks

polemarch_database_config: {}
# section: database
# options:
#   engine: django.db.backends.mysql
#   name: '{{ polemarch_db_name }}'
#   user: '{{ polemarch_db_user }}'
#   password: '{{ polemarch_db_pass }}'

polemarch_database_options_config:
  section: database.options
  options:
    connect_timeout: 20
    init_command: SET sql_mode='STRICT_TRANS_TABLES', default_storage_engine=INNODB, NAMES 'utf8', CHARACTER SET 'utf8', SESSION collation_connection = 'utf8_unicode_ci'

polemarch_uwsgi_config:
  section: uwsgi
  options:
    processes: 4
    threads: 4
    harakiri: 120
    vacuum: True
    pidfile: '{{ polemarch_main_dir }}/pid/polemarch.pid'
    log_file: '{{ polemarch_main_dir }}/logs/{PROG_NAME}_web.log'

polemarch_worker_config:
  section: worker
  options:
    logfile: '{{ polemarch_main_dir }}/logs/{PROG_NAME}_worker.log'
    pidfile: '{{ polemarch_main_dir }}/pid/{PROG_NAME}_worker.pid'
    loglevel: INFO

polemarch_cache_config: {}
# section: cache
# options:
#   backend: django_redis.cache.RedisCache
#   location: redis://127.0.0.1:6379/1

polemarch_locks_config: {}
# section: locks
# options:
#   backend: django_redis.cache.RedisCache
#   location: redis://127.0.0.1:6379/2

polemarch_rpc_config: {}
# section: rpc
# options:
#   connection:
#   heartbeat: 5
#   concurrency: 8
#   enable_worker: true
```

Dependencies
------------

- geerlingguy.repo-epel
- cloudweeb.mariadb
- geerlingguy.redis (Optional)
- cloudweeb.polemarch

Example Playbook
----------------

Installing Polemarch with standard installation
```YAML
---
- name: Converge
  hosts: all

  vars:

    polemarch_db_name: polemarch_db
    polemarch_db_user: polemarch_user
    polemarch_db_pass: password

    polemarch_database_config:
      section: database
      options:
        engine: django.db.backends.mysql
        name: '{{ polemarch_db_name }}'
        user: '{{ polemarch_db_user }}'
        password: '{{ polemarch_db_pass }}'

    mariadb_root_password: "{{ lookup('password', '/tmp/mariadb_root_password length=15 chars=ascii_letters,digits,hexdigits') }}"

    mariadb_packages:
      - MariaDB-shared

    mariadb_databases:
      - name: '{{ polemarch_db_name }}'
        state: present

    mariadb_users:
      - name: '{{ polemarch_db_user }}'
        password: '{{ polemarch_db_pass }}'
        priv: "{{ polemarch_db_name }}.*:ALL"
        state: present

  roles:

    - role: geerlingguy.repo-epel
      when: ansible_os_family == 'RedHat'

    - cloudweeb.mariadb

    - cloudweeb.polemarch
```

Installing Polemarch with redis cache
```YAML
---
- name: Converge
  hosts: all

  vars:

    polemarch_db_name: polemarch_db
    polemarch_db_user: polemarch_user
    polemarch_db_pass: password

    polemarch_database_config:
      section: database
      options:
        engine: django.db.backends.mysql
        name: '{{ polemarch_db_name }}'
        user: '{{ polemarch_db_user }}'
        password: '{{ polemarch_db_pass }}'

    mariadb_root_password: "{{ lookup('password', '/tmp/mariadb_root_password length=15 chars=ascii_letters,digits,hexdigits') }}"

    mariadb_packages:
      - MariaDB-shared

    mariadb_databases:
      - name: '{{ polemarch_db_name }}'
        state: present

    mariadb_users:
      - name: '{{ polemarch_db_user }}'
        password: '{{ polemarch_db_pass }}'
        priv: "{{ polemarch_db_name }}.*:ALL"
        state: present

    polemarch_cache_config:
      section: cache
      options:
        backend: django_redis.cache.RedisCache
        location: redis://127.0.0.1:6379/1

  roles:

    - role: geerlingguy.repo-epel
      when: ansible_os_family == 'RedHat'

    - cloudweeb.mariadb

    - geerlingguy.redis

    - cloudweeb.polemarch
```

License
-------

MIT

Author Information
------------------

Agnesius Santo Naibaho
