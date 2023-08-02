Ansible role for MongoDB 
===========

Version v1.5.6

## Content
------------
- [General info](#general-info)
  - [What's new](#whats-new-in-v156)
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
  - [Other examples](#other-examples)
    - [Adding and deleting members in the replicaset](#adding-and-deleting-members-in-the-replicaset)
    - [Adding normal users to the worked production database](#adding-normal-users-to-the-worked-production-database)
    - [Deleting normal users from the worked production database](#deleting-normal-users-from-the-worked-production-database)
    - [Updating passwords for admins and normal users](#updating-passwords-for-admins-and-normal-users)
  - [License](#license)

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

### What's new in v1.5.6
- Added ckeck for installed mongodb prometheus exporter version
- Added variable `mongodb_storage_journal_commitIntervalMs` to default
- Added variable `mongodb_exporter_force_install: true`
- Converted variable `mongodb_exporter_enabled` to bool
- Deleted `become` from tasks
- Fixed check that logfile exists - don't calculate checksum

### Feature
- Supported versions MongoDB: 3.4, 3.6, 4.0, 4.2, 4.4, 5.0, 6.0
- Supports creation users on worked base without crash
- Supports vault
- Supports replicaset
- Supports sharding
- Supports arbiter
- Supports SSL/TLS
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
- set required values for keys in `mongodb_net_tls_config` dict, when `mongodb_net_tls_enabled: true`
  - `mode`
  - `certificateKeyFileContent` or `certificateSelector`
  - `CAFileContent`

### Tags
- mongodb (main tag for all mongodb tasks | optional tag)
- mongodb-install (install mongodb only)
- mongodb-configure (initial mongodb configure)
- mongodb-logrotate (configure mongodb logrotate)
- mongodb-replicaset (configure replicaset)
- mongodb-create-admin-users (create initial users)
- mongodb-create-oplog-users (create oplog users)
- mongodb-add-users (add normal users anytime even on worked prod mongodb)
- mongodb-force-restart (restart service mongodb | default is start only)
- mongos (main tag for all mongos tasks | optional tag)
- mongos-install (install mongos only)
- mongos-configure (initial mongos configure)
- mongos-logrotate (configure mongos logrotate)
- mongos-sharding (configure shards)
- mongos-force-restart (restart service mongos | default is start only)
- mongodb-mms (install mms agent)
- mongodb-exporter (install mongodb-exporter)
- mongodb-uninstall (uninstall mongodb, mongodb-exporter, but don't delete database)
- mongodb-uninstall-exporter (uninstall mongodb-exporter, but don't delete database)
- mongodb-dbdelete (delete database)
- mongos-uninstall (uninstall mongos, mongodb-exporter, but don't delete database)

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
mongodb_daemon_name: "{{ 'mongod' if ('mongodb-org' in mongodb_package) else 'mongodb' }}"
mongodb_package: "mongodb-org"
mongodb_package_state: "present"
mongodb_version: "4.4"                           # Default MongoDB version

mongodb_pymongo_from_pip: true                   # Install latest PyMongo via PIP or package manager
mongodb_pymongo_pip_version: 4.2.0               # Choose PyMong version to install from pip. If not set use latest

mongodb_admin_update_password: false             # Update admin passwords every play if true
mongodb_user_update_password: false              # Update normal user passwords every play if true
mongodb_manage_service: true

mongodb_systemd_unit_limit_nofile: 64000
mongodb_systemd_unit_limit_nproc: 64000

mongodb_disable_transparent_hugepages: false

mongodb_use_numa: false

mongodb_login_database: "admin"

mongodb_user: "{{ 'mongod' if ('RedHat' == ansible_os_family) else 'mongodb' }}"
mongodb_group: "{{ 'mongod' if ('RedHat' == ansible_os_family) else 'mongodb' }}"
  
## Net options
mongodb_net_bindip: 0.0.0.0                      # Comma separated list of ip addresses to listen on
mongodb_net_bind_ip_all: false
mongodb_net_http_enabled: false                  # Enable http interface
mongodb_net_ipv6: false                          # Enable IPv6 support (disabled by default)
mongodb_net_maxconns: 65536                      # Max number of simultaneous connections
mongodb_net_port: 27017                          # Specify port number
mongodb_net_ssl_enabled: false                   # Enable or disable SSL connections (mongodb_net_ssl_enabled and mongodb_net_tls_enabled options are mutually exclusive. You can only specify one)
mongodb_net_ssl_config:                          # MongoDB SSL config is used if MongoDB version < 4.2
    mode: "" # <requireSSL / preferSSL / allowSSL / disabled> Enables SSL used for all network connections. The argument to the net.ssl.mode setting can be only one.
    certificateSelector: "" # Specifies a certificate property in order to select a matching certificate from the operating system's certificate store to use for TLS/SSL (net.ssl.PEMKeyFile and net.ssl.certificateSelector options are mutually exclusive. You can only specify one).
    PEMKeyFileContent: "" # The .pem content with both the SSL certificate and key (net.ssl.PEMKeyFile and net.ssl.certificateSelector options are mutually exclusive. You can only specify one).
    PEMKeyPassword: "" # The password to de-crypt the certificate-key file (i.e. PEMKeyFile). Use the net.ssl.PEMKeyPassword option only if the certificate-key file is encrypted. In all cases, the mongos or mongod will redact the password from all logging and reporting output.
    CAFileContent: "" # The .pem content with the root certificate chain from the Certificate Authority.
    CRLFileContent: "" # The .pem content with the Certificate Revocation List.
    clusterCertificateSelector: "" # Specifies a certificate property to select a matching certificate from the operating system's secure certificate store to use for internal x.509 membership authentication (net.ssl.clusterFile and net.ssl.clusterCertificateSelector options are mutually exclusive. You can only specify one).
    clusterFileContent: "" # The .pem content with the x.509 certificate-key file for membership authentication for the cluster or replica set (net.ssl.clusterFile and net.ssl.clusterCertificateSelector options are mutually exclusive. You can only specify one).
    clusterPassword: "" # The password to de-crypt the x.509 certificate-key file specified with --sslClusterFile. Use the net.ssl.clusterPassword option only if the certificate-key file is encrypted. In all cases, the mongos or mongod will redact the password from all logging and reporting output.
    clusterCAFileContent: "" # The .pem content with the root certificate chain from the Certificate Authority used to validate the certificate presented by a client establishing a connection. net.ssl.clusterCAFile requires that net.ssl.CAFile is set.
    allowConnectionsWithoutCertificates: # <true|false> For clients that don't provide certificates, mongod or mongos encrypts the TLS/SSL connection, assuming the connection is successfully made.
    allowInvalidCertificates: # <true|false> Enable or disable the validation checks for SSL certificates on other servers in the cluster and allows the use of invalid certificates to connect.
    allowInvalidHostnames: # <true|false> When net.ssl.allowInvalidHostnames is true, MongoDB disables the validation of the hostnames in TLS/SSL certificates, allowing mongod to connect to MongoDB instances if the hostname their certificates do not match the specified hostname. 
    FIPSMode: # <true|false> Enable or disable the use of the FIPS mode of the SSL library for the mongos or mongod. Your system must have a FIPS compliant library to use the net.ssl.FIPSMode option.
    disabledProtocols: "" # <TLS1_0 / TLS1_1 / TLS1_2 / none> Prevents a MongoDB server running with TLS/SSL from accepting incoming connections that use a specific protocol or protocols. To specify multiple protocols, use a comma separated list of protocols.
  
mongodb_net_tls_enabled: false                   # Enable or disable TLS connections (mongodb_net_ssl_enabled and mongodb_net_tls_enabled options are mutually exclusive. You can only specify one)
mongodb_net_tls_config:                          # MongoDB TLS config is used if MongoDB version >= 4.2
  mode: "" # <requireTLS / preferTLS / allowTLS / disabled> Enables TLS used for all network connections. The argument to the net.tls.mode setting can be only one.
  certificateSelector: "" # Specifies a certificate property in order to select a matching certificate from the operating system's certificate store to use for TLS/SSL (net.tls.certificateKeyFile and net.tls.certificateSelector options are mutually exclusive. You can only specify one).
  certificateKeyFileContent: "" # The .pem content with both the TLS certificate and key (net.tls.certificateKeyFile and net.tls.certificateSelector options are mutually exclusive. You can only specify one).
  certificateKeyFilePassword: "" # The password to de-crypt the certificate-key file (i.e. certificateKeyFile). Use the net.tls.certificateKeyFilePassword option only if the certificate-key file is encrypted. In all cases, the mongos or mongod will redact the password from all logging and reporting output.
  CAFileContent: "" # The .pem content with the root certificate chain from the Certificate Authority.
  CRLFileContent: "" # The .pem content with the Certificate Revocation List.
  clusterCertificateSelector: "" # Specifies a certificate property to select a matching certificate from the operating system's secure certificate store to use for internal x.509 membership authentication (net.tls.clusterFile and net.tls.clusterCertificateSelector options are mutually exclusive. You can only specify one).
  clusterFileContent: "" # The .pem content with the x.509 certificate-key file for membership authentication for the cluster or replica set (net.tls.clusterFile and net.tls.clusterCertificateSelector options are mutually exclusive. You can only specify one).
  clusterPassword: "" # The password to de-crypt the x.509 certificate-key file specified with --sslClusterFile. Use the net.tls.clusterPassword option only if the certificate-key file is encrypted. In all cases, the mongos or mongod will redact the password from all logging and reporting output.
  clusterCAFileContent: "" # The .pem content with the root certificate chain from the Certificate Authority used to validate the certificate presented by a client establishing a connection. net.tls.clusterCAFile requires that net.tls.CAFile is set.
  allowConnectionsWithoutCertificates: # <true|false> For clients that don't provide certificates, mongod or mongos encrypts the TLS/SSL connection, assuming the connection is successfully made.
  allowInvalidCertificates: # <true|false> Enable or disable the validation checks for TLS certificates on other servers in the cluster and allows the use of invalid certificates to connect.
  allowInvalidHostnames: # <true|false> When net.tls.allowInvalidHostnames is true, MongoDB disables the validation of the hostnames in TLS certificates, allowing mongod to connect to MongoDB instances if the hostname their certificates do not match the specified hostname. 
  FIPSMode: # <true|false> Enable or disable the use of the FIPS mode of the TLS library for the mongos or mongod. Your system must have a FIPS compliant library to use the net.tls.FIPSMode option.
  disabledProtocols: "" # <TLS1_0 / TLS1_1 / TLS1_2 / none> Prevents a MongoDB server running with TLS from accepting incoming connections that use a specific protocol or protocols. To specify multiple protocols, use a comma separated list of protocols.
  logVersions: "" # <TLS1_0 / TLS1_1 / TLS1_2 / TLS1_3> Instructs mongos or mongod to log a message when a client connects using a specified TLS version. Specify either a single TLS version or a comma-separated list of multiple TLS versions.

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
mongodb_storage_journal_commitIntervalMs: 100    # The maximum amount of time in milliseconds that the mongod process allows between journal operations. Values can range from 1 to 500 milliseconds. Lower values increase the durability of the journal, at the expense of disk performance. (Default: 100)
mongodb_storage_prealloc: true                   # Enable data file preallocation

mongodb_storage_wiredtiger_cache_size: ""
mongodb_storage_wiredtiger_directory_for_indexes: false

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
mongodb_replication_enabled: "{{ true if (mongodb_replication_host_group in mongodb_main_group or mongodb_sharded_host_group in mongodb_main_group or mongodb_config_host_group in mongodb_main_group) else false }}"  # Enable replication
mongodb_replication_host_group: "mongo_cluster"
mongodb_replication_replset: "{{ ('rs' + mongodb_main_group.split('_')[-1] if mongodb_sharded_host_group in mongodb_main_group else mongodb_config_replication_replset_name if mongodb_main_group == mongodb_config_host_group else '') if mongodb_sharding_enabled else 'rs01' if mongodb_main_group == mongodb_replication_host_group else '' }}"      # Default name of replicaset
mongodb_replication_replindexprefetch: "all"                                            # Specify index prefetching behavior (if secondary) [none|_id_only|all]
mongodb_replication_oplogsize: 4096                                                     # Specifies a maximum size in megabytes for the replication operation log
mongodb_replication_reconfigure: false                                                  # Reconfigure replicaset for add or delete members

## Sharding options
mongodb_sharding_state: "present"                                                       # Adding replicaset to sharding the cluster
mongodb_sharded_databases: []                                                           # List of databases to run command sh.enableSharding()
mongodb_sharded_host_group: "mongo_shard_"                                              # Prefix for shards group in the hosts file

## Mongocfg options
mongodb_config_host_group: "mongocfg_servers"
mongodb_config_replication_replset_name: "cfg"

## Mongos options
mongos_host_group: "mongos_servers"
mongos_daemon_name: "mongos"
mongos_package: "mongodb-org-mongos"
mongos_package_state: "present"
mongos_version: "{{ mongodb_version }}"

mongos_user: "{{ 'mongos' if ('RedHat' == ansible_os_family) else 'mongodb' }}"
mongos_group: "{{ 'mongos' if ('RedHat' == ansible_os_family) else 'mongodb' }}"

mongos_systemlog_destination: "file"
mongos_systemlog_logappend: true
mongos_systemlog_logrotate: "rename"
mongos_systemlog_path: /var/log/mongodb/{{ mongos_daemon_name }}.log
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

mongodb_users: {} # Optional: If you want to add multiple regular users
  # - name: ""
  #   password: ""
  #   roles: ""
  #   database: "" # Optional: (Default: user name)
  #   state: "" # Optional: present|absent (Default: present)
  #   update_password: false # Optional: true|false (Default: false)

mongodb_oplog_users: {} # Optional: If you want to add multiple oplog users
  # - name: ""
  #   password: ""
  #   state: "" # Optional: present|absent (Default: present)
  #   update_password: false # Optional: true|false (Default: false)

# Custom config options
mongodb_config: {}

# MongoDB prometheus exporter
mongodb_exporter_user: "mongodb-exporter"
mongodb_exporter_group: "{{ mongodb_group if mongodb_main_group != mongos_host_group else mongos_group if mongodb_main_group == mongos_host_group else mongodb_exporter_user }}"
mongodb_exporter_enabled: true
mongodb_exporter_force_install: false
mongodb_exporter_version: "0.37.0"
mongodb_exporter_version_arbiter: "0.11.2"
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
mongodb_version: "6.0"

mongodb_net_tls_enabled: true # Optional: true|false (Default: false)
mongodb_net_tls_config:
  mode: "requireTLS"
  certificateKeyFileContent: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:certificateKeyFileContent token={{ vault_token }} url={{ vault_url }}') }}"
  CAFileContent: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:CAFileContent token={{ vault_token }} url={{ vault_url }}') }}"

mongodb_user_admin_password: "mongoadm"
mongodb_root_admin_password: "mongoroot"
mongodb_root_backup_password: "mongobackup"
mongodb_exporter_password: "mongodbexporter" # Required if variable mongodb_exporter_enabled is True

mongodb_users: # Optional if you want to add multiple regular users
  - name: admin
    password: "admin_password"
    roles: readWriteAnyDatabase
    database: admin
  - name: user_rw
    password: "user_rw_password"
    roles: readWrite
    database: user_database
  - name: user_ro
    password: "user_ro_password"
    roles: read
    database: user_database
```

## Replicaset

### Example hosts file for replication

```ini
[mongo_cluster]
www.host1.com mongodb_master=True # Optional variable, because if it's not set, then the 1st host will be master by default
www.host2.com
www.host3.com
www.host4.com
www.host5.com mongodb_arbiter=True # Optional variable if you need arbiter
```

### Example var file for replication
```yaml
mongodb_version: "6.0"

mongodb_net_tls_enabled: true # Optional: true|false (Default: false)
mongodb_net_tls_config:
  mode: "requireTLS"
  certificateKeyFileContent: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:certificateKeyFileContent token={{ vault_token }} url={{ vault_url }}') }}"
  CAFileContent: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:CAFileContent token={{ vault_token }} url={{ vault_url }}') }}"

mongodb_root_admin_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_root_admin_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_user_admin_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_user_admin_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_root_backup_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_root_backup_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_exporter_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_exporter_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_keyfile_content: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_keyfile_content token={{ vault_token }} url={{ vault_url }}') }}"

mongodb_users: # Optional if you want to add multiple regular users
  - name: admin
    password: "admin_password"
    roles: readWriteAnyDatabase
    database: admin
  - name: user_rw
    password: "user_rw_password"
    roles: readWrite
    database: user_database
  - name: user_ro
    password: "user_ro_password"
    roles: read
    database: user_database
    
mongodb_oplog_users: # Optional if you want to add oplog user
  - name: oplog
    password: "oplog_password"
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
www.mongoshard-2-3.com

[mongo_sharded_cluster:children]
mongocfg_servers
mongos_servers
mongo_shard_01
mongo_shard_02
```

### Example var file for sharded cluster
```yaml
mongodb_version: "6.0"

mongodb_net_tls_enabled: true # Optional: true|false (Default: false)
mongodb_net_tls_config:
  mode: "requireTLS"
  certificateKeyFileContent: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:certificateKeyFileContent token={{ vault_token }} url={{ vault_url }}') }}"
  CAFileContent: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:CAFileContent token={{ vault_token }} url={{ vault_url }}') }}"

mongodb_sharded_databases: # List of databases to run command sh.enableSharding()
  - user_database_1
  - user_database_2

mongodb_root_admin_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_root_admin_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_user_admin_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_user_admin_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_root_backup_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_root_backup_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_exporter_password: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_exporter_password token={{ vault_token }} url={{ vault_url }}') }}"
mongodb_keyfile_content: "{{ lookup('hashi_vault', 'secret=services/test-namespace/prd/mongodb:mongodb_keyfile_content token={{ vault_token }} url={{ vault_url }}') }}"

mongodb_users: # Optional if you want to add multiple regular users
  - name: admin
    password: "admin_password"
    roles: readWriteAnyDatabase
    database: admin
  - name: user_rw_1
    password: "user_rw_password"
    roles: readWrite
    database: user_database_1
  - name: user_ro_1
    password: "user_ro_password"
    roles: read
    database: user_database_1
  - name: user_rw_2
    password: "user_rw_password"
    roles: readWrite
    database: user_database_2
  - name: user_ro_2
    password: "user_ro_password"
    roles: read
    database: user_database_2
    
mongodb_oplog_users: # Optional if you want to add oplog user
  - name: oplog
    password: "oplog_password"
```

## Other examples

### Adding and deleting members in the replicaset 

To add or delete members in the replicaset:

1) Add new or remove unwanted members from the hosts file
2) Make sure the number of members is odd
3) Run playbook with extra-var `-e mongodb_replication_reconfigure=true`

### Adding normal users to the worked production database

To add users, simply add a new user (e.g. `new_user`) to your var file and run the playbook with the `mongodb-add-users` tag. It's safe even in production.
```yaml
mongodb_users: # Optional if you want to add multiple regular users
  - name: admin
    password: "admin_password"
    roles: readWriteAnyDatabase
    database: admin
  - name: user_rw
    password: "user_rw_password"
    roles: readWrite
    database: user_database
  - name: user_ro
    password: "user_ro_password"
    roles: read
    database: user_database
  - name: new_user
    password: "new_user_password"
    roles: readWrite
    database: new_user_database
```

### Deleting normal users from the worked production database

To delete users, add key `state: absent` for a specific user (e.g. `new_user`) to your var file and run the playbook with the `mongodb-add-users` tag. It's safe even in production.
```yaml
mongodb_users: # Optional if you want to add multiple regular users
  - name: admin
    password: "admin_password"
    roles: readWriteAnyDatabase
    database: admin
  - name: user_rw
    password: "user_rw_password"
    roles: readWrite
    database: user_database
  - name: user_ro
    password: "user_ro_password"
    roles: read
    database: user_database
  - name: new_user
    database: new_user_database
    state: absent # Optional: present|absent (Default: present)

mongodb_oplog_users: # Optional if you want to add oplog user
  - name: oplog
    state: absent # Optional: present|absent (Default: present)
```

### Updating passwords for admins and normal users

To update passwords for all admin users, set global variable `mongodb_admin_update_password: true`

To update passwords for all normal users, set global variable `mongodb_user_update_password: true`

To selectively update normal user passwords, add key `update_password: true` for a specific user (e.g. `user_ro`) to your var file and run the playbook with the `mongodb-add-users` tag. It's safe even in production.
```yaml
mongodb_users: # Optional if you want to add multiple regular users
  - name: admin
    password: "admin_password"
    roles: readWriteAnyDatabase
    database: admin
  - name: user_rw
    password: "user_rw_password"
    roles: readWrite
    database: user_database
  - name: user_ro
    password: "user_ro_new_password"
    roles: read
    database: user_database
    update_password: true # Optional: true|false (Default: false)

mongodb_oplog_users: # Optional if you want to add oplog user
  - name: oplog
    password: "oplog_new_password"
    update_password: true # Optional: true|false (Default: false)
```

**See worked example in tests/ directory**

## License

MIT
