
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
    
    openstack project create --domain default --description "service project" service
    
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

### user: stack
su - stack
source  admin-openrc

openstack catalog list **[nova(compute),keystone(identity),glance(img),placement(placement)]**


### user: root
mysql -uroot -p

    CREATE DATABASE neutron;
    GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'neutronDBPass';
    GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'neutronDBPass';
    flush privileges;
    exit

### user: stack
su - stack
source  admin-openrc

[criar usuario neutron no openstack]

    openstack user create --domain default --projet service --password neutronUserPass neutron

[adc role admin]

    openstack role add --project service --user neutron admin

[serviço neutron, tipo network]

    openstack service create --name neutron --description "TheSkillPedia openstack networking" network

[criar endpoint para a network criada: [public,internal,admin]]

    oṕenstack endpoint create --region RegionOne network public http://controller:9696
    oṕenstack endpoint create --region RegionOne network internal http://controller:9696
    oṕenstack endpoint create --region RegionOne network admin http://controller:9696


### user: root

    dnf -y install openstack-neutron openstack-neutron-ml2 openstack-neutron-openvswitch
    
    mv /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org

[editar/criar arquivo neutron.conf]
#### nano /etc/neutron/neutron.conf

    [DEFAULT]
    core_plugin = ml2
    service_plugins = router
    auth_strategy = keystone
    state_path = /var/lib/neutron
    dhcp_agent_notification = True
    allow_overlapping_ips = True
    notify_nova_on_port_status_changes = True
    notify_nova_on_port_data_changes = True
    
    transport_url = rabbit://openstack:rabbitPass@controller
    
    [agent]
    root_helper = "sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf"
    
    # keystone auth info
    [keystone_authtoken]
    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = neutron
    password = neutronUserPass
    
    [database]
    connection = mysql+pymysql://neutron:neutronDBPass@controller/neutron
    
    #nova auth info
    [nova]
    auth_url = http://controller:5000
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = nova
    password = novaPass
    
    [oslo_concurrency]
    lock_path = /var/lib/neutron/tmp

--

    chmod 640 /etc/neutron/neutron.conf
    chgrp neutron /etc/neutron/neutron.conf

[edit arq l3_agent.ini (adicionar a linha em [default])]
#### nano /etc/neutron/l3_agent.ini

> interface_driver = openvswitch

[edit arq dhcp_agent.ini (adicionar as linhas em [default])]
#### nano /etc/neutron/dhcp_agent.ini

    interface_driver = openvswitch
    dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
    enable_isolated_metadata = true

--
[edit arq metadata_agent.ini 
#### nano /etc/neutron/metadata_agent.ini
EM DEFAULT:

    nova_metadata_host = controller
    # specify any secret key you like
    metadata_proxy_shared_secret = neutronMetaPass

[linha:212] - DESCOMENTAR memcahe_servers = localhost:11211  

    memcahe_servers = controller:11211 

--
[adicionar no final do arquivo]

> nano /etc/neutron/plugins/ml2/ml2_conf.ini

    [ml2]
    type_drivers = flat,vlan,gre,vxlan
    tenant_network_types =
    mechanism_drivers = openvswitch
    extension_drivers = port_security
--
[adicionar no final do arquivo]

> nano /etc/neutron/plugins/ml2/openvswitch_agent.ini

    [securitygroup]
    firewall_driver = openvswitch
    enable_security_group = true
    enable_ipset = true
---
NOVA - ***~~NAO INSTALADO~~***
---
[edit nova.conf]

> nano /etc/nova/nova.conf

#### EM [DEFAULT]
    my_ip = 10.0.0.11
    state_path = /var/lib/nova
    enabled_apis = osapi_compute,metadata
    log_dir = /var/log/nova
    transport_url = rabbit://openstack:rabbitPass@controller
    use_neutron = True
    linuxnet_interface_driver = nova.network.linux_net.LinuxOVSInterfaceDriver
    firewall_driver = nova.virt.firewall.NoopFirewallDriver
    vif_plugging_is_fatal = True
    vif_plugging_timeout = 300

#### EM [NEUTRON]

    auth_url = http://controller:5000
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = neutron
    password = neutronUserPass
    service_metadata_proxy = True
    metadata_proxy_shared_secret = neutronMetaPass

--

    dnf install openstack-selinux
    
    setsebool -P neutron_can_network on
    setsebool -P haproxy_connect_any on
    setsebool -P daemons_enable_cluster_mode on

[criar arq]

> nano ovsofctl.te

    module ovsofctl 1.0;
    
    require {
    	type neutron_t;
            type neutron_exec_t;
            type neutron_t;
            type dnsmasq_t;
            class file execute_no_trans;
            class capability { dac_override sys_rawio };
    }
    
    allow neutron_t self:capability { dac_override sys_rawio };
    allow neutron_t neutron_exec_t:file execute_no_trans;
    
    allow dnsmasq_t self:capability dac_override;

checkmodule -m -M -o ovsofctl.mod ovsogctl.te   **[checar erro no arq]**
semodule_package --outfile ovsofctl.pp --module ovsofctl.mod
semodule -i ovsofctl.pp

firewall-cmd --add-port=9696/tcp --permanent
firewall-cmd --reload

systeamctl enable --now openvswitch
ovs-vsctl add-br br-int

    ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
    su -s /bin/bash neutron -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugin.ini upgrade head"
    
    for service in server dhcp-agent l3-agent metadata-agent openvswitch-agent; do systemctl enable --now neutron-$service done
---
### PROBLEMA NO MODULO NOVA - 

    systemctl restart openstack-nova-api openstack-nova-compute
    systemctl status openstack-nova-compute

modulo openstack-nova-compute nao operando
---
### user: stack

    su - stack
    source admin-openrc
    
    openstack network agent list   [min video:11:09]

### user: root
ovs-vsctl add-br br-enp0s9
ovs-vsctl add-port br-enp0s9 enp0s9

--
[editar arq, adicionar no final]

> vi /etc/neutron/plugins/ml2/ml2_conf.ini

    [ml2_type_flat]
    flat_networks = physnet1

[editar arq, adicionar no final]

> nano /etc/neutron/plugins/ml2/openvswitch_agent.ini

    [ovs]
    bridge_mappings = physnet1:br-enp0s9

--

    systemctl restart neutron-openvswitch-agent

### user: stack

    source admin-openrc
    projectID=$ (openstack project list | grep service | awk '{print $2}')
    
--

    openstack network create --project $projectID \ --share --provider-network-type flat --provider-physical-network physnet1 sharednet1

--
#### criar rede openstack
openstack subnet create subnet1 --network sharednet1 \
    --project $projectID --subnet-range 10.0.0.0/24 \
    --allocation-pool start=10.0.0.200,end=10.0.0.254 \
    --gateway 10.0.0.1 --dns-nameserver 10.0.0.10

    openstack network list
    openstack subnet list

---
---
## NOVA ON CONTROLLER

## NOVA ON CONTROLLER
Openstack Nova on Controller and Compute Node

### user: stack
source admin-openrc
openstack service list   [keystone e glance]

##### mysql -uroot -p

    CREATE DATABASE nova;
    CREATE DATABASE nova_api;
    CREATE DATABASE nova_cell0;
    CREATE DATABASE placement;
    
    GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'novaDBPass';
    GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'novaDBPass';
    
    GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'novaDBPass';
    GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'novaDBPass';
    
    GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'novaDBPass';
    GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'novaDBPass';  

    GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY 'placementDBPass';
    GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY 'placementDBPass';      
     
    flush privileges;
    exit


[criar user:nova]

    openstack user create --domain default --project service --password novaPass nova
    openstack role add --project service --user nova admin

    openstack user create --domain default --project service --password placementPass placement
    openstack role add --project service --user placement admin

    openstack service create --name nova --description "TheSkillPedia openstack compute" compute
    openstack service create --name placement --description "TheSkillPedia placement api" placement

    openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1/%\(tenant_id\)s
    openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1/%\(tenant_id\)s
    openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1/%\(tenant_id\)s

    openstack endpoint create --region RegionOne placement public http://controller:8778
    openstack endpoint create --region RegionOne placement internal http://controller:8778
    openstack endpoint create --region RegionOne placement admin http://controller:8778

### user: root

    dnf -y installcopenstack-nova openstack-placement-api
    ou
    dnf --enablerepo=centos-openstack-ussuri,PowerTools -y installcopenstack-nova openstack-placement-api

--

### [editar arq nova.conf]
> nano /etc/nova/nova.conf

#### EM [DEFAULT]
    [DEFAULT]
    my_ip = 10.0.0.11
    state_path = /var/lib/nova
    enabled_apis = osapi_compute,metadata
    log_dir = /var/log/nova
    transport_url = rabbit://openstack:rabbitPass@controller
    use_neutron = True
    firewall_driver = nova.virt.firewall.NoopFirewallDriver

#### EM [API] - descomente: auth_strategy=keystone

    auth_strategy=keystone

#### EM [API_DATABASE]

    connection = mysql+pymysql://nova:novaDBPass@controller/nova_api

#### EM [database]

    connection = mysql+pymysql://nova:novaDBPass@controller/nova

#### EM [glance]

    api_servers = http://controller:9292

#### EM [keystone_authtoken]

    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = nova
    password = novaPass

#### EM [oslo_concurrency] descomente:

    lock_path=/var/lib/nova/tmp

#### EM [placement]

    auth_url = http://controller:5000
    region_name = RegionOne
    project_domain_name = default
    project_name = service
    auth_type = password
    user_domain_name = Default
    username = placement
    password = placementPass

#### EM [scheduler]

    discover_hosts_in_cells_interval=300

#### EM [vnc]

    enabled = true
    server_listen = $my_ip
    server_proxyclient_address = $my_ip

#### EM [wsgi]

    api_paste_config = /etc/nova/api-paste.ini

--

    chmod 640 /etc/nova/nova.conf
    chgrp nova /etc/nova/nova.conf

--
### [editar arquivo placement.conf]

### nano /etc/placement/placement.conf
#### EM [DEFAULT]

    debug = false
#### EM [api] descomente:

    auth_strategy = keystone

#### EM [placement_database]

    connection = mysql+pymysql://placement:placementDBPass@controller/placement

#### EM [keystone_authtoken]

    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = Default
    user_domain_name = Default
    project_name = service
    username = placement
    password = placementPass

--

    chmod 640 /etc/placement/placement.conf
    chgrp placement /etc/placement/placement.conf
   
~~chown placement. /var/log/placement/placement-api.log~~ [dir nao encontrado]

### [editar arquivo 00-placement-api.conf]     *VERIF IDENTAÇÃO*

> nano /etc/httpd/conf.d/00-placement-api.conf


--
    
    Listen 8778
    
    <VirtualHost *:8778>
      WSGIProcessGroup placement-api
      WSGIApplicationGroup %{GLOBAL}
      WSGIPassAuthorization On
      WSGIDaemonProcess placement-api processes=3 threads=1 user=placement group=placement
      WSGIScriptAlias / /usr/bin/placement-api
      <Directory /usr/bin>
      <IfVersion >= 2.4>
         Require all granted
      </IfVersion>
      <IfVersion < 2.4>
         Order allow,deny
         Allow from all
      </IfVersion>
      </Directory>
      <IfVersion >= 2.4>
        ErrorLogFormat "%M"
      </IfVersion>
      ErrorLog /var/log/placement/placement-api.log
      #SSLEngine On
      #SSLCertificateFile ...
      #SSLCertificateKeyFile ...
    </VirtualHost>
    
    Alias /placement-api /usr/bin/placement-api
    <Location /placement-api>
      SetHandler wsgi-script
      Options +ExecCGI
      WSGIProcessGroup placement-api
      WSGIApplicationGroup %{GLOBAL}
      WSGIPassAuthorization On
    </Location>

--


--

    dnf install openstack-selinux
    
    semanage port -a -t http_port_t -p tcp 8778
    
    firewall-cmd --permanent --add-port={6080,6081,6082,8774,8775,8778}/tcp
    firewall-cmd --reload
    
    su -s /bin/bash placement -c "placement-manage db sync"

### verificar comando (l ou 1)

    su -s /bin/bash placement -c "placement-manage db sync"
    
    su -s /bin/bash nova -c "nova-manage api_db sync"  [erro: Access denied for user 'nova'@'%' to database 'nova_api'"]
    
    su -s /bin/bash nova -c "nova-manage cell_v2 map_cell0"
    
    su -s /bin/bash nova -c "nova-manage db sync"  [warnings]
   
    
    su -s /bin/bash nova -c "nova-manage cell_v2 create_cell --name celll" [warning]
    

    systemctl restart httpd
    
    for service in api conductor scheduler novncproxy; do systemctl enable --now openstack-nova-$service done

### user: stack

    su - stack
    source admin-openr

    openstack service list   -[nova,keystone,glance,placement]

### MUDAR PARA MAQUINA: COMPUTE  23:
### user: root
    dnf -y install centos-release-openstack-ussuri
    dnf -y install centos-release-openstack-victoria
    
    yum -f install openstack-nova-compute openstack-selinux
    ou
    dnf --enablerepos=centos-openstack-ussuri,PoweTools -y openstack-nova-compute openstack-selinux

### [editar arq nova.conf]

> nano /etc/nova/nova.conf

#### EM [DEFAULT]

    my_ip = 10.0.0.31
    state_path = /var/lib/nova
    enabled_apis = osapi_compute,metadata
    log_dir = /var/log/nova
    transport_url = rabbit://openstack:rabbitPass@controler
    use_neutron = true
    firewall_driver = nova.virt.firewall.NoopFirewallDriver

#### EM [api] descomente:

    auth_strategy=keystone

#### EM [glance]

    api_servers = http://controller:9292

#### EM [keystone_authtoken]
```
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = novaPass
```
#### EM [libvirt] descomente: virt_type=kvm  para

    virt_type=qemu

#### EM [oslo_concurrency] descomente

    lock_path=/var/lib/nova/tmp

#### EM [placement]
```
auth_url = http://controller:5000
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
username = placement
password = placementPass
```
#### EM [vnc]

    enabled = true
    server_listen = 0.0.0.0
    server_proxyclient_address = $my_ip
    novncproxy_base_url = http://controller:6080/vnc_auto.html

#### EM [wsgi]

    api_paste_config = /etc/nova/api-paste.ini

--

    chmod 640 /etc/nova/nova.conf
    chgrp nova /etc/nova/nova.conf
    semanage port -a -t http_port_t -p tcp 8778
    
    firewall-cmd --permanent --add-port={6080,6081,6082,8774,8775,8778}/tcp
    firewall-cmd --reload
    
    systemctl enable --now openstack-nova-compute
    systemctl status openstack-nova-compute [servico ATIVO]
    
    systemctl enable libvirtd.service openstack-nova-compute.service
    systemctl start libvirtd.service openstack-nova-compute.service
    systemctl startus libvirt.service openstack-nova-compute.service

## CONTROLLER ...

    su -s /bin/bash nova -c "nova-manage cell_v2 discover_hosts"

### user:stack

    su - stack
    source admin-openrc
    
    checar serviços
    openstack compute service list [nova-compute ON]
    openstack catalog list [nova,keystone,glance,placement]

### user:root 

    nova-status upgrade check [32;27]

---
---











































































































