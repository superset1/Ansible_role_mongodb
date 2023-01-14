Ansible role for MongoDB 
===========

Version v1.3.1

## Content
------------
- [General info](#general-info)
  - [Feature](#feature)
  - [Requirements](#requirements)
  - [Tags](#tags)
- [MongoDB support matrix](#mongodb-support-matrix)
- [Variables](#variables)
- [Usage](#usage)
  - [Prepare system environment variables](#prepare-system-environment-variables)
  - [Standalone](#standalone)
    - [Example hosts file without replication](#example-hosts-file-without-replication)
    - [Example var file without replication with required variables](#example-var-file-without-replication-with-required-variables)
  - [Replicaset](#replicaset)
    - [Example hosts file for replication](#example-hosts-file-for-replication)
    - [Example var file for replication](#example-var-file-for-replication)
  - [Sharding](#sharding)
    - [Example hosts file for sharded cluster](#example-hosts-file-for-sharded-cluster)
    - [Example var file for sharded cluster](#example-var-file-for-sharded-cluster)

## General info

Ansible role which manages [MongoDB](http://www.mongodb.org/)

- Install and configure the MongoDB
- Configure mongodb users
- Configure logrotare
- Configure replication
- Configure sharding
- Configure arbiter
- Provide handlers for restart and reload
- Setup MMS automation agent
- Setup mongodb-exporter prometheus metrics

### Feature
- Supported versions MongoDB: 3.4, 3.6, 4.0, 4.2, 4.4, 5.0, 6.0
- Supports creation users on worked base without crash
- Supports vault
- Supports replicaset
- Supports sharding
- Supports arbiter
- Idempotency: the role does not reload the working database
- Able to remove everything related to MongoDB from the server

### Requirements
- pip install hvac (if need to use vault module)
- ansible-galaxy collection install community.mongodb (check that `community.mongodb` module >= 1.4)
- run AVX if `mongodb_version` >= 5.0
- run `openssl rand -base64 756` and fill the `mongodb_keyfile_content` variable for MongoDB cluster or sharding
- set admins passwords
  - `mongodb_user_admin_password`
  - `mongodb_root_admin_password`
  - `mongodb_root_backup_password`
  - `mongodb_exporter_password`
- create groups in the hosts file
  - `mongo_standalone` for standalone server
  - `mongo_cluster` for replicaset cluster
  - `mongocfg_servers` for sharded cluster (config servers)
  - `mongos_servers` for sharded cluster (mongos or in other words router servers)
  - `mongo_shard_[0-9]` for sharded cluster (replicaset shards)

### Tags
- mongodb (main tag for all mongodb tasks | optional tag)
- mongodb-install (install mongodb only)
- mongodb-configure (initial mongodb configure)
- mongodb-logrotate (configure mongodb logrotate)
- mongodb-replicaset (configure replicaset when `mongodb_replication_enabled` is True)
- mongodb-create-admin-users (create initial users)
- mongodb-create-oplog-users (create oplog users)
- mongodb-add-users (add normal users anytime even on worked prod mongodb)
- mongodb-mms (install mms agent on mongodb)
- mongodb-exporter (install mongodb-exporter)
- mongodb-uninstall (uninstall mongodb, mongodb-exporter, but don't delete database)
- mongodb-uninstall-exporter (uninstall mongodb-exporter, but don't delete database)
- mongodb-dbdelete (delete database)
- mongos (main tag for all mongos tasks | optional tag)
- mongos-install (install mongos only)
- mongos-configure (initial mongos configure)
- mongos-logrotate (configure mongos logrotate)
- mongos-sharding (configure shards)
- mongos-mms (install mms agent on mongos)
- mongos-exporter (install mongodb-exporter)
- mongos-uninstall (uninstall mongos, mongos-exporter, but don't delete database)
- mongos-uninstall-exporter (uninstall mongos-exporter, but don't delete database)

## MongoDB support matrix:

| Distribution   | < MongoDB 3.2 |    MongoDB 3.4     |    MongoDB 3.6     |    MongoDB 4.0     |   MongoDB 4.2      |   MongoDB 4.4-6.0  |
| -------------- | :-----------: | :----------------: | :----------------: | :----------------: | :----------------: | :----------------: |
| Ubuntu 16.04   |  :no_entry:   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| Ubuntu 18.04   |  :no_entry:   |        :x:         |        :x:         | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| Ubuntu 20.04   |  :no_entry:   |        :x:         |        :x:         |        :x:         |        :x:         | :white_check_mark: |
| Debian 8.x     |  :no_entry:   | :white_check_mark: | :white_check_mark: | :white_check_mark: |        :x:         |        :x:         |
| Debian 9.x     |  :no_entry:   |        :x:         | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| Debian 10.x    |  :no_entry:   |        :x:         |        :x:         |        :x:         | :white_check_mark: | :white_check_mark: |
| RHEL 6.x       |  :no_entry:   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| RHEL 7.x       |  :no_entry:   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| RHEL 8.x       |  :no_entry:   |        :x:         | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| Amazon Linux 2 |  :no_entry:   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |

:exclamation: **MongoDB version >= 5.0 requires use of the AVX instruction set, available on select Intel and AMD processors**

Для запуска сервиса mongodb ver >= 5.0 требуется поддержка avx инструкций на уровне CPU

- :white_check_mark: - fully tested, should works fine
- :x: - don't have official support
- :no_entry: - MongoDB has reached EOL

## Variables
```yaml
# You can use this variable to control installation source of MongoDB
# 'mongodb' will be installed from Debian/Ubuntu repos
# 'mongodb-org' will be installed from MongoDB official repos

# You can control installed version via this param.
# Should be '3.4', '3.6', '4.0', '4.2', '4.4', '5.0', or '6.0'. This role doesn't support MongoDB < 3.4.
# I will recommend you to use latest version of MongoDB.

## Main options
ansible_user: root
mongodb_package: "mongodb-org"
mongodb_package_state: "present"
mongodb_version: "4.4"

mongodb_pymongo_from_pip: true                   # Install latest PyMongo via PIP or package manager
mongodb_pymongo_pip_version: 4.2.0               # Choose PyMong version to install from pip. If not set use latest

mongodb_admin_update_password: false             # Update password every play if true
mongodb_user_update_password: false              # Update password every play if true
mongodb_manage_service: true
mongodb_systemd_unit_limit_nofile: 64000
mongodb_systemd_unit_limit_nproc: 64000

mongodb_disable_transparent_hugepages: false

mongodb_use_numa: false

mongodb_login_database: "admin"

mongodb_user: "{{ 'mongod' if ('RedHat' == ansible_os_family) else 'mongodb' }}"
mongodb_group: "{{ 'mongod' if ('RedHat' == ansible_os_family) else 'mongodb' }}"
mongodb_daemon_name: "{{ 'mongod' if ('mongodb-org' in mongodb_package) else 'mongodb' }}"
  
## Net options
mongodb_net_bindip: 0.0.0.0                      # Comma separated list of ip addresses to listen on
mongodb_net_http_enabled: false                  # Enable http interface
mongodb_net_ipv6: false                          # Enable IPv6 support (disabled by default)
mongodb_net_maxconns: 65536                      # Max number of simultaneous connections
mongodb_net_port: 27017                          # Specify port number
mongodb_net_ssl_enabled: false                   # Enable or disable ssl connections
mongodb_net_ssl_mode: ""                         # Set the ssl mode (requireSSL / preferSSL / allowSSL / disabled)
mongodb_net_ssl_pemfile: ""                      # Location of the pemfile to use for ssl

## ProcessManagement options
# Fork server process
# Enabled by default for RedHat as the init scripts assume forking is enabled.
mongodb_processmanagement_fork: "{{ 'RedHat' == ansible_os_family }}"

## Security options
# Disable or enable security. Possible values: 'disabled', 'enabled'
mongodb_security_authorization_enabled: true
mongodb_security_javascript_enabled: false
mongodb_security_keyfile_path: /etc/mongodb-keyfile       # Specify path to keyfile with password for inter-process authentication
mongodb_keyfile_content: ""                      # Please generate this file on production environment with command 'openssl rand -base64 756'

## Storage options
mongodb_storage_dbpath: /data/db                 # Directory for datafiles
mongodb_storage_dirperdb: false                  # Use one directory per DB

# The storage engine for the mongod database
mongodb_storage_engine: "wiredTiger"
# mmapv1 specific options
mongodb_storage_quota_enforced: false            # Limits each database to a certain number of files
mongodb_storage_quota_maxfiles: 8                # Number of quota files per DB
mongodb_storage_smallfiles: false                # Very useful for non-data nodes

mongodb_storage_journal_enabled: true            # Enable journaling
mongodb_storage_prealloc: true                   # Enable data file preallocation

mongodb_wiredtiger_directory_for_indexes: false

## SystemLog options
## The destination to which MongoDB sends all log output. Specify either 'file' or 'syslog'.
## If you specify 'file', you must also specify mongodb_systemlog_path.
mongodb_systemlog_destination: "file"
mongodb_systemlog_logappend: true                                        # Append to logpath instead of over-writing
mongodb_systemlog_logrotate: "rename"                                    # Logrotation behavior
mongodb_systemlog_path: /var/log/mongodb/{{ mongodb_daemon_name }}.log   # Log file to send write to instead of stdout
mongodb_systemlog_logrotate_config:
  period: daily
  size: 1G
  rotate: 5
  create: true
  missingok: true
  compress: true
  delaycompress: true
  notifempty: true
  sharedscripts: true
  postrotate: /bin/kill -SIGUSR1 $(pidof {{ mongodb_daemon_name }}) && rm -f {{ mongodb_systemlog_path }}.20*

## Operation Profiling options
mongodb_operation_profiling_slow_op_threshold_ms: 100
mongodb_operation_profiling_mode: "off"

## Cloud options (MongoDB >= 4.0)
mongodb_cloud_monitoring_free_state: "runtime"

## MongoDB Standalone options
mongodb_standalone_host_group: "mongo_standalone"

## Replication options
mongodb_replication_enabled: "{{ true if (mongodb_replication_host_group in mongodb_main_group or mongodb_sharding_host_group in mongodb_main_group or mongodb_config_host_group in mongodb_main_group) else false }}"  # Enable replication
mongodb_replication_host_group: "mongo_cluster"
mongodb_replication_replset: "{{ ('rs' + mongodb_main_group.split('_')[-1] if mongodb_sharding_host_group in mongodb_main_group else mongodb_config_replication_replset_name if mongodb_main_group == mongodb_config_host_group else '') if mongodb_sharding_enabled else 'rs01' if mongodb_main_group == mongodb_replication_host_group else '' }}"      # Default name of replicaset
mongodb_replication_replindexprefetch: "all"                                            # Specify index prefetching behavior (if secondary) [none|_id_only|all]
mongodb_replication_oplogsize: 16384                                                    # Specifies a maximum size in megabytes for the replication operation log
mongodb_replication_reconfigure: false                                                  # Reconfigure replicaset for add or delete members

## Sharding options
mongodb_sharding_state: "present"
mongodb_sharding_host_group: "mongo_shard_"

## Mongocfg options
mongodb_config_host_group: "mongocfg_servers"
mongodb_config_replication_replset_name: "cfg"

## Mongos options
mongos_host_group: "mongos_servers"
mongos_daemon_name: "mongos"
mongos_package: "mongodb-org-mongos"
mongos_package_state: "present"

mongos_user: "{{ 'mongod' if ('RedHat' == ansible_os_family) else 'mongodb' }}"
mongos_group: "{{ 'mongod' if ('RedHat' == ansible_os_family) else 'mongodb' }}"

mongos_systemlog_destination: "file"
mongos_systemlog_logappend: true
mongos_systemlog_logrotate: "rename"
mongos_systemlog_path: "/var/log/mongodb/mongos.log"
mongos_systemlog_logrotate_config:
  period: daily
  size: 1G
  rotate: 5
  create: true
  missingok: true
  compress: true
  delaycompress: true
  notifempty: true
  sharedscripts: true
  postrotate: /bin/kill -SIGUSR1 $(pidof {{ mongos_daemon_name }}) && rm -f {{ mongos_systemlog_path }}.20*

mongos_systemd_unit_limit_nofile: 64000
mongos_systemd_unit_limit_nproc: 64000

mongos_net_port: 27017                                   # Specify port number
mongos_net_bindip: 0.0.0.0                               # Comma separated list of ip addresses to listen on
mongos_net_bind_ip_all: false
mongos_net_tls_enabled: false
mongos_net_compressors: null
mongos_certificate_key_file: ""
mongos_certificate_ca_file: ""
mongos_security_keyfile_path: "{{ mongodb_security_keyfile_path }}"        # Specify path to keyfile with password for inter-process authentication
mongos_keyfile_content: "{{ mongodb_keyfile_content }}"  # Please generate this file on production environment with command 'openssl rand -base64 756'

# MMS Agent
mongodb_mms_agent_pkg: https://cloud.mongodb.com/download/agent/monitoring/mongodb-mms-monitoring-agent_7.2.0.488-1_amd64.ubuntu1604.deb
mongodb_mms_group_id: ""
mongodb_mms_api_key: ""
mongodb_mms_base_url: https://mms.mongodb.com

# Names and passwords for administrative users
mongodb_user_admin_name: "mongoadm"
mongodb_user_admin_password: ""

mongodb_root_admin_name: "mongoroot"
mongodb_root_admin_password: ""

mongodb_root_backup_name: "mongobackup"
mongodb_root_backup_password: ""

mongodb_exporter_name: "mongodbexporter"
mongodb_exporter_password: ""

mongodb_users: # Optional: If you want to add multiple regular users
  - {
    name: "",
    password: "",
    roles: "",
    database: ""
    }

mongodb_oplog_users: # Optional: If you want to add multiple oplog users
  - {
    name: "",
    password: ""
    }

# Set parameter config
mongodb_set_parameters:

# Custom config options
mongodb_config:

# MongoDB prometheus exporter
mongodb_exporter_user: "mongodb-exporter"
mongodb_exporter_group: "{{ mongodb_exporter_user }}"
mongodb_exporter_enabled: "true"
mongodb_exporter_version: "{{ '0.30.0' if mongodb_sharding_enabled else '0.11.2' }}"
mongodb_exporter_link: "https://github.com/percona/mongodb_exporter/releases/download/v{{ mongodb_exporter_version }}/mongodb_exporter-{{ mongodb_exporter_version }}.{{ os }}-{{ bin_arch }}.tar.gz"
mongodb_exporter_path: "/usr/local/bin/mongodb-exporter"
mongodb_exporter_checksum_link: "https://github.com/percona/mongodb_exporter/releases/download/v{{ mongodb_exporter_version }}/mongodb_exporter_{{ mongodb_exporter_version }}_checksums.txt"
mongodb_exporter_checksum_path: "{{ mongodb_exporter_temp_dir }}/mongodb_exporter_v{{ mongodb_exporter_version }}_SHA256SUMS"
mongodb_exporter_temp_dir: "/tmp/mongodb_exporter"
```

## Usage

Add `mongodb` to your roles and set vars in your playbook file (see tests/group_vars).

### Prepare system environment variables
```
export VAULT_ADDR=https://vault.example.com:8200
export VAULT_TOKEN=your_deploy_token
# For MacOs
export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES
```

## Standalone

### Example hosts file without replication
```ini
[mongo_standalone]
www.host1.com 
```

### Example var file without replication with required variables
```yaml
mongodb_user_admin_password: "mongoadm"
mongodb_root_admin_password: "mongoroot"
mongodb_root_backup_password: "mongobackup"
mongodb_exporter_password: "mongodbexporter" # Required if variable mongodb_exporter_enabled is True

mongodb_users: # Optional if you want to add multiple regular users
  - {
    name: user1,
    password: "password1",
    roles: readWrite,
    database: database1
    }
  - {
    name: user2,
    password: "password2",
    roles: read,
    database: database2
    }

```

## Replicaset

### Example hosts file for replication

```ini
[mongo_cluster]
www.host1.com mongodb_master=True # Optional variable, because if it's not set, then the 1st host will be master by default when mongodb_replication_enabled
www.host2.com
www.host3.com
www.host4.com
www.host5.com mongodb_arbiter=True # Optional variable if you need arbiter
```

### Example var file for replication
```yaml
mongodb_version: "6.0"
mongodb_replication_enabled: true

mongodb_root_admin_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_root_admin_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_user_admin_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_user_admin_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_root_backup_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_root_backup_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_exporter_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_exporter_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_keyfile_content: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_keyfile_content token={{ vault_token }} url={{ vault_url }}') }}"

mongodb_users: # Optional if you want to add multiple regular users
  - {
    name: user1,
    password: "password1",
    roles: readWrite,
    database: database1
    }
  - {
    name: user2,
    password: "password2",
    roles: read,
    database: database2
    }
    
mongodb_oplog_users: # Optional if you want to add oplog user
  - {
    name: oplog,
    password: oplog_password
    }
```

## Sharding

### Example hosts file for sharded cluster

```ini
[mongocfg_servers]
www.mongocfg-1.com
www.mongocfg-2.com
www.mongocfg-3.com

[mongos_servers]
www.mongos-1.com
www.mongos-2.com
www.mongos-3.com

[mongo_shard_01]
www.mongoshard-1-1.com
www.mongoshard-1-2.com
www.mongoshard-1-3.com

[mongo_shard_02]
www.mongoshard-2-1.com
www.mongoshard-2-2.com
www.mongoshard-2-3.com mongodb_arbiter=True # Optional variable if you need arbiter

[mongo_sharding_cluster:children]
mongocfg_servers
mongos_servers
mongo_shard_01
mongo_shard_02
```

### Example var file for sharded cluster
```yaml
mongodb_version: "6.0"
mongodb_replication_enabled: true

mongodb_root_admin_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_root_admin_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_user_admin_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_user_admin_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_root_backup_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_root_backup_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_exporter_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_exporter_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_keyfile_content: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_keyfile_content token={{ vault_token }} url={{ vault_url }}') }}"

mongodb_users: # Optional if you want to add multiple regular users
  - {
    name: user1,
    password: "password1",
    roles: readWrite,
    database: database1
    }
  - {
    name: user2,
    password: "password2",
    roles: read,
    database: database2
    }
    
mongodb_oplog_users: # Optional if you want to add oplog user
  - {
    name: oplog,
    password: oplog_password
    }
```

## Other examples

### Adding normal users to the worked production database

To add users, simply add a new user (e.g. `User3`) to your var file and run the playbook with the `mongodb-add-users` tag. It's safe even in production.
```yaml
mongodb_users:
  - {
    name: user1,
    password: "password1",
    roles: readWrite,
    database: database1
    }
  - {
    name: user2,
    password: "password2",
    roles: read,
    database: database2
    }
  - {
    name: user3,
    password: "password3",
    roles: readWrite,
    database: database3
    }
```

**See worked example in tests/ directory**

## License

MIT
