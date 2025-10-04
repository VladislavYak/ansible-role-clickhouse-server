[![Build Status](https://travis-ci.com/nl2go/ansible-role-clickhouse.svg?branch=master)](https://travis-ci.com/nl2go/ansible-role-clickhouse)
[![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/nl2go/ansible-role-clickhouse)](https://galaxy.ansible.com/nl2go/clickhouse)
[![Ansible Galaxy](https://img.shields.io/badge/role-nl2go.clickhouse-blue.svg)](https://galaxy.ansible.com/nl2go/clickhouse/)
[![Ansible Galaxy Quality Score](https://img.shields.io/ansible/quality/49634)](https://galaxy.ansible.com/nl2go/clickhouse/)
[![Ansible Galaxy Downloads](https://img.shields.io/ansible/role/d/49634.svg?color=blue)](https://galaxy.ansible.com/nl2go/clickhouse/)

# Ansible Role: ClickHouse

An Ansible Role that manages installation and configuration of [ClickHouse](https://clickhouse.tech/).

## Role Variables

```
---
# Basic ClickHouse configuration settings
# See more information about every setting in the config template
clickhouse_key_server: hkp://keyserver.ubuntu.com:80
clickhouse_listen_host: 127.0.0.1
clickhouse_http_port: 8123
clickhouse_tcp_port: 9000
clickhouse_mysql_port: 9004
clickhouse_interserver_http_port: 9009
clickhouse_version: "{{ clickhouse_version }}"
clickhouse_deb_rep: "{{ clickhouse_deb_rep }}"

clickhouse_path: /var/lib/clickhouse
clickhouse_tmp_path: "{{ clickhouse_path }}/tmp"
clickhouse_user_files_path: "{{ clickhouse_path }}/user_files"
clickhouse_access_control_path: "{{ clickhouse_path }}/access"
clickhouse_conf_dir: /etc/clickhouse-server
clickhouse_loglevel: "information"  # More options: https://github.com/pocoproject/poco/blob/poco-1.9.4-release/Foundation/include/Poco/Logger.h#L105
clickhouse_log_directory: "{{ clickhouse_path }}/logs"

clickhouse_profiles: # More settings: https://clickhouse.tech/docs/en/operations/settings/settings-profiles/
  default:
    max_memory_usage: 10000000000
    use_uncompressed_cache: 0
    load_balancing: "random"
    optimize_skip_unused_shards: 1
    background_pool_size: 64
    background_move_pool_size: 32
    max_partitions_per_insert_block: 120

clickhouse_users: # More settings: https://clickhouse.tech/docs/en/operations/settings/settings-users/
  default:
    password: ""
    profile: "default"
    quota: "default"

clickhouse_quotas: # More settings: https://clickhouse.tech/docs/en/operations/quotas/
  default:
    intervals:
      - duration: 3600
        queries: 0
        errors: 0
        result_rows: 0
        read_rows: 0
        execution_time: 0


clickhouse_group: clickhouse
clickhouse_user: clickhouse
clickhouse_shard: 01
clickhouse_cluster: clickhouse-cluster
keeper_cluster: keeper-cluster
clickhouse_distributed_cluster: distributed_cluster


# Zookeeper/Keeper Configuration
keeper_enabled: true
keeper_port: 9181
keeper_raft_port: 9234
```

A settings profile is a collection of [settings](https://clickhouse.tech/docs/en/operations/settings/settings-profiles/) grouped under the same name.

    clickhouse_profiles:
      default:
        password: "qwerty"
        profile: "default"
        quota: "default"

The `users` section of the `user.xml` configuration file contains user  [settings](https://clickhouse.tech/docs/en/operations/settings/settings-users/).

    clickhouse_profiles:
      default:
        intervals:
          - duration: 3600
            queries: 0
            errors: 0
            result_rows: 0
            read_rows: 0
            execution_time: 0

**clickhouse_path**: Path to the directory where ClickHouse will store data

**clickhouse_log_directory**: Path to the error and normal logs.

**clickhouse_clusters**: is the list of clusters and also contains the shards that are part of each cluster.

    clickhouse_clusters:
      distributed_cluster_1:
        shard1:
          - { host: "host_1", port: 9000 }
          - { host: "host_2", port: 9000 }
        shard2:
          - { host: "host_3", port: 9000 }
          - { host: "host_4", port: 9000 }
      distributed_cluster_2:
        shard1:
          - { host: "host_5", port: 9000 }
          - { host: "host_6", port: 9000 }
        shard2:
          - { host: "host_7", port: 9000 }
          - { host: "host_8", port: 9000 }

For the file macros.xml, you need to fill the variables below:

**clickhouse_cluster**: is the cluster name which is used for replication.

**keeper_cluster**: is the cluster name for keeper nodes

**clickhouse_distributed_cluster**: is the distributed cluster name defined in **clickhouse_clusters**.

**clickhouse_shard**: is the shard number (01, 02) that being part of the replication inside one shard.

## Dependencies

None.

## Example Playbook
```yaml
- name: Deploy Clickhouse server
  hosts: clickhouse_cluster
  become: true
  vars:
    clickhouse_version: 25.3.3.42
    clickhouse_deb_rep: "deb https://packages.clickhouse.com/deb stable main"
    clickhouse_replica_id: "-0" # display_name suffix
    clickhouse_replica_name: "testtest"
    clickhouse_timezone: "Europe/Moscow"
    clickhouse_kafka_client_users:
      - username: guest
        password: guest
    clickhouse_clusters:
      sharded_cluster:
        shard_1:
            - { host: "194.87.118.205", port: 9000 }
        shard_2:
            - { host: "194.87.118.36", port: 9000 }
      replicated_cluster:
        shard_1:
            - { host: "194.87.118.205", port: 9000}
            - { host: "194.87.118.36", port: 9000}

    clickhouse_distributed_user: "testtest"
    clickhouse_listen_addresses:
      - "{{ ansible_default_ipv4.address }}"
  roles:
    - role: ansible-role-clickhouse-server
```
## License

See the [LICENSE.md](LICENSE.md) file for details.

## Author Information

tba