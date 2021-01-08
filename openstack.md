
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

### ~> controller

   echo "stack ALL=(ALL)	NOPASSWD: ALL" > /etc/sudoers.d/stack

	mysql -uroot -p
		CREATE DATABASE keystone;
		GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'keystoneDBPass';
		GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'keystoneDBPass';
		exit

dnf -y install openstack-keystone httpd
dnf -y install python3-mod_wsgi

#### nano /etc/keystone/keystone.conf
[descomente e edite]

    memcache_servers = controller:11211
[adicione a linha no bloco **[database]** ]

    connection = mysql+pymysql://keystone:keystoneDBPass@controller/keystone

[descomente]

    provider = fernet


#### su -s /bin/sh -c "keystone-manage db_symc" keystone

    mysql -uroot -p
    	show databases;
    	use keystone;
    	show tables;
    	exit
--

    keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
    keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

--

    keystone-manage bootstrap --bootstrap-password adminPass \
    	--bootstrap-admin-url http://controller:5000/v3/ \
    	--bootstrap-internal-url http://controller/v3/ \
    	--bootstrap-region-id RegionOne

--

    setsebool -P httpd_use_openstack on
    setsebool -P httpd_can_network_connect on
    setsebool -P httpd_can_network_connect_db on
    firewall-cmd --add-port=5000/tcp --permanent
    firewall-cmd --reload

[edite httpd.conf | linha SeverName www.example.com:80]

    nano /etc/httpd/conf/httpd.conf
	    ServerName controller
--

    ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd.conf.d/
    systemctl enable httpd.service
    systemctl start httpd.service
    systemctl status httpd.service

[user stack]
su - stack

    export OS_USERNAME=admin
    export OS_PASSWORD=adminPass
    export OS_PROJECT_NAME=admin
    export OS_USER_DOMAIN_NAME=Default
    export OS_PROJECT_DOMAIN_NAME=Default
    export OS_AUTH_URL=http://controller/v3
    export OS_IDENTITY_API_VERSION=3


    sudo dnf -y install python3-openstackclient mod_ssl ~~python3-mod_wsgi~~ python3-oauth2client
    sudo firewall-cmd --add-service={mysql,memcache} --permanent
    sudo firewall-cmd --add-port=5672/tcp --permanent

--

    openstack domain create --description "an example domain" example
    
    openstack project create --domain deafult --description "service project" service
    
    openstack project list
    
    openstack project create --domain default --description "Demo TheSkillPedia project" TheSkillPedia
    
    openstack user create --domain default --password demoPass demo
    
    openstack role create myrole
    openstack role add --project TheSkillPedia --user demo myrole
    openstack role add --project TheSkillPedia --user demo member

    unset OS_AUTH_URL OS_PASSWORD

    openstack --os-auth-url http://controller:5000/v3 \ 
    	--os-project-domain-name Default --os-user-domain-name Default  \
    	--os-project-name admin --os-username admin token issue

    oénstack --os-auth-url http://controller:5000/v3  /
    	--os-project-domain-name Default --os-user-domain-name Default  \
    	--os-project-name TheSkillPedia --os-username demo token issue

#### [criar arquivo admin-openrc]

    nano admin-openrc
	    export OS_USERNAME=admin
	    export OS_PASSWORD=adminPass
	    export OS_PROJECT_NAME=admin
	    export OS_USER_DOMAIN_NAME=Default
	    export OS_PROJECT_DOMAIN_NAME=Default
	    export OS_AUTH_URL=http://controller:5000/v3
	    export OS_IDENTITY_API_VERSION=3
	    export OS_IMAGE_API_VERSION=2

#### [criar arq demo-openrc]
nano demo-openrc
```
    export OS_USERNAME=demo
    export OS_PASSWORD=demoPass
    export OS_PROJECT_NAME=TheSkillPedia
    export OS_USER_DOMAIN_NAME=Default
    export OS_PROJECT_DOMAIN_NAME=Default
    export OS_AUTH_URL=http://controller:5000/v3
    export OS_IDENTITY_API_VERSION=3
    export OS_IMAGE_API_VERSION=2
```
--

    source admin-openrc
    openstack token issue   [validade token]


### GLANCE

### GLANCE
 Install Openstack ussuri Image Service Glance
#### usuario: stack

    source admin-openrc
    env | grep OS
    openstack user list
    openstack role list
    openstack project list
    openstack service list [somente keystone na lista]

mysql -uroot -P

    CREATE DATABASE glance;
    GRANT ALL PRIVILEGES ON glance.*  TO 'glance'@'localhost' IDENTIFIED BY 'glanceDBPass';
    GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'glanceDBPass';
    flush privileges;
    exit

[criar projeto/servico glance]
openstack user create --domain default --project service --password glancePass glance

openstack role add --project service --user glance admin

openstack service create --name glance --description "TheSkillPedia openstack image" image

--
openstack endpoint create --region RegionOne image public http://controller:9292
openstack endpoint create --region RegionOne image internal http://controller:9292
openstack endpoint create --region RegionOne image admin http://controller:9292
exit
## usuario: root

    dnf --enablerepo=centos-openstack-ussuri, PowerTools -y install openstack-glace wget

---
#### [editar arquivo ]

> nano /etc/glance/glance-api.conf

[colocar em [default]]

    bind_host = 0.0.0.0

[colocar em [glance_store]]

    stores = file,http
    default_store = file
    filesystem_store_datadir = /var/lib/glance/images/

[colocar em [database]]

    connection = mysql+pymysql://glance:glanceDBPass@controller/glance

--

[colocar em [keystone_authtoken]]

    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = Default
    user_domain_name = Default
    project_name = service
    username = glance
    password = glancePass

[colocar em[paste_deploy]]

    flavor = keystone

--
chmod 640 /etc/glance/glance-api.conf
chown root:glance /etc/glance/glance-api.conf

su -s /bin/bash glance -c "glance-manage db_sync"

systemctl enable --now openstack-glance-api
setsebool -P glance_api_can_network on

[criar aquivo glanceapi.te]

>    nano glanceapi.te

   module glanceapi 1.0;

require {
	type glance_api_t;
        type httpd_config_t;
        type iscsid_exec_t;
        class dir search;
        class file { getattr open read };
}

#============= glance_api_t =====
allow glance_api_t httpd_config_t:dir search;
allow glance_api_t iscsid_exec_t:file { getattr open read };

checkmodule -m -M -o glanceapi.mod glanceapi.te
semodule_package --outfile glanceapi.pp --module glanceapi.mod
semodule -i glanceapi.pp

systemctl enable openstack-glance-api
systemctl start openstack-glance-api
systemctl status openstack-glance-api

firewall-cmd --permanent --add-port{9191,9292}/tcp
firewall-cmd --reload

### usuario: stack

    su - stack
    source admin-openrc
    [download image]
    wget http://download.cirros-cloud.net/0.5.1/cirros-0.5.1-x86_64-disk.img

[adicionar imagem ao openstack]

    openstack image create "cirros" --file cirros-0.5.1-x86_64-disk.img disk-format raw --container-format bare --public

> openstack image list
---
---

# neutron
Openstack Ussuri and Victoria Neutron Service











































