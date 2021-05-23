## Openstack Ubuntu 20.04 Focal

### OpenStack Victoria : Pré-Requisitos
##### Configure NTP Server (Chrony) - 
CONTROLLER NODE:

    apt -y install chrony
##### Editar arquivo e adicionar: vi /etc/chrony/chrony.conf

    pool controller iburst 
    allow 10.0.0.0/24
--

    systemctl restart chrony
    chronyc sources

Instalar e configurar mysql -> MariaDB

    apt -y install mariadb-server

##### Editar arquivo e adicionar: vi /etc/mysql/mariadb.conf.d/50-server.cnf

    #linha 104: Verificar as linhas
    character-set-server  = utf8mb4
    collation-server      = utf8mb4_general_ci
    
 --

     systemctl restart mariadb

##### Configure MariaDB

    mysql_secure_installation
Adicionar repositório openstack victoria:

    apt -y install software-properties-common
    add-apt-repository cloud-archive:victoria
    apt update && apt upgrade -y
Instalar RabbitMQ e Memcached

    apt -y install rabbitmq-server memcached python3-pymysql
adicionar usuário, senha(rabbitPass) e permissão para rabbitmq:

    rabbitmqctl add_user openstack rabbitPass
    rabbitmqctl set_permissions openstack ".*" ".*" ".*"
edite o arquivo: vi /etc/mysql/mariadb.conf.d/50-server.cnf

    #linha 28: mude:
    bind-address = 0.0.0.0
    #linha 40: descomente e mude:
    max_connections = 500
edite o arquivo: vi /etc/memcached.conf

    #linha 35: mude
    -l 0.0.0.0
--

    systemctl restart mariadb rabbitmq-server memcached

## Keystone
Criar database no mysql para keystone.

    mysql
    CREATE DATABASE keystone;
    GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'keystoneDBPass';
    GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'keystoneDBPass';
    exit
Instale os pacotes do keystone

    apt -y install keystone python3-openstackclient apache2 libapache2-mod-wsgi-py3 python3-oauth2client
Configure keystone, edite: vi /etc/keystone/keystone.conf

    #linha 436: descomente e esfecifique memcache server
    memcache_servers = controller:11211
    #linha 594: mude para conexão mysql-mariadb
    connection = mysql+pymysql://keystone:keystoneDBPass@controller/keystone
    #linha 2508: descomente
    provider = fernet
popular banco keystone e defina fernet key.

    su -s /bin/bash keystone -c "keystone-manage db_sync"
    
    keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
    keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
bootstrap keystone

    keystone-manage bootstrap --bootstrap-password adminPass \
     --bootstrap-admin-url http://controller:5000/v3/ \
     --bootstrap-internal-url http://controller:5000/v3/ \
     --bootstrap-public-url http://controller:5000/v3/ \
     --bootstrap-region-id RegionOne
Configure apache: vi /etc/apache2/apache2.conf

    #linha 70: especifique name server
    ServerName controller
--

    systemctl restart apache2
Criar arquivo de credenciais: vi admin-openrc

    export OS_PROJECT_DOMAIN_NAME=default  
    export OS_USER_DOMAIN_NAME=default  
    export OS_PROJECT_NAME=admin  
    export OS_USERNAME=admin  
    export OS_PASSWORD=adminPass  
    export OS_AUTH_URL=http://controller:5000/v3  
    export OS_IDENTITY_API_VERSION=3  
    export OS_IMAGE_API_VERSION=2  
    export PS1='\u@\h \W(keystone)\$ '
Dar  permissão ao arquivo

    chmod 600 ~/admin-openrc
    source admin-openrc
Adicionar projeto.

    openstack project create --domain default --description "Service Project" service
    openstack project list
## Glance
Adicionar usuarios para o serviço glance em keystone.

    openstack user create --domain default --project service --password glancePass glance
    openstack role add --project service --user glance admin
    openstack service create --name glance --description "OpenStack Image service" image
Criar os endpoints

    openstack endpoint create --region RegionOne image public http://controller:9292
    openstack endpoint create --region RegionOne image internal http://controller:9292
    openstack endpoint create --region RegionOne image admin http://controller:9292
    exit
Criar database para o serviço glance.

    mysql
    CREATE DATABASE glance;
    GRANT ALL PRIVILEGES ON glance.*  TO 'glance'@'localhost' IDENTIFIED BY 'glanceDBPass';
    GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'glanceDBPass';
    flush privileges;
    exit
Instale os pacotes do glance

    apt -y install glance
edite:

> mv /etc/glance/glance-api.conf /etc/glance/glance-api.conf.org vi
> vi /etc/glance/glance-api.conf
> 
adicione no arquivo:


    DEFAULT]
    bind_host = 0.0.0.0
    
    [glance_store]
    stores = file,http
    default_store = file
    filesystem_store_datadir = /var/lib/glance/images/
    
    [database]
    connection = mysql+pymysql://glance:glanceDBPass@controller/glance
    
    [keystone_authtoken]
    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = glance
    password = glancePass
    
    [paste_deploy]
    flavor = keystone
Dar as permissões.

    chmod 640 /etc/glance/glance-api.conf
    chown root:glance /etc/glance/glance-api.conf
    
    su -s /bin/bash glance -c "glance-manage db_sync"
    systemctl restart glance-api

Repositorio de Imagem

    mkdir -p /var/kvm/images
    wget http://cloud-images.ubuntu.com/releases/20.04/release/ubuntu-20.04-server-cloudimg-amd64.img -P /var/kvm/images
    openstack image create "Ubuntu2004-Official" --file /var/kvm/images/ubuntu-20.04-server-cloudimg-amd64.img --disk-format qcow2 --container-format bare --public

## Nova
Criar usuário e rules para nova.

    openstack user create --domain default --project service --password novaPass nova
    openstack role add --project service --user nova admin
    
    openstack user create --domain default --project service --password placementPass placement
    openstack role add --project service --user placement admin
    
    openstack service create --name nova --description "OpenStack Compute service" compute
    openstack service create --name placement --description "OpenStack Compute Placement service" placement
Criar os endpoints

    openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1/%\(tenant_id\)s
    openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1/%\(tenant_id\)s
    openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1/%\(tenant_id\)s
    
    openstack endpoint create --region RegionOne placement public http://controller:8778
    openstack endpoint create --region RegionOne placement internal http://controller:8778
    openstack endpoint create --region RegionOne placement admin http://controller:8778
Criar as databases para o serviço nova.

    mysql
    
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
Instalar os pacotes nova.

    apt -y install nova-api nova-conductor nova-scheduler nova-novncproxy placement-api python3-novaclient
Edite o arquivo:

> mv /etc/nova/nova.conf /etc/nova/nova.conf.org 
> vi /etc/nova/nova.conf

adicione no arquivo:

    [DEFAULT]
    my_ip = 10.0.0.11
    state_path = /var/lib/nova
    enabled_apis = osapi_compute,metadata
    log_dir = /var/log/nova
    
    transport_url = rabbit://openstack:rabbitPass@controller
    
    [api]
    auth_strategy = keystone
    
    [glance]
    api_servers = http://controller:9292
    
    [oslo_concurrency]
    lock_path = $state_path/tmp
    
    [api_database]
    connection = mysql+pymysql://nova:novaDBPass@controller/nova_api
    
    [database]
    connection = mysql+pymysql://nova:novaDBPass@controller/nova
    
    [keystone_authtoken]
    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = nova
    password = novaPass
    
    [placement]
    auth_url = http://controller:5000
    os_region_name = RegionOne
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = placement
    password = placementPass
    
    [wsgi]
    api_paste_config = /etc/nova/api-paste.ini
Dar as permissões.

    chmod 640 /etc/nova/nova.conf
    chgrp nova /etc/nova/nova.conf
Editar o arquivo:

> mv /etc/placement/placement.conf /etc/placement/placement.conf.org vi
> /etc/placement/placement.conf

Adicione no arquivo:

    [DEFAULT]
    debug = false
    
    [api]
    auth_strategy = keystone
    
    [keystone_authtoken]
    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = placement
    password = placementPass
    
    [placement_database]
    connection = mysql+pymysql://placement:placementDBPass@controller/placement
Dar as permissões.

    chmod 640 /etc/placement/placement.conf
    chgrp placement /etc/placement/placement.conf
Popular as databases do nova.

    su -s /bin/bash placement -c "placement-manage db sync"
    
    su -s /bin/bash nova -c "nova-manage api_db sync"
    
    su -s /bin/bash nova -c "nova-manage cell_v2 map_cell0"
    
    su -s /bin/bash nova -c "nova-manage db sync"
    
    su -s /bin/bash nova -c "nova-manage cell_v2 create_cell --name cell1"
    
    systemctl restart apache2
    
    for service in api conductor scheduler novncproxy; do  
    systemctl restart nova-$service  
    done
Verificar status dos serviços nova.

    openstack compute service list
Instalar pacotes do nova compute.

    apt -y install nova-compute nova-compute-kvm
edite o arquivo e adicione: vi /etc/nova/nova.conf

    [vnc]
    enabled = True
    server_listen = 0.0.0.0
    server_proxyclient_address = 10.0.0.11
    novncproxy_base_url = http://10.0.0.11:6080/vnc_auto.html
--

    systemctl restart nova-compute
Procure por nós compute:

    su -s /bin/bash nova -c "nova-manage cell_v2 discover_hosts"
mostre os status:

    openstack compute service list
## Neutron

    Criar usuário e regras para o serviço neutron:
    openstack user create --domain default --project service --password neutronUserPass neutron
    openstack role add --project service --user neutron admin

Criar serviço para neutron:

    openstack service create --name neutron --description "OpenStack Networking service" network
Criar os endpoints

    openstack endpoint create --region RegionOne network public http://controller:9696
    openstack endpoint create --region RegionOne network internal http://controller:9696
    openstack endpoint create --region RegionOne network admin http://controller:9696
Criar database para neutron.

    mysql
    
    ######## CentOs - databse criada ########
    
    CREATE DATABASE neutron;
    GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'neutronDBPass';
    GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'neutronDBPass';
    
    ######## Ubuntu- databse criada[~~ERROR - verificar ~~] ########
    mudar banco em neutron.conf
    (connection = mysql+pymysql://neutron:neutronDBPass@controller/neutron_ml2)
    create database neutron_ml2;
    grant all privileges on neutron_ml2.* to neutron@'localhost' identified by 'password';
    grant all privileges on neutron_ml2.* to neutron@'%' identified by 'password';
    
    flush privileges;
    exit
Instale os pacotes do serviço neutron:

    apt -y install neutron-server neutron-plugin-ml2 neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent python3-neutronclient

Edite o arquivo:

> mv /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org vi
> /etc/neutron/neutron.conf

Adicione:

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
    root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf
    
    [keystone_authtoken]
    www_authenticate_uri = controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = neutron
    password = neutronUserPass
    
    [database]
    connection = mysql+pymysql://neutron:neutronDBPass@controller/neutron_ml2
    
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
    lock_path = $state_path/tmp
Dar as permissões:

    chmod 640 /etc/neutron/neutron.conf
    chgrp neutron /etc/neutron/neutron.conf
Edite o arquivo: vi /etc/neutron/l3_agent.ini

    #linha 21: adicione
    interface_driver = linuxbridge
Edite o arquivo: vi /etc/neutron/dhcp_agent.ini

    #linha 21: adicione
    interface_driver = linuxbridge
    #linha 43: descomente
    dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
    #linha 52: descomente e mude
    enable_isolated_metadata = true
Edite o arquivo: vi /etc/neutron/metadata_agent.ini

    #linha 22: descomente e especifique  nova api server
    nova_metadata_host = controller  
    #linha 34: descomente e escolha uma senha para proxy-metadata
    metadata_proxy_shared_secret = neutronMetaPass
    #linha 307: descomente e especifique memcache server
    memcache_servers = controller:11211
Edite o arquivo: vi /etc/neutron/plugins/ml2/ml2_conf.ini
#linha 154: adicione

    [ml2]  
    type_drivers = flat,vlan,vxlan  
    tenant_network_types =  
    mechanism_drivers = linuxbridge  
    extension_drivers = port_security
Edite o arquivo: vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini

    #linha 225: adicione
    [securitygroup]  
    enable_security_group = True  
    firewall_driver = iptables  
    enable_ipset = True
    
    #linha 284: adicione
    local_ip = 10.0.0.11
Edite nova.conf novamente e adicione:

    #EM [DEFAULT] adicione
    use_neutron = True
    linuxnet_interface_driver = nova.network.linux_net.LinuxBridgeInterfaceDriver
    firewall_driver = nova.virt.firewall.NoopFirewallDriver
    vif_plugging_is_fatal = True
    vif_plugging_timeout = 300
    
    #[ADICIONE NO FINAL]
    [neutron]
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
Inicie os serviços neutron:

    ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
    su -s /bin/bash neutron -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugin.ini upgrade head"
    
    for service in server l3-agent dhcp-agent metadata-agent linuxbridge-agent; do  
    systemctl restart neutron-$service  
    systemctl enable neutron-$service  
    done
    systemctl restart nova-api nova-compute
Liste os agentes de rede:

    openstack network agent list

---
create a setting file for anonymous interface

> vi /etc/systemd/network/eth1.network

    [Match]
    Name=eth1
    
    [Network]
    LinkLocalAddressing=no
    IPv6AcceptRA=no
--

    systemctl restart systemd-networkd
edite: vi /etc/neutron/plugins/ml2/ml2_conf.ini

    #linha 206: adicione  
    [ml2_type_flat]  
    flat_networks = physnet1

edite: vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini

    #linha 190: adicione  
    [linux_bridge]  
    physical_interface_mappings = physnet1:eth1
    
    #linha 257: descomente e mude
    enable_vxlan = false
--

    systemctl restart neutron-linuxbridge-agent
Criar uma rede virtual:

    projectID=$(openstack project list | grep service | awk '{print $2}')
    
    openstack network create --project $projectID \  
    --share --provider-network-type flat --provider-physical-network physnet1 sharednet1
    
    openstack subnet create subnet1 --network sharednet1 \  
    --project $projectID --subnet-range 10.0.0.0/24 \  
    --allocation-pool start=10.0.0.200,end=10.0.0.254 \  
    --gateway 10.0.0.1 --dns-nameserver 10.0.0.10

--
Verifique as redes e subnets criadas.

    openstack network list
    openstack subnet list

##
##
## Criando projetos e usuarios.
Criando projeto:

    openstack project create --domain default --description "Meu Projeto" MeuProjeto

Criando usuário:

    openstack user create --domain default --project MeuProjeto --password MinhaSenha MeuUsuario

Criando role:

    openstack role create MinhaRole

Adicionando um usuário a role:

    openstack role add --project MeuProjeto --user MeuUsuario MinhaRole

Criando flavors:

    openstack flavor create --id 0 --vcpus 1 --ram 128 --disk 2 m1.MeuFlavor

### Criando Instâncias
Logar com suas credenciais.
source admin-openrc
Listar flavor,image,network,security-groups e keypair:

    openstack flavor list
    openstack image list
    openstack network list
    openstack security-groups list
    openstack keypair list

Criar keypair:

    ssh-keygen -q -N ""

Adicionar public-key:

    openstack keypair create --public-key ~/.ssh/id_rsa.pub MinhaChave

Criando instância:

    openstack server create --flavor m1.MeuFlavor --image cirros --security-group default --nic net-id=$netID --key-name MinhaChave Minha-Instancia

Listar instâncias:

    openstack server list


Configurar security-group para permitir acesso SSH e PING na instância.

    openstack security group rule create --protocol icmp --ingress default
    openstack security group rule create --protocol tcp --dst-port 22:22 secgroup01

Listar regras no security-group:

    openstack security group rule list
Obter link do vnc da sua instância:

    openstack console url show MinhaInstancia

##
##
## Horizon
Instale os pacote do horizon:

    apt -y install openstack-dashboard

Edite o arquivo: vi /etc/openstack-dashboard/local_settings.py

    #linha 99: mude memcache server
     'LOCATION': 'controller:11211',
    #linha 133: adicione
    SESSION_ENGINE = "django.contrib.sessions.backends.cache"
    #linha 126&127: 
    OPENSTACK_HOST = "controller"
    OPENSTACK_KEYSTONE_URL = "http://controller:5000/v3"
    #linha 131: mude timezone
    TIME_ZONE = "America/Cuiaba"
    #adicione no final do arquivo:
    OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
    OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'Default'

Reinicie apache:

    systemctl restart apache2
--
Permitir que usuários comuns acessem os detalhes das instancias [OPCIONAL].
Crie e adicione: vi /etc/nova/policy.json

    {
      "os_compute_api:os-extended-server-attributes": "rule:admin_or_owner",
    }
Dar as permissões ao arquivo criado:

    chgrp nova /etc/nova/policy.json
    chmod 640 /etc/nova/policy.json
    
    systemctl restart nova-api
Acessar o horizon pelo navegador:

    http://10.0.0.11/horizon
    http://controller/horizon *

## Compute em um 2° host
Configurar o novo host com duas interfaces de rede de acordo com a maquina controller.

Instalando pacotes compute no segundo nó:
apt -y install nova-compute nova-compute-kvm qemu-system-data
Edite:

> mv /etc/nova/nova.conf /etc/nova/nova.conf.org vi /etc/nova/nova.conf

Adicione no arquivo:

    [DEFAULT]
    my_ip = 10.0.0.31
    state_path = /var/lib/nova
    enabled_apis = osapi_compute,metadata
    log_dir = /var/log/nova
    
    transport_url = rabbit://openstack:rabbitPass@controller
    
    [api]
    auth_strategy = keystone
    
    [vnc]
    enabled = True
    server_listen = 0.0.0.0
    server_proxyclient_address = $my_ip
    novncproxy_base_url = http://10.0.0.11:6080/vnc_auto.html
    
    [glance]
    api_servers = http://controller:9292
    
    [oslo_concurrency]
    lock_path = $state_path/tmp
    
    [keystone_authtoken]
    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = nova
    password = novaPass
    
    [placement]
    auth_url = http://controller:5000
    os_region_name = RegionOne
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = placement
    password = placementPass
    
    [wsgi]
    api_paste_config = /etc/nova/api-paste.ini
Dê as permissões ao arquivo:

    chmod 640 /etc/nova/nova.conf
    chgrp nova /etc/nova/nova.conf
    
    systemctl restart nova-compute
### Execute no CONTROLLER
Procurar novos nós compute:

    su -s /bin/bash nova -c "nova-manage cell_v2 discover_hosts"

Verifique os nós compute:

    openstack compute service list

## Configure Neutron no Compute Node
Instale os pacote neutron:
apt -y install neutron-common neutron-plugin-ml2 neutron-linuxbridge-agent
Edite:

> mv /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org vi
> /etc/neutron/neutron.conf

Adicione no arquivo:

    [DEFAULT]
    core_plugin = ml2
    service_plugins = router
    auth_strategy = keystone
    state_path = /var/lib/neutron
    allow_overlapping_ips = True

    transport_url = rabbit://openstack:rabbitPass@controller
    
    [agent]
    root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf
    
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
    
    [oslo_concurrency]
    lock_path = $state_path/lock
Dar as permissões ao arquivo:

    chmod 640 /etc/neutron/neutron.conf
    chgrp neutron /etc/neutron/neutron.conf

Edite: vi /etc/neutron/plugins/ml2/ml2_conf.ini

    #linha 154: adicione
    [ml2]
    type_drivers = flat,vlan,vxlan  
    tenant_network_types =  
    mechanism_drivers = linuxbridge  
    extension_drivers = port_securit
Edite: vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini

    #linha 225:
    [securitygroup]  
    enable_security_group = True  
    firewall_driver = iptables  
    enable_ipset = True
    #linha 284:
    local_ip = 10.0.0.31

Edite: vi /etc/nova/nova.conf

    #adicione em [DEFAULT]
    use_neutron = True  
    linuxnet_interface_driver = nova.network.linux_net.LinuxBridgeInterfaceDriver  
    firewall_driver = nova.virt.firewall.NoopFirewallDriver  
    vif_plugging_is_fatal = True  
    vif_plugging_timeout = 300
    
    #adicione no final:
    [neutron]  
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
Execute:

    ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
    
    systemctl restart nova-compute neutron-linuxbridge-agent
    systemctl enable neutron-linuxbridge-agent

---
---
# CINDER

### CONTROLLER

    openstack user create --domain default --project service --password cinderPass cinder
    openstack role add --project service --user cinder admin
    openstack service create --name cinderv3 --description "OpenStack Block Storage" volumev3

Criar os endpoints:

    openstack endpoint create --region RegionOne volumev3 public http://controller:8776/v3/%\(tenant_id\)s
    openstack endpoint create --region RegionOne volumev3 internal http://controller:8776/v3/%\(tenant_id\)s
    openstack endpoint create --region RegionOne volumev3 admin http://controller:8776/v3/%\(tenant_id\)s

Criar database para o serviço cinder:

    mysql
    create database cinder;
    grant all privileges on cinder.* to cinder@'localhost' identified by 'cinderDBPass';
    grant all privileges on cinder.* to cinder@'%' identified by 'cinderDBPass';
    flush privileges;
    exit

instalar pacotes do cinder:

    apt -y install cinder-api cinder-scheduler python3-cinderclient

Renomeie o arquivo e crie:

> mv /etc/cinder/cinder.conf /etc/cinder/cinder.conf.org vi
> 
> /etc/cinder/cinder.conf

    [DEFAULT]
    my_ip = 10.0.0.11
    rootwrap_config = /etc/cinder/rootwrap.conf
    api_paste_confg = /etc/cinder/api-paste.ini
    state_path = /var/lib/cinder
    auth_strategy = keystone
    
    transport_url = rabbit://openstack:rabbitPass@controller
    enable_v3_api = True
    
    [database]
    connection = mysql+pymysql://cinder:cinderDBPass@controller/cinder
    
    [keystone_authtoken]
    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = cinder
    password = cinderPass
    
    [oslo_concurrency]
    lock_path = $state_path/tmp

--
Dar as permissões:

    chmod 640 /etc/cinder/cinder.conf
    chgrp cinder /etc/cinder/cinder.conf
--

    su -s /bin/bash cinder -c "cinder-manage db sync"
    systemctl restart cinder-scheduler
    openstack volume service list

### STORAGE
Instale os pacotes cinder:

    apt -y install cinder-volume python3-mysqldb python3-rtslib-fb

renomeie e crie:

> mv /etc/cinder/cinder.conf /etc/cinder/cinder.conf.org 
> vi /etc/cinder/cinder.conf

adicione:

    DEFAULT]
    my_ip = 10.0.0.41
    rootwrap_config = /etc/cinder/rootwrap.conf
    api_paste_confg = /etc/cinder/api-paste.ini
    state_path = /var/lib/cinder
    auth_strategy = keystone
    
    transport_url = rabbit://openstack:rabbitPass@controller
    enable_v3_api = True
    
    glance_api_servers = http://controller:9292
    
    enabled_backends =
    
    [database]
    connection = mysql+pymysql://cinder:cinderDBPass@controller/cinder
    
    [keystone_authtoken]
    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = cinder
    password = cinderPass
    
    [oslo_concurrency]
    lock_path = $state_path/tmp


--
Dar as permissões:

    chmod 640 /etc/cinder/cinder.conf
    chgrp cinder /etc/cinder/cinder.conf

    systemctl restart cinder-volume

#### VOLUME LVM (STORAGE NODE)
pvcreate /dev/sdb1
vgcreate -s 32M vg_volume01 /dev/sdb1

Instale:
apt -y install tgt thin-provisioning-tools

edite o arquivo cinder.conf: 

> vi /etc/cinder/cinder.conf

    #adicione na linha:
    enabled_backends = lvm
    #adicione no final:
    [lvm]
    target_helper = tgtadm
    target_protocol = iscsi
    target_ip_address = 10.0.0.41
    volume_group = vg_volume01
    volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
    volumes_dir = $state_path/volumes

reinicie o cinder:

    systemctl restart cinder-volume tgt

#### ADICIONE NO COMPUTE
Edite em nova.conf: 

> vi /etc/nova/nova.conf

#adicione no final

    [cinder]  
    os_region_name = RegionOne
--

    systemctl restart nova-compute

Crie um volume:

    openstack volume create --size 10 disk01


