
##openstack ussuri on centos 8 - Configure Prerequisites RabbitMQ, MariaDB ETCD, Memcache and Chrony
CONTROLLER:

    dnf install https://rdoproject.org/repos/rdo-release.el8.rpm

    dnf install python3-openstackclient openstack-selinux

    dnf -y install mariadb mariadb-server python3-PyMySQL

##### [edit file]    **vi /etc/my.cnf.d/openstack.cnf**

[mysqld]
bind-address = 10.0.0.11
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8

    systemctl enable mariadb.service
    systemctl start mariadb.service

##### configure mysql

    mysql_secure_installation   [senha mariadb]
    firewall-cmd --permanent --add-service=mysql
    firewall-cmd --reload

##### install rabbitmq server

    yum -y install rabbitmq-server   [caso de erro, continue abaixo]
    yum config-manager --set-enable powertools
    yum -y install rabbitmq-server

    systemctl enable rabbitmq-server.service && systemctl start rabbitmq-server.service
    systemctl status rabbitmq-server [service on]

add user **openstack** on **rabbit**: passwork:**rabbitPass**

    rabbitmqctl add_user openstack rabbitPass
    rabbitmqctl set_permissions openstack ".*" ".*" ".*"
    firewall-cmd --permanent --add-port={11211/tcp,5672/tcp}
    firewall-cmd --reload
install memcached

    dnf -y install memcached python3-memcached
edit file memcached

> nano /etc/sysconfig/memcached

    PORT="11211"
    USER="memcached"
    MAXCONN="1024"
    CACHESIZE="64"
    OPTIONS="-l 127.0.0.1,::1,controller"

-
    systemctl enable memcached.service
    systemctl status memcached.service     ->[service on]

##### install apache/etcd

    yum -y install etcd

##### [edit file etcd.conf] -> nano /etc/etcd/etcd.conf

    #[Member]
    #ETCD_CORS=""
    ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
    #ETCD_WAL_DIR=""
    ETCD_LISTEN_PEER_URLS="http://10.0.0.11:2380"
    ETCD_LISTEN_CLIENT_URLS="http://10.0.0.11:2379"
    #ETCD_MAX_SNAPSHOTS="5"
    #ETCD_MAX_WALS="5"
    ETCD_NAME="controller"
    #ETCD_SNAPSHOT_COUNT="100000"
    #ETCD_HEARTBEAT_INTERVAL="100"
    #ETCD_ELECTION_TIMEOUT="1000"
    #ETCD_QUOTA_BACKEND_BYTES="0"
    #ETCD_MAX_REQUEST_BYTES="1572864"
    #ETCD_GRPC_KEEPALIVE_MIN_TIME="5s"
    #ETCD_GRPC_KEEPALIVE_INTERVAL="2h0m0s"
    #ETCD_GRPC_KEEPALIVE_TIMEOUT="20s"
    #
    #[Clustering]
    ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.0.0.11:2380"
    ETCD_ADVERTISE_CLIENT_URLS="http://10.0.0.11:2379"
    #ETCD_DISCOVERY=""
    #ETCD_DISCOVERY_FALLBACK="proxy"
    #ETCD_DISCOVERY_PROXY=""
    #ETCD_DISCOVERY_SRV=""
    ETCD_INITIAL_CLUSTER="controller=http://10.0.0.11:2380"
    ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
    ETCD_INITIAL_CLUSTER_STATE="new"
    #ETCD_STRICT_RECONFIG_CHECK="true"
    #ETCD_ENABLE_V2="true"
    #

systemctl enable etcd
systemctl start etcd
systemctl status etcd

    firewall-cmd --permanent --add-port={2380/tcp,2379/tcp}
    firewall-cmd --reload

[entrar no mysql, verificar funcionamento]

    mysql -uroot -p
    
   systemctl status chronyd  [service on]
   

##### nano /etc/chrony.conf [descomente a linha e edite]
	allow 10.0.0.0/24
    
systemctl restart chronyd
```
firewall-cmd --add-service=ntp --permanent
firewall-cmd --reload
chronyc sources
```
---
MUDE PARA MAQUINA **COMPUTE**

##### nano /etc/chrony.conf [descomente a linha e edite]

    pool controller iburst
#### systemctl restart chronyd
#### systemctl status chronyd

	 chronyc sources
	 firewall-cmd --add-service=ntp --permanent
	 firewall-cmd --reload
	 chronyc sources


---
MUDE PARA MAQUINA **STORAGE**
##### nano /etc/chrony.conf [descomente a linha e edite]

	pool controller iburst

#### systemctl restart chronyd
#### systemctl status chronyd

    firewall-cmd --add-service=ntp --permanent
    firewall-cmd --reload
    chronyc sources

----
# keystone
Openstack ussuri Keystone CentOS 8















