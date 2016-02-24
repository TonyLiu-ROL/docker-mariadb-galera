# docker-mariadb-galera
Building a Mariadb Galera Cluster with Docker

# How to use this

- Build docker image
```
#sudo docker build -t ubuntu_trusty/mariadb-galera .
#sudo docker images |grep mariadb-galera
ubuntu_trusty/mariadb-galera    latest              63ff6a4fd348        2 hours ago         575.8 MB
```

- Launch docker images
```
#sudo docker run --name mariadb1 -i -t -d ubuntu_trusty/mariadb-galera /bin/bash
#sudo docker run --name mariadb2 -i -t -d ubuntu_trusty/mariadb-galera /bin/bash
#sudo docker run --name mariadb3 -i -t -d ubuntu_trusty/mariadb-galera /bin/bash

#sudo docker ps |cut -d' ' -f1 |grep -v CONTAINER | xargs sudo docker inspect |egrep '"Name"|IPAddress'

"IPAddress": "172.17.75.122",
"SecondaryIPAddresses": null,
"Name": "/mariadb3",
"IPAddress": "172.17.75.121",
"SecondaryIPAddresses": null,
"Name": "/mariadb2",
"IPAddress": "172.17.75.120",
"SecondaryIPAddresses": null,
"Name": "/mariadb1",

```

- Connect to docker image

Install nsenter, a little Linux tool to fiddle with namespaces and enter them, usually provided by util-linux.
```
#wget https://www.kernel.org/pub/linux/utils/util-linux/v2.27/util-linux-2.27.tar.gz
#tar zxf util-linux-2.27.tar.gz
#cd util-linux-2.27/
#./configure --without-ncurses
#make
#chmod +x nsenter
#sudo cp nsenter /usr/local/bin
```
Enter containter
```
#sudo docker ps
43b6db863782        ubuntu_trusty/mariadb-galera                                       "/bin/bash"            2 hours ago         Up 2 hours          3306/tcp, 4444/tcp, 4567-4568/tcp   mariadb3            
f1db2870c06e        ubuntu_trusty/mariadb-galera                                       "/bin/bash"            2 hours ago         Up 2 hours          3306/tcp, 4444/tcp, 4567-4568/tcp   mariadb2            
02aaffd533b6        ubuntu_trusty/mariadb-galera                                       "/bin/bash"            2 hours ago         Up 2 hours          3306/tcp, 4444/tcp, 4567-4568/tcp   mariadb1

#PID=$(sudo docker inspect --format '{{.State.Pid}}' 02aaffd533b6)
#sudo nsenter --target $PID --mount --uts --ipc --net --pid
root@02aaffd533b6:/#
```

- Configure Mariadb Cluster

On containter mariadb1:
```
root@02aaffd533b6:/#cat /proc/mounts > /etc/mtab
root@02aaffd533b6:/#vi /etc/mysql/my.cnf
wsrep_cluster_address=gcomm://172.17.75.118,172.17.75.120,172.17.75.121
root@02aaffd533b6:/#service mysql restart
 * Stopping MariaDB database server mysqld         [ OK ] 
 * Starting MariaDB database server mysqld         [ OK ] 
 * Checking for corrupt, not cleanly closed and upgrade needing tables.
```
On containter mariadb2/3, repeat above, and any one of the three containters, for example mariadb3:
```
root@43b6db863782:/#mysql -uroot -proot
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 30
Server version: 5.5.48-MariaDB-1~trusty-wsrep mariadb.org binary distribution, wsrep_25.14.r9949137

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]> show status like 'wsrep_cluster%';
+--------------------------+--------------------------------------+
| Variable_name            | Value                                |
+--------------------------+--------------------------------------+
| wsrep_cluster_conf_id    | 13                                   |
| wsrep_cluster_size       | 3                                    |
| wsrep_cluster_state_uuid | f5b06a9c-dab1-11e5-a596-fb66c1dd04f9 |
| wsrep_cluster_status     | Primary                              |
+--------------------------+--------------------------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> show status like 'wsrep_%';
+------------------------------+----------------------------------------------------------+
| Variable_name                | Value                                                    |
+------------------------------+----------------------------------------------------------+
| wsrep_local_state_uuid       | f5b06a9c-dab1-11e5-a596-fb66c1dd04f9                     |
| wsrep_protocol_version       | 7                                                        |
| wsrep_last_committed         | 1                                                        |
| wsrep_replicated             | 0                                                        |
| wsrep_replicated_bytes       | 0                                                        |
| wsrep_repl_keys              | 0                                                        |
| wsrep_repl_keys_bytes        | 0                                                        |
| wsrep_repl_data_bytes        | 0                                                        |
| wsrep_repl_other_bytes       | 0                                                        |
| wsrep_received               | 2                                                        |
| wsrep_received_bytes         | 303                                                      |
| wsrep_local_commits          | 0                                                        |
| wsrep_local_cert_failures    | 0                                                        |
| wsrep_local_replays          | 0                                                        |
| wsrep_local_send_queue       | 0                                                        |
| wsrep_local_send_queue_max   | 1                                                        |
| wsrep_local_send_queue_min   | 0                                                        |
| wsrep_local_send_queue_avg   | 0.000000                                                 |
| wsrep_local_recv_queue       | 0                                                        |
| wsrep_local_recv_queue_max   | 1                                                        |
| wsrep_local_recv_queue_min   | 0                                                        |
| wsrep_local_recv_queue_avg   | 0.000000                                                 |
| wsrep_local_cached_downto    | 18446744073709551615                                     |
| wsrep_flow_control_paused_ns | 0                                                        |
| wsrep_flow_control_paused    | 0.000000                                                 |
| wsrep_flow_control_sent      | 0                                                        |
| wsrep_flow_control_recv      | 0                                                        |
| wsrep_cert_deps_distance     | 0.000000                                                 |
| wsrep_apply_oooe             | 0.000000                                                 |
| wsrep_apply_oool             | 0.000000                                                 |
| wsrep_apply_window           | 0.000000                                                 |
| wsrep_commit_oooe            | 0.000000                                                 |
| wsrep_commit_oool            | 0.000000                                                 |
| wsrep_commit_window          | 0.000000                                                 |
| wsrep_local_state            | 4                                                        |
| wsrep_local_state_comment    | Synced                                                   |
| wsrep_cert_index_size        | 0                                                        |
| wsrep_causal_reads           | 0                                                        |
| wsrep_cert_interval          | 0.000000                                                 |
| wsrep_incoming_addresses     | 172.17.75.122:3306,172.17.75.120:3306,172.17.75.121:3306 |
| wsrep_evs_delayed            |                                                          |
| wsrep_evs_evict_list         |                                                          |
| wsrep_evs_repl_latency       | 0/0/0/0/0                                                |
| wsrep_evs_state              | OPERATIONAL                                              |
| wsrep_gcomm_uuid             | fe925c68-daca-11e5-ba57-92eed784132d                     |
| wsrep_cluster_conf_id        | 13                                                       |
| wsrep_cluster_size           | 3                                                        |
| wsrep_cluster_state_uuid     | f5b06a9c-dab1-11e5-a596-fb66c1dd04f9                     |
| wsrep_cluster_status         | Primary                                                  |
| wsrep_connected              | ON                                                       |
| wsrep_local_bf_aborts        | 0                                                        |
| wsrep_local_index            | 2                                                        |
| wsrep_provider_name          | Galera                                                   |
| wsrep_provider_vendor        | Codership Oy <info@codership.com>                        |
| wsrep_provider_version       | 25.3.14(r3560)                                           |
| wsrep_ready                  | ON                                                       |
| wsrep_thread_count           | 2                                                        |
+------------------------------+----------------------------------------------------------+
57 rows in set (0.00 sec)
```

# License

Copyright 2016 TonyLiu

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
