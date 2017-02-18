Instalando OpenStack no VirtualBox
-------

Source: 

> http://docs.openstack.org/icehouse/training-guides/content/building-training-cluster.html

> http://docs.openstack.org/newton/install-guide-ubuntu/

Ambiente Utilizado
-------

 - Ubuntu 14.04.4 LTS
 - 12 GB DDR3
 - Intel(R) Core(TM) i7-3770S CPU @ 3.10GHz

Instalação do VirtualBox
------------------------

    sudo apt-get update; sudo apt-get install virtualbox
   
   O VirtualBox requere que o securete boot seja desativado. Se sua placa mãe suportar UEFI, e este estiver habilitado, você precisa desabilitá-lo. Por sorte, o instalador te ajudar neste processo. Ele configura para você o MOK ([Machine Owener Key](https://firmware.intel.com/blog/using-mok-and-uefi-secure-boot-suse-linux)), perguntando um password e guiando através do restart. Quando a tela para verificação do MOK for exibida, alguns **carectéres do password serão solicitados por vez em ordem aleatória**.
   Também é necessário verificar se sua máquina suporta VT-x (no caso de processadores Intel Core i5-7, Intel Virtualization Technology). Caso suporte, tenha certeza que esta ativado no BIOS. Caso sua máquina não suporte virtualização, será necessário desativar esta opção durante a criação das VMs, além disso não será possível utilizar a versão de 64 bits, apenas a de 32 bits. Para desabilitar a VT-x, desabilite esta opção nas configurações da VM na sessão **Sistema > Aceleração**.

Conceitos Básicos
-----------------

A instalação do OpenStack será feita através da instalação de três máquinas virtuais, cada uma com um papel específico, eles são: Compute, Controller e Network.

Serão necessários criar três interfaces de redes virtuais através do VirtualBox:

- **vboxnet0** - OpenStack management network - host static IP 170.10.10.1
- **vboxnet1** - VM conf. network - host static IP 171.10.10.1
- **vboxnet2** - VM external network access (host machine) 172.10.10.1

Antes de criar estas redes, verifique se sua máquina física não utiliza IPs conflitantes com as networks criadas.

Criando as Redes
-------

No VirtualBox navegue **Arquivo > Preferencias > Rede > Redes Exclusivas de Hospedeiro**. Adicione as três redes, para então configurar cada uma individualmente:

**vboxnet0**

 - **Endereço IPv4**: 170.10.10.1
 - **Máscara de Rede IPv4**: 255.255.255.0
 - **Endereço IPv6**:
 - **Tamanho da Máscara de Rede IPv6**: 0


**vboxnet1**

 - **Endereço IPv4**: 171.10.10.1
 - **Máscara de Rede IPv4**: 255.255.255.0
 - **Endereço IPv6**:
 - **Tamanho da Máscara de Rede IPv6**: 0

**vboxnet2**

 - **Endereço IPv4**: 172.10.10.1
 - **Máscara de Rede IPv4**: 255.255.255.0
 - **Endereço IPv6**:
 - **Tamanho da Máscara de Rede IPv6**: 0

Durante a criação das instâncias, na etapa de configuração dos adaptadores de rede para cada uma, a ordem dos adaptadores é a mesma ordem que as interfaces serão enumeradas no Sistema Operacional. Por exemplo Adaptador 1 equivale a enp0s3,  Adaptador 2 enp0s8,  Adaptador 3 enp0s9 e Adaptador 4 enp0s10.

Instalação das VMs
-------
Para a instalação das VMs, o OS utilizado será [Ubuntu 16.04.1 LTS](http://releases.ubuntu.com/16.04/).

**Control Node**
Requisítos recomendados: 4096 MB RAM e 20 GB de HDD, 2 ou mais cpus

Na janela principal do VirtualBox

 - Clique em Novo
 - Defina um nome para a VM (Control Node OS), Tipo (Linux), Versão (Ubuntu 64bits)
 - Crie um disco virtual do tipo VDI com alocação dinâmica.

Configure os adaptadores para as redes:

 - Clique com o botão direito na VM criada > Configurações
 - Na aba Rede

Adaptador 1: vboxnet0
![Configurando Adaptador 1](https://i.snag.gy/RFAZV9.jpg)

Adaptador 2: vboxnet2
![Configurando Adaptador 2](https://i.snag.gy/5NldBF.jpg)

Adaptador 3:  NAT
![Configurando Adaptador 3](https://i.snag.gy/KksZ7R.jpg)

Os Endereços MAC são gerados pelo VirtualBox

Na aba Armazenamento

 - Selecione a image do Ubuntu baixada
![Selecione a imagem do Ubuntu a ser instalada](https://i.snag.gy/knNogY.jpg)

**Inicie a VM**

Durante a instalação do Sistema algumas opções de packages serão oferecidas para instalação, aceite apenas a instalação do servidor/cliente SSH (para selecionar pressione ESPAÇO não ENTER, ENTER apenas prossegue com a instalação). Demais packages não precisam ser instalados, isto evitará conflitos durante a instalação dos serviços do OpenStack.
Quando se esta instalando o Ubuntu, será solicitado sua placa de rede primária. Selecione aquela correspondente ao Adaptador configurado para o NAT (Se você configurou o Control Node OS como acima, esta será eth2).

Uma vez logado, prossiga com a instalação dos componentes necessários para rodar: Keystone (Identity), Nova (Compute), Neutron (Quantum)

Dicas:

 - Para evitar problemas, salve o estado da instância ao invés de desligá-la totalmente. 
 - Faça Snapshots regulares, isto te permite voltar a uma versão segura em caso de defeitos.

No terminal do Control Node OS no VirtualBox:

    sudo vi /etc/network/interfaces

Edite o arquivo para que este fique:

    # This file describes the network interfaces available on your system
	# and how to activate them. For more information, see interfaces(5).

	# The loopback network interface
	auto lo
	iface lo inet loopback

	# The primary network interface
	# Esta é a interface de NAT, com acesso a internet
	auto enp0s9
	iface enp0s9 inet dhcp

	# VirtualBox vboxnet0 - OpenStack Management Network
	# (VirtualBox Network Adaptador 1)
	auto enp0s3
	iface enp0s3 inet static
	address 170.10.10.51
	netmask 255.255.255.0

	# VirtualBox vboxnet2 - OpenStack External Network
	# (VirtualBox Network Adaptador 2)
	auto enp0s8
	iface enp0s8 inet static
	address 172.10.10.51
	netmask 255.255.255.0

Coloque ambas as interfaces UP

    sudo ifup enp0s3;
    sudo ifup enp0s8

A partir de agora você é capaz de acessar a Control Node OS através de sua máquina usando SSH, para isto, a partir da sua máquina local:

    ssh control@170.10.10.51

 Atualize a lista de pacotes e repositórios.

    sudo apt-get update

Atualize Ubuntu Cloud Archive para Newton (Mais recente release do OpenStack)
O [Cloud Archive](https://wiki.ubuntu.com/OpenStack/CloudArchive) permite e facilita a instalação das ferramentas do OpenStack (o repositório dependerá da sua versão do ubuntu).
Algumas das ferramentas e funcionalidades apenas estão disponíveis na versão Newton, e esta foi lançada para o ubuntu 16.04.

    sudo apt-get install ubuntu-cloud-keyring software-properties-common python-software-properties;
    sudo add-apt-repository cloud-archive:newton;
    sudo apt-get update

Reinicie a VM:

    sudo reboot

Instale o cli para OpenStack, ele será utilizado para testar o sucesso da instalação dos serviçoes:

    sudo apt-get install python-openstackclient 

Relogue e então Instale os pacotes vlan e bridge-utils:

    sudo apt-get install vlan bridge-utils

Instale NTP:

    sudo apt-get install -y ntp

Configure o servidor NTP para o Control Node OS:

    sudo sed -i 's/server 0.ubuntu.pool.ntp.org/#server 0.ubuntu.pool.ntp.org/g' /etc/ntp.conf;
    sudo sed -i 's/server 1.ubuntu.pool.ntp.org/#server 1.ubuntu.pool.ntp.org/g' /etc/ntp.conf;
    sudo sed -i 's/server 2.ubuntu.pool.ntp.org/#server 2.ubuntu.pool.ntp.org/g' /etc/ntp.conf;
    sudo sed -i 's/server 3.ubuntu.pool.ntp.org/#server 3.ubuntu.pool.ntp.org/g' /etc/ntp.conf;
    sudo sed -i 's/server ntp.ubuntu.com/server 170.10.10.51/g' /etc/ntp.conf
  

**MySQL**

Instale e configure o MySQL (Dica: Defina password do root como **password**, no caso de você esquecer):

    sudo apt-get install -y mysql-server python-mysqldb

Configure para aceitar  requisições não locais:

    sudo sed -i 's/127.0.0.1/0.0.0.0/g' /etc/mysql/mysql.conf.d/mysqld.cnf;
    sudo service mysql restart

Crie os banco de dados para os serviços:

    mysql -u root -p

Na linha de comando do MySQL:

    CREATE DATABASE keystone;
    GRANT ALL ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS';
    
    CREATE DATABASE glance;
    GRANT ALL ON glance.* TO 'glance'@'%' IDENTIFIED BY 'GLANCE_DBPASS';
    
    CREATE DATABASE neutron;
    GRANT ALL ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'NEUTRON_DBPASS';
    
    CREATE DATABASE nova;
    GRANT ALL ON nova.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
    
    CREATE DATABASE nova_api;
    GRANT ALL ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
    
    quit;

**Instale RabbitMQ**:

    sudo apt-get install -y rabbitmq-server
    sudo rabbitmqctl add_user openstack RABBIT_PASS
    sudo rabbitmqctl set_permissions openstack ".*" ".*" ".*"

**Instale Memcached**

    sudo apt-get install memcached python-memcache
    
    sudo sed -i 's/-l 127.0.0.1/-l 170.10.10.51/g' /etc/memcached.conf;
    
    sudo service memcached restart; 

**Keystone**

Instale o Keystone:

    sudo apt-get install keystone

Configure-o para utilizar o MySQL:

    sudo vi /etc/keystone/keystone.conf

Na seção [database], faça connection igual:

    connection = mysql://keystone:KEYSTONE_DBPASS@170.10.10.51/keystone

Na seção [token], faça provider igual:

    provider = fernet

Sincronize e Populate o Banco de Dados do Keystone:

    sudo keystone-manage db_sync

Inicie o repositório para as chaves Fernet:

    sudo keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
    sudo keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

Bootstrap o Identity:

    sudo keystone-manage bootstrap --bootstrap-password ADMIN_PASS --bootstrap-admin-url http://170.10.10.51:35357/v3/ --bootstrap-internal-url http://170.10.10.51:35357/v3/ --bootstrap-public-url http://170.10.10.51:5000/v3/ --bootstrap-region-id RegionOne

Configure o Apache ServerName para o ip da máquina:

    sudo sed -i 'ServerName 170.10.10.51' /etc/apache2/apache2.conf;
    sudo service apache2 restart
    
Remova o BD criado pelo keystone para SQLite3:

    sudo rm -f /var/lib/keystone/keystone.db

Configure as variaveis de ambiente para se autenticar utilizando o Keystone V3, crie um arquivo keystone_admin.rc para armazenar estas informações :

    echo "export OS_USERNAME=admin;
	  export OS_PASSWORD=ADMIN_PASS;
	  export OS_PROJECT_NAME=admin;
	  export OS_USER_DOMAIN_NAME=default;
	  export OS_PROJECT_DOMAIN_NAME=default;
	  export OS_AUTH_URL=http://170.10.10.51:35357/v3;
	  export OS_IDENTITY_API_VERSION=3;
	  export OS_IMAGE_API_VERSION=2" > ~/keystone_admin.rc

	  source ~/keystone_admin.rc

Verifique se o Keystone foi instalado com sucesso rodando:

    openstack user list

Crie os projetos e usuários para os outros serviços:

    openstack project create --domain default --description "Service Project" service;
    openstack project create --domain default --description "Demo Project" demo;
    openstack user create --domain default --password DEMO_PASS demo;
    openstack role create user;
    openstack role add --project demo --user demo user

**Glance**

Crie o usuário glance e adicione o role admin nele:

    openstack user create --domain default --password GLANCE_PASS glance;
    openstack role add --project service --user glance admin

Registre o serviço Glance no BD:

    openstack service create --name glance --description "OpenStack Image" image

Crie os endpoints necessários para o Glance:

    openstack endpoint create --region RegionOne image public http://170.10.10.51:9292;
    openstack endpoint create --region RegionOne image internal http://170.10.10.51:9292;
    openstack endpoint create --region RegionOne image admin http://170.10.10.51:9292

Instale o Glance:

    sudo apt-get install glance

Na seção `[database]` do `/etc/glance/glance-api.conf`, altere:

    connection = mysql://glance:GLANCE_DBPASS@170.10.10.51/glance

Na seção `[keystone_authtoken]` Altere:

    auth_uri = http://170.10.10.51:5000
	memcached_servers = 170.10.10.51:11211
	auth_type = password

Na seção `[keystone_authtoken]` adicione:

	auth_url = http://170.10.10.51:35357
	project_domain_name = default
	user_domain_name = default
	project_name = service
	username = glance
	password = GLANCE_PASS

Na seção `[paste_deploy]`:

    flavor = keystone

Na seção `glance_store`:

    stores = file,http
	  default_store = file
	  filesystem_store_datadir = /var/lib/glance/images/

Faça o mesmo para o arquivo `/etc/glance/glance-registry.conf`, exceto a seção `[glance_store]`

Sincronize as informações do BD:

    sudo glance-manage db_sync

Restart Glance:

    sudo service glance-registry restart
    sudo service glance-api restart

**Registre uma imagem no glance**
Baixe a imagem:

    wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img

Registre a Image no Glance:

    openstack image create "cirros" --file cirros-0.3.4-x86_64-disk.img --disk-format qcow2 --container-format bare --public

> Caso você tenha problema nesta etapa, verifique se as credenciais do glance foram criadas corretamente, usuário, password

**Configurando o Nova**

Crie o usuário nova e adicione o role admin nele:

    openstack user create --domain default --password NOVA_PASS nova;
    openstack role add --project service --user nova admin

Registre o serviço Nova no BD:

    openstack service create --name nova --description "OpenStack Compute" compute

Crie os endpoints necessários para o Nova:

    openstack endpoint create --region RegionOne compute public http://170.10.10.51:8774/v2.1/%\(tenant_id\)s;
    openstack endpoint create --region RegionOne compute internal http://170.10.10.51:8774/v2.1/%\(tenant_id\)s;
    openstack endpoint create --region RegionOne compute admin http://170.10.10.51:8774/v2.1/%\(tenant_id\)s

Instale o Nova

    sudo apt-get install nova-api nova-conductor nova-consoleauth nova-novncproxy nova-scheduler

Na seção `[database]` do `/etc/nova/nova.conf`, altere:

    connection = mysql://nova:NOVA_DBPASS@170.10.10.51/nova

Na seção `[api_database]` altere:

    connection = mysql://nova:NOVA_DBPASS@170.10.10.51/nova_api

Na seção `[DEFAULT]` adicione:

    auth_strategy = keystone
    transport_url = rabbit://openstack:RABBIT_PASS@170.10.10.51
    use_neutron = True
    firewall_driver = nova.virt.firewall.NoopFirewallDriver
    metadata_listen_port = 8775
    metadata_host = 170.10.10.51
    metadata_manager = nova.api.manager.MetadataManager
    metadata_listen = 0.0.0.0
    metadata_port = 8775


Na seção `[keystone_authtoken]` adicione (crie a seção se não existir):

    auth_uri = http://170.10.10.51:5000
	auth_url = http://170.10.10.51:35357
	memcached_servers = 170.10.10.51:11211
	auth_type = password
	project_domain_name = default
	user_domain_name = default
	project_name = service
	username = nova
	password = NOVA_PASS

Na seção `[vnc]` altere:

    vncserver_listen = 170.10.10.51
	vncserver_proxyclient_address = 170.10.10.51

Na seção `[glance]` altere:

    api_servers = http://170.10.10.51:9292

Na seção `[oslo_concurrency]` altere:

    lock_path = /var/lib/nova/tmp

Feche e salve o arquivo

Sincronize o BD:

    sudo nova-manage api_db sync;
    sudo nova-manage db sync

Restart o Nova:

    sudo service nova-api restart;
	sudo service nova-consoleauth restart;
	sudo service nova-scheduler restart;
	sudo service nova-conductor restart;
	sudo service nova-novncproxy restart

Verifique se o OpenStack cli consegue se comunicar o Nova:

    openstack server list

Se nenhum erro ocorrer, e nada for retornado, o Nova respondeu corretamente.

**Configurando o Neutron**

Crie o usuário neutron e adicione o role admin nele:

    openstack user create --domain default --password NEUTRON_PASS neutron;
    openstack role add --project service --user neutron admin

Registre o serviço Neutron no BD:

    openstack service create --name neutron --description "OpenStack Networking" network

Crie os endpoints necessários para o Neutron:

    openstack endpoint create --region RegionOne network public http://170.10.10.52:9696;
    openstack endpoint create --region RegionOne network internal http://170.10.10.52:9696;
    openstack endpoint create --region RegionOne network admin http://170.10.10.52:9696

**Instale o Horizon**

    sudo apt-get install openstack-dashboard

No arquivo `/etc/openstack-dashboard/local_settings.py` altere:

    ALLOWED_HOSTS = ['*', ]
    SESSION_ENGINE = 'django.contrib.sessions.backends.file'
	OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
	OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
	OPENSTACK_API_VERSIONS = {
	    "identity": 3,
	    "image": 2,
	    "volume": 2,
	}
	OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "default"
	OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
	
	OPENSTACK_NEUTRON_NETWORK = {
	    ...
	    'enable_router': False,
	    'enable_quotas': False,
	    'enable_ipv6': False,
	    'enable_distributed_router': False,
	    'enable_ha_router': False,
	    'enable_lb': False,
	    'enable_firewall': False,
	    'enable_vpn': False,
	    'enable_fip_topology_check': False,
	}
	
	TIME_ZONE = "America/Recife"

Salve e feche

Agora altere o arquivo `/etc/apache2/conf-avaliable/openstack-dashboard.conf`, adicionado a seguinte linha ao início do arquivo:

    WSGIApplicationGroup %{GLOBAL}

Restart as configurações Web:

    sudo service apache2 reload
    sudo service apache2 restart

Verifique se o serviço foi instalado corretamente, acesse do seu Browser:

    http://170.10.10.51/horizon

**Network Node**
Requisítos mínimos: 1024 MB RAM e 8 GB de HDD

Adaptador 1: vboxnet0
![enter image description here](https://i.snag.gy/tU9Slg.jpg)

Adaptador 2: vboxnet1
![enter image description here](https://i.snag.gy/wRfLGg.jpg)

Adaptador 3: vboxnet2
![enter image description here](https://i.snag.gy/7oAMc9.jpg)

Adaptador 4: NAT
![enter image description here](https://i.snag.gy/q2hWuJ.jpg)

Verifique que o seu Controller Node Esta Ligado.

No terminal do Network Node OS no VirtualBox:

    sudo vi /etc/network/interfaces

Edite o arquivo para que este fique:

    # This file describes the network interfaces available on your system
	# and how to activate them. For more information, see interfaces(5).

	# The loopback network interface
	auto lo
	iface lo inet loopback

	# The primary network interface
	# Esta é a interface de NAT, com acesso a internet
	auto enp0s10
	iface enp0s10 inet dhcp

	# VirtualBox vboxnet0 - OpenStack Management Network
	# (VirtualBox Network Adaptador 1)
	auto enp0s3
	iface enp0s3 inet static
	address 170.10.10.52
	netmask 255.255.255.0

	# VirtualBox vboxnet1 - OpenStack API Network
	# (VirtualBox Network Adaptador 2)
	auto enp0s8
	iface enp0s8 inet static
	address 171.10.10.52
	netmask 255.255.255.0

	# VirtualBox vboxnet2 - For exposing external network
	# (VirtualBox Network Adaptador 2)
	auto enp0s9
	iface enp0s9 inet static
	address 172.10.10.52
	netmask 255.255.255.0

Coloque ambas as interfaces UP

    sudo ifup enp0s3;
    sudo ifup enp0s8;
    sudo ifup enp0s9

 Atualize a lista de pacotes e repositórios.

    sudo apt-get update

Atualize Ubuntu Cloud Archive para Newton

    sudo apt-get install ubuntu-cloud-keyring software-properties-common python-software-properties;
    sudo add-apt-repository cloud-archive:newton;
    sudo apt-get update

Reinicie a VM:

    sudo reboot

Relogue e então Instale os pacotes vlan e bridge-utils:

    sudo apt-get install vlan bridge-utils

Instale NTP:

    sudo apt-get install -y ntp

Configure o servidor NTP para o Control Node OS:

    sudo sed -i 's/server 0.ubuntu.pool.ntp.org/#server 0.ubuntu.pool.ntp.org/g' /etc/ntp.conf;
    sudo sed -i 's/server 1.ubuntu.pool.ntp.org/#server 1.ubuntu.pool.ntp.org/g' /etc/ntp.conf;
    sudo sed -i 's/server 2.ubuntu.pool.ntp.org/#server 2.ubuntu.pool.ntp.org/g' /etc/ntp.conf;
    sudo sed -i 's/server 3.ubuntu.pool.ntp.org/#server 3.ubuntu.pool.ntp.org/g' /etc/ntp.conf;
    sudo sed -i 's/server ntp.ubuntu.com/server 170.10.10.51/g' /etc/ntp.conf

Instale o Neutron

    sudo apt-get install neutron-server neutron-plugin-ml2 neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent

Na seção `[database]` do arquivo `/etc/neutron/neutron.conf`, altere:

    connection = mysql://neutron:NEUTRON_DBPASS@170.10.10.51/neutron

Na seção `[DEFAULT]` altere/crie:

    core_plugin = ml2
	service_plugins =
	transport_url = rabbit://openstack:RABBIT_PASS@170.10.10.51
	auth_strategy = keystone
	notify_nova_on_port_status_changes = True
	notify_nova_on_port_data_changes = True

Na seção `[keystone_authtoken]` altere/crie:

    auth_uri = http://170.10.10.51:5000
	auth_url = http://170.10.10.51:35357
	memcached_servers = 170.10.10.51:11211
	auth_type = password
	project_domain_name = default
	user_domain_name = default
	project_name = service
	username = neutron
	password = NEUTRON_PASS

Na seção `[nova]` altere/crie:

    auth_url = http://170.10.10.51:35357
	auth_type = password
	project_domain_name = default
	user_domain_name = default
	region_name = RegionOne
	project_name = service
	username = nova
	password = NOVA_PASS

Configure o Modular Layer 2 (ML2):

Na seção `[ml2]` do arquivo `/etc/neutron/plugins/ml2/ml2_conf.ini`, altere:

    type_drivers = flat,vlan
    tenant_network_types =
    mechanism_drivers = linuxbridge
    extension_drivers = port_security

Na seção `[securitygroup]`, altere/crie:

    enable_ipset = True

Configure o Linux bridge Agent

Na seção [linux_bridge] do arquivo `/etc/neutron/plugins/ml2/linuxbridge_agent.ini`, altere:

    physical_interface_mappings = provider:enp0s9
    
Onde enp0s9 é a interface referente a rede que você configurou para ser sua external network

Na seção `[securitygroup]`, altere:

    enable_security_group = True
    firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

Na seção `[vxlan]`, altere:

    enable_vxlan = False

Salve e feche o arquivo.

Configure o DHCP Agent

Na seção `[DEFAULT]` do arquivo `/etc/neutron/dhcp_agent.ini`, altere:

    interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
	dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
	enable_isolated_metadata = True

Salve o arquivo e feche

Configure o metadata agent

Na seção `[DEFAULT]` do arquivo `/etc/neutron/metadata_agent.ini`, altere:

    nova_metadata_ip = 170.10.10.51
    nova_metadata_port = 8775
    metadata_proxy_shared_secret = METADATA_SECRET

Salve o arquivo e feche

Logue no Control Node

    ssh control@170.10.10.51

Na seção `[neutron]` do arquivo `/etc/nova/nova.conf`, altere/crie:

    url = http://170.10.10.52:9696
    auth_url = http://170.10.10.51:35357
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = neutron
    password = NEUTRON_PASS
    service_metadata_proxy = True
    metadata_proxy_shared_secret = METADATA_SECRET

Restart o Nova:

    sudo service nova-api restart

Deslogue do Control Node e retorne para o Network Node.

No Network Node, restart o Neutron e atualize a BD:

    sudo neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head;
    sudo service neutron-server restart;
	sudo service neutron-linuxbridge-agent restart;
	sudo service neutron-dhcp-agent restart;
	sudo service neutron-metadata-agent restart
 
**Compute Node**
Requisítos mínimos: 1 GB RAM e 8GB de HDD (Se você tiver disponibilidade, use mais alocação para a memória RAM/HD, esta VM é a que realizara a criação das Máquinas Virtuais pelo OpenStack, isso se voce pretende criar varias VMs)

Adaptador 1: vboxnet0
![enter image description here](https://i.snag.gy/3k8Upq.jpg)

Adaptador 2: vboxnet1
![enter image description here](https://snag.gy/pvqzfw.jpg)

Adaptador 3: NAT
![enter image description here](https://snag.gy/CGewkd.jpg)

Adaptador 4: vboxnet2
![enter image description here](https://i.snag.gy/sQwHie.jpg)

Verifique que o seu Controller e Network Node estão ligados.

No terminal do Compute Node OS no VirtualBox:

    sudo vi /etc/network/interfaces

Edite o arquivo para que este fique:

    # This file describes the network interfaces available on your system
	# and how to activate them. For more information, see interfaces(5).

	# The loopback network interface
	auto lo
	iface lo inet loopback

	# The primary network interface
	# Esta é a interface de NAT, com acesso a internet
	auto enp0s9
	iface enp0s9 inet dhcp

	# VirtualBox vboxnet0 - OpenStack Management Network
	# (VirtualBox Network Adaptador 1)
	auto enp0s3
	iface enp0s3 inet static
	address 170.10.10.53
	netmask 255.255.255.0

	# VirtualBox vboxnet1 - OpenStack API Network
	# (VirtualBox Network Adaptador 2)
	auto enp0s8
	iface enp0s8 inet static
	address 171.10.10.53
	netmask 255.255.255.0

	# VirtualBox vboxnet2 - OpenStack External Network
	# (VirtualBox Network Adaptador 4)
	auto enp0s10
	iface enp0s10 inet static
	address 172.10.10.53
	netmask 255.255.255.0

Coloque todas as interfaces UP

    sudo ifup enp0s3;
    sudo ifup enp0s8;
    sudo ifup enp0s9;
    sudo ifup enp0s10

Da sua máquina logue no Compute Node:

    ssh compute@170.10.10.53

 Atualize a lista de pacotes e repositórios.

    sudo apt update

Atualize Ubuntu Cloud Archive para Newton

    sudo apt install ubuntu-cloud-keyring software-properties-common python-software-properties;
    sudo add-apt-repository cloud-archive:newton;
    sudo apt update

Instale NTP:

    sudo apt install -y ntp

Configure o servidor NTP para o Control Node OS:

    sudo sed -i 's/server 0.ubuntu.pool.ntp.org/#server 0.ubuntu.pool.ntp.org/g' /etc/ntp.conf;
    sudo sed -i 's/server 1.ubuntu.pool.ntp.org/#server 1.ubuntu.pool.ntp.org/g' /etc/ntp.conf;
    sudo sed -i 's/server 2.ubuntu.pool.ntp.org/#server 2.ubuntu.pool.ntp.org/g' /etc/ntp.conf;
    sudo sed -i 's/server 3.ubuntu.pool.ntp.org/#server 3.ubuntu.pool.ntp.org/g' /etc/ntp.conf;
    sudo sed -i 's/server ntp.ubuntu.com/server 170.10.10.51/g' /etc/ntp.conf

Reinicie a VM:

    sudo reboot

Instale o Nova Compute:

    sudo apt install nova-compute

Na seção `[DEFAULT]` do arquivo `/etc/nova/nova.conf`, altere:

    transport_url = rabbit://openstack:RABBIT_PASS@170.10.10.51
    auth_strategy = keystone
    my_ip = 170.10.10.53
    use_neutron = True
    firewall_driver = nova.virt.firewall.NoopFirewallDriver
    metadata_host = 170.10.10.51
    metadata_port = 8775
    metadata_manager = nova.api.manager.MetadataManager

Remove `log-dir` da seção `[DEFAULT]`

Na seção `[database]`, altere:

    connection=mysql://nova:NOVA_DBPASS@170.10.10.51/nova

Na seção `[api_database]`, altere:

    connection=mysql://nova:NOVA_DBPASS@170.10.10.51/nova_api

Na seção `[keystone_authtoken]`, altere:

    auth_uri = http://170.10.10.51:5000
	auth_url = http://170.10.10.51:35357
	memcached_servers = 170.10.10.51:11211
	auth_type = password
	project_domain_name = default
	user_domain_name = default
	project_name = service
	username = nova
	password = NOVA_PASS

Na seção `[vnc]`, altere:

    enabled = True
	vncserver_listen = 0.0.0.0
	vncserver_proxyclient_address = $my_ip
	novncproxy_base_url = http://170.10.10.51:6080/vnc_auto.html

Na seção `[glance]`, altere:

    api_servers = http://170.10.10.51:9292

Na seção `[oslo_concurrency]`, altere:

    lock_path = /var/lib/nova/tmp

Feche o arquivo e salve

Verifique se a VM suporta KVM:

    egrep -c '(vmx|svm)' /proc/cpuinfo

Se o retorno foi `0`, não suporta, neste caso você tem que utilizar QEMU, para isto altere na seção `[libvirt]` no arquivo `/etc/nova/nova-compute.conf`

    virt_type = qemu

Na seção `[DEFAULT]`, adicione:

    vif_plugging_is_fatal=false
    vif_plugging_timeout=0

Isto impedira a falha da criação das instâncias em caso de o Nova Compute não conseguir uma resposta do vif plugging.

Restart o Nova Compute:

    sudo service nova-compute restart

Para verificar que o Controller reconhece o Compute, acesse o Controller, se autentique no OpenStack e verifique se o compute service reconhece o compute-os:

    openstack compute service list

Novamente no Compute Node OS

**Configure o Neutron**

Instale o Neutron Linux Bridge:

    sudo apt-get install neutron-linuxbridge-agent

Na seção `[DEFAULT]` do arquivo `/etc/neutron/neutron.conf`, altere:

    transport_url = rabbit://openstack:RABBIT_PASS@170.10.10.51
    auth_strategy = keystone
  
Na seção `[database]` comente as referências a connection, pois o Compute Node não precisa acessar o banco de dados diretamente.

Na seção `[keystone_authtoken]`, altere:

    auth_uri = http://170.10.10.51:5000
    auth_url = http://170.10.10.51:35357
    memcached_servers = 170.10.10.51:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = neutron
    password = NEUTRON_PASS

Salve e feche o arquivo

Na seção `[linux_bridge]` do arquivo `/etc/neutron/plugins/ml2/linuxbridge_agent.ini`, altere:

    physical_interface_mappings = provider:enp0s10

Neste caso enp0s10 representa a vboxnet2, sua External Network.

Na seção `[securitygroup]`, altere:

    enable_security_group = True
	firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

Na seção `[vxlan]`, altere:

    enable_vxlan = False

Salve o arquivo e feche.

Na seção `[neutron]` do arquivo `/etc/nova/nova.conf`, altere:

    url = http://170.10.10.52:9696
    auth_url = http://170.10.10.51:35357
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = neutron
    password = NEUTRON_PASS

Salve e feche.

Reinicie o Nova Compute e o Linux Bridge Agent:

    sudo service nova-compute restart;
    sudo service neutron-linuxbridge-agent restart

Logue no Control Node:

    ssh control@170.10.10.51

Verifique se todas as extensões foram carregadas corretamente:

    neutron ext-list

Verifique se os agentes do Neutron estão funcionando:

    openstack network agent list

Últimas configurações
-------

 - **ATENÇÃO**: Todos os nodes(Compute, Network e Control) devem ter a mesma configuração de horário, senão os heartbeats dos serviços não funcionarão, logo, os serviços não funcionarão.

Você pode atualizar a hora manualmente com o o comando: (substitua com a hora atual)
  
    sudo date --set "Thu Feb 16 13:48:33 BRT 2017"

Depois que você verificar que os horários estão iguais, logue no Controller Node:

    ssh control@170.10.10.51

Crie a rede **provider**:

    openstack network create  --share --provider-physical-network provider --provider-network-type flat provider

Crie uma subnet na rede:

    openstack subnet create --network provider --allocation-pool start=172.10.10.101,end=172.10.10.250 --dns-nameserver 8.8.4.4 --gateway 172.10.10.1 --subnet-range 172.10.10.0/24 provider

Crie o Flavor m1.nano:

    openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano

Modifique o grupo de segurança `default` para permitir os seguintes acessos:

    openstack security group rule create --proto icmp default;
    openstack security group rule create --proto tcp --dst-port 22 default
    
**Crie uma instância**

Para isto precisaremos do ID da rede provider que foi criada, neste caso, primeiro:

    openstack network list

Com o ID da rede provider em mãos, altere o espaço <**ID-PROVIDER**> no comando a seguir e execute-o:

    openstack server create --image cirros --flavor m1.nano --nic net-id=<ID-PROVIDER> --security-group default primeira_instancia

Após o comando ser executado, você pode verificar o estado da criação da instância através do comando:

    openstack server list

Se o estado da máquina é ACTIVE, a instância foi criada com sucesso.

Para ter acesso direto a instãncia, você pode obter um terminal VNC para acessá-la através do navegador, execute:

    openstack console url show primeira_instancia

O próximo passo é acessar através de uma conexão SSH:

Para isso, veja o IP fornecido para a instância:

    openstack server list

Utilizando as credenciais padrões do Cirros, acesse-o utilizando o comando ssh:

    ssh cirros@X.X.X.X

O password padrão para o sistema é `cubswin:)`

Agora para habilitar acesso à internet a partir das VMs, você precisa fazer com que a máquina hospedeira rodando o VirtualBox faça um NAT da rede host-only do virtualbox (172.10.10.0/24) para a rede que está conectada à internet (geralmente é a interface eth0). Para isto, na máquina hospedeira, execute (caso esteja usando Linux):

    sudo /sbin/iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    sudo /sbin/iptables -A FORWARD -i eth0 -o vboxnet2 -m state --state RELATED,ESTABLISHED -j ACCEPT
    sudo /sbin/iptables -A FORWARD -i vboxnet2 -o eth0 -j ACCEPT

Pronto! :)
