=====================================================================
  OpenStack Ussuri Install (Controller, Network, Compute) on CentOS 8
=====================================================================

:Version: 1.0
:Source: https://github.com/conankiz/OpenStackUssuri-Centos8-Three-Node
:Keywords: Three node, Ussuri, Quantum, Nova, Keystone, Glance, Horizon, Cinder, OpenVSwitch, KVM, Centos 8 (64 bits).

Authors
==========

`Conankiz <http://www.linkedin.com/profile/>`_ 

::

      ------------+---------------------------+---------------------------+------------
                  |                           |                           |
              eth0|192.168.100.12         eth0|192.168.100.13         eth0|192.168.100.14
      +-----------+-----------+   +-----------+-----------+   +-----------+-----------+
      |    [ Control Node ]   |   |    [ Network Node ]   |   |    [ Compute Node ]   |
      |                       |   |                       |   |                       |
      |  MariaDB    RabbitMQ  |   |      Open vSwitch     |   |        Libvirt        |
      |  Memcached  httpd     |   |        L2 Agent       |   |     Nova Compute      |
      |  Keystone   Glance    |   |        L3 Agent       |   |      Open vSwitch     |
      |  Nova API             |   |     Metadata Agent    |   |        L2 Agent       |
      |  Neutron Server       |   |                       |   |                       |
      |  Metadata Agent       |   |                       |   |                       |
      +-----------------------+   +-----------+-----------+   +-----------------------+
                                          eth1|(UP with no IP)

Contributors
==========

=================================================== =======================================================

 
=================================================== =======================================================

Wana contribute ? Read the guide, send your contribution and get your name listed ;)
Main Components of OpenStack::


      +-------------------+------------------------------------------------------------------------------+
      | Service           | Code Name         | Description                                              |
      +-------------------+-------------------+----------------------------------------------------------+
      | Identity Service  | 	Keystone      | User Management                                          |
      +--------------+------------------------+----------------------------------------------------------+
      | Compute Service   | 	Nova          | Virtual Machine Management                               |
      +--------------+------------------------+----------------------------------------------------------+
      | Image Service     | 	Glance        | Manages Virtual image like kernel image or disk image    |
      +--------------+------------------------+----------------------------------------------------------+
      | Dashboard         | 	Horizon       | Provides GUI console via Web browse                      |
      +--------------+------------------------+----------------------------------------------------------+
      | Object Storage    | 	Swift         | Provides Cloud Storage                                   |
      +--------------+------------------------+----------------------------------------------------------+
      | Network Service   | 	Neutron       | Virtual Networking Management                            |
      +--------------+------------------------+----------------------------------------------------------+


 

Table of Contents
=================

::

  0. What is it?
  1. Requirements
  3. Controller Node
  4. Network Node
  5. Compute Node
  5. Your first VM
  6. Licensing
  7. Contacts
  8. Credits
  9. To do

0. What is it?
==============

OpenStack Ussuri Install Guide is an easy and tested way to create your own OpenStack platform. 

If you like it, don't forget to star it !

Status: Stable


1. Requirements
====================
:public network (Floating IP network): 192.168.100.0/24
:nternal network (on each node): no IP space, physical connection only (eth1)
:Node Role: NICs
:Control Node: eth0 (192.168.100.12), eth1 ()
:Network Node: eth0 (192.168.100.13), eth1 (), eth2 ()
:Compute Node: eth0 (192.168.100.14), eth1 ()

**Note 1:** Always use dpkg -s <packagename> to make sure you are using grizzly packages (version : 2013.1)

**Note 2:** This is my current network architecture, you can add as many compute node as you wish.

.. image:: http://i.imgur.com/Frsughe.jpg

2. Controller, Network, Compute
====================
- Update system
::

  # yum update -y ; reboot
   
- Edit the Hosts file on each server and set the below entries in case you don’t your local DNS server.
::

    192.168.100.12 controllernode.test.local controllernode
    192.168.100.14 computenode.test.local computenode
    192.168.100.13 networknode.test.local networknode

- Stop and disable firewalld & NetworkManager Service
Execute the beneath commands one after the another to stop and disable firewalld and NetworkManager Service on all nodes.
::

    ~]# systemctl stop firewalld 
    ~]# systemctl disable firewalld 
    ~]# systemctl stop NetworkManager 
    ~]# systemctl disable NetworkManager
    
- Disable SELinux using below command
::

    ~]# setenforce 0 ; sed -i 's/=enforcing/=disabled/g' /etc/sysconfig/selinux
    
3. Controller node
====================
3.1 Config NTP
--------------
::

    timedatectl set-timezone Asia/Ho_Chi_Minh
    yum -y install chrony
    vi /etc/chrony.conf
    line 3 and 24

    systemctl enable --now chronyd
    systemctl restart chronyd
    chronyc sources


3.2 Install MariaDB to configure Database Server:
---------------
::

   [root@controllernode ~]# dnf module -y install mariadb:10.3
   [root@controllernode ~]# vi /etc/my.cnf.d/charaset.cnf
  
   
      # create new
      # set default charaset
      # if not set, default is [latin1]
      # for the case of 4 bytes UTF-8, specify [utf8mb4]
      [mysqld]
      character-set-server = utf8mb4

      [client]
      default-character-set = utf8mb4
::

  [root@controllernode ~]# systemctl enable --now mariadb

3.3 Initial Settings for MariaDB
----------------
::

    #mysql_secure_installation
    
3.4 	Initial Settings for MariaDB.
----------------
::

      [root@controllernode ~]# mysql_secure_installation
      Enter current password for root (enter for none): 
      OK, successfully used password, moving on...

      Setting the root password ensures that nobody can log into the MariaDB
      root user without the proper authorisation.

      Set root password? [Y/n] y
      New password: 
      Re-enter new password: 
      Password updated successfully!
      Reloading privilege tables..
       ... Success!


      By default, a MariaDB installation has an anonymous user, allowing anyone
      to log into MariaDB without having to have a user account created for
      them.  This is intended only for testing, and to make the installation
      go a bit smoother.  You should remove them before moving into a
      production environment.

      Remove anonymous users? [Y/n] y
       ... Success!

      Normally, root should only be allowed to connect from 'localhost'.  This
      ensures that someone cannot guess at the root password from the network.

      Disallow root login remotely? [Y/n] y
       ... Success!

      By default, MariaDB comes with a database named 'test' that anyone can
      access.  This is also intended only for testing, and should be removed
      before moving into a production environment.

      Remove test database and access to it? [Y/n] y
       - Dropping test database...
       ... Success!
       - Removing privileges on test database...
       ... Success!

      Reloading the privilege tables will ensure that all changes made so far
      will take effect immediately.

      Reload privilege tables now? [Y/n] y
       ... Success!

      Cleaning up...

      All done!  If you've completed all of the above steps, your MariaDB
      installation should now be secure.

      Thanks for using MariaDB!

- Connect to MariaDB with root
::

    [root@controllernode ~]# mysql -u root -p
    Enter password: 
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 18
    Server version: 10.3.17-MariaDB MariaDB Server

    Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    MariaDB [(none)]>
    
3.5 Add Repository of Openstack Ussuri and also Upgrade CentOS System
------------
- Especially, it needs to upgrade some Python3 packages from Openstack Ussuri repository.
::

     [root@controllernode ~]# dnf -y install centos-release-openstack-ussuri
     [root@controllernode ~]# sed -i -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/CentOS-OpenStack-ussuri.repo
     [root@controllernode ~]# dnf --enablerepo=centos-openstack-ussuri -y upgrade

3.6 	Install RabbitMQ, Memcached.
---------------
- enable PowerTools
::

    [root@controllernode ~]# dnf --enablerepo=PowerTools -y install rabbitmq-server memcached
    [root@controllernode ~]# vi /etc/my.cnf.d/mariadb-server.cnf
      # add into [mysqld] section
      [mysqld]
      .....
      .....
      # default value 151 is not enough on Openstack Env
      max_connections=500
      
    [root@controllernode ~]# vi /etc/sysconfig/memcached
      # line 5: change (listen all)
      OPTIONS="-l 0.0.0.0,::"
    
    [root@controllernode ~]# systemctl restart mariadb rabbitmq-server memcached
    [root@controllernode ~]# systemctl enable mariadb rabbitmq-server memcached
    
- add openstack user
- set any password you like for [password]

::

    [root@controllernode ~]# rabbitmqctl add_user openstack password
    Adding user "openstack"
    [root@controllernode ~]# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
    Setting permissions for user "openstack" in vhost "/" ...


3.7 Configure Keystone
------------
Install and Configure OpenStack Identity Service (Keystone)
- Add a User and Database on MariaDB for Keystone

::

      [root@controllernode ~]# mysql -u root -p
      Enter password: 
      Welcome to the MariaDB monitor.  Commands end with ; or \g.
      Your MariaDB connection id is 8
      Server version: 10.3.17-MariaDB MariaDB Server

      Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

      Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

      MariaDB [(none)]> create database keystone;
      Query OK, 1 row affected (0.001 sec)

      MariaDB [(none)]> grant all privileges on keystone.* to keystone@'localhost' identified by 'password'; 
      Query OK, 0 rows affected (0.003 sec)

      MariaDB [(none)]> grant all privileges on keystone.* to keystone@'%' identified by 'password'; 
      Query OK, 0 rows affected (0.000 sec)

      MariaDB [(none)]> flush privileges;
      Query OK, 0 rows affected (0.001 sec)

      MariaDB [(none)]> exit
      Bye

- Install Keystone
 + install from Ussuri, EPEL, PowerTools
 
::
    
    [root@controllernode ~]# dnf --enablerepo=centos-openstack-ussuri,PowerTools -y install openstack-keystone python3-openstackclient httpd mod_ssl python3-mod_wsgi python3-oauth2client
 
- Config Keystone

::
      
      
      [root@controllernode ~]# vi /etc/keystone/keystone.conf
      # line 430: add the line to specify Memcache server
      memcache_servers = 192.168.100.12:11211
      # line 574: add the line to specify MariaDB connection info
      connection = mysql+pymysql://keystone:password@192.168.100.12/keystone
      # line 2446: uncomment
      provider = fernet
      
      [root@controllernode ~]# su -s /bin/bash keystone -c "keystone-manage db_sync"
  + initialize keys
  ::
  
      [root@controllernode ~]# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
      [root@controllernode ~]# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
  
  + define Keystone Host
  ::
  
      [root@controllernode ~]# export controller=192.168.100.12
  + bootstrap keystone
  +replace any password you like for [adminpassword] section
  ::
  
        [root@controllernode ~]# keystone-manage bootstrap --bootstrap-password adminpassword \
        > --bootstrap-admin-url http://$controller:5000/v3/ \
        > --bootstrap-internal-url http://$controller:5000/v3/ \
        > --bootstrap-public-url http://$controller:5000/v3/ \
        > --bootstrap-region-id RegionOne
- 	Enable settings for Keystone and start Apache httpd
::

      [root@controllernode ~]# vi /etc/httpd/conf/httpd.conf
      # line 98: uncomment and change to your server name
      ServerName ServerName controllernode.test.local:80
      
      [root@controllernode ~]# ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
      [root@controllernode ~]# systemctl enable --now httpd
      Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
      
3.7.1 Add Projects on Keystone
* Create and Load environment variables file
  The password for [OS_PASSWORD] is the one you set it on bootstrapping keystone.
  The URL for [OS_AUTH_URL] is the Keystone server's hostname or IP address.
  ::
  
    [root@controllernode ~]#  vi ~/keystonerc
    
    export OS_PROJECT_DOMAIN_NAME=default
    export OS_USER_DOMAIN_NAME=default
    export OS_PROJECT_NAME=admin
    export OS_USERNAME=admin
    export OS_PASSWORD=adminpassword
    export OS_AUTH_URL=http://192.168.100.12:5000/v3
    export OS_IDENTITY_API_VERSION=3
    export OS_IMAGE_API_VERSION=2
    export PS1='[\u@\h \W(keystone)]\$ '
    
    
    
    
    
    
    
    [root@controllernode ~]# chmod 600 ~/keystonerc
    [root@controllernode ~]# source ~/keystonerc
    [root@controllernode ~(keystone)]# echo "source ~/keystonerc " >> ~/.bash_profile
    
* create [service] project
::

    [root@controllernode ~(keystone)]# openstack project create --domain default --description "Service Project" service
    +-------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    | description | Service Project                  |
    | domain_id   | default                          |
    | enabled     | True                             |
    | id          | 0834ee0e0aab41faaea4652d5880fa90 |
    | is_domain   | False                            |
    | name        | service                          |
    | options     | {}                               |
    | parent_id   | default                          |
    | tags        | []                               |
    +-------------+----------------------------------+
    
* confirm settings
::

      [root@controllernode ~(keystone)]# openstack project list
      +----------------------------------+---------+
      | ID                               | Name    |
      +----------------------------------+---------+
      | 0834ee0e0aab41faaea4652d5880fa90 | service |
      | 0ac7f9ef37e340cc9aaeba4ef1d3d15e | admin   |
      +----------------------------------+---------+
3.9. Configure Glance - Install and Configure OpenStack Image Service (Glance)
-------------------
- Add users and others for Glance in Keystone
   * create [glance] user in [service] project::
   
      [root@controllernode ~(keystone)]# openstack user create --domain default --project service --password servicepassword glance
      +---------------------+----------------------------------+
      | Field               | Value                            |
      +---------------------+----------------------------------+
      | default_project_id  | 0834ee0e0aab41faaea4652d5880fa90 |
      | domain_id           | default                          |
      | enabled             | True                             |
      | id                  | 4354f39a6fec461f9659a1dd5dc124e6 |
      | name                | glance                           |
      | options             | {}                               |
      | password_expires_at | None                             |
      +---------------------+----------------------------------+
  * add [glance] user in [admin] role::
      
      [root@controllernode ~(keystone)]# openstack role add --project service --user glance admin
      
  * create service entry for [glance]::
      
      [root@controllernode ~(keystone)]# openstack service create --name glance --description "OpenStack Image service" image
      +-------------+----------------------------------+
      | Field       | Value                            |
      +-------------+----------------------------------+
      | description | OpenStack Image service          |
      | enabled     | True                             |
      | id          | a82f9d6328db46a284a2df2c42cbbb52 |
      | name        | glance                           |
      | type        | image                            |
      +-------------+----------------------------------+
      
   * define Glance API Host::
   
      [root@controllernode ~(keystone)]# export controller=192.168.100.12
      
   * create endpoint for [glance] (public)::
      
      [root@controllernode ~(keystone)]# openstack endpoint create --region RegionOne image public http://$controller:9292
      +--------------+----------------------------------+
      | Field        | Value                            |
      +--------------+----------------------------------+
      | enabled      | True                             |
      | id           | e7cbcc9df4b04480a57ae8eb311906a9 |
      | interface    | public                           |
      | region       | RegionOne                        |
      | region_id    | RegionOne                        |
      | service_id   | a82f9d6328db46a284a2df2c42cbbb52 |
      | service_name | glance                           |
      | service_type | image                            |
      | url          | http://192.168.100.12:9292       |
      +--------------+----------------------------------+
      
   * create endpoint for [glance] (internal)::
   
      [root@controllernode ~(keystone)]# openstack endpoint create --region RegionOne image internal http://$controller:9292
      +--------------+----------------------------------+
      | Field        | Value                            |
      +--------------+----------------------------------+
      | enabled      | True                             |
      | id           | bc91960d561340249af12f06b081cf0d |
      | interface    | internal                         |
      | region       | RegionOne                        |
      | region_id    | RegionOne                        |
      | service_id   | a82f9d6328db46a284a2df2c42cbbb52 |
      | service_name | glance                           |
      | service_type | image                            |
      | url          | http://192.168.100.12:9292       |
      +--------------+----------------------------------+
      
   * create endpoint for [glance] (admin)::
      
      [root@controllernode ~(keystone)]# openstack endpoint create --region RegionOne image admin http://$controller:9292
      +--------------+----------------------------------+
      | Field        | Value                            |
      +--------------+----------------------------------+
      | enabled      | True                             |
      | id           | 267a040c6592460aaff7bb49fea9e3a3 |
      | interface    | admin                            |
      | region       | RegionOne                        |
      | region_id    | RegionOne                        |
      | service_id   | a82f9d6328db46a284a2df2c42cbbb52 |
      | service_name | glance                           |
      | service_type | image                            |
      | url          | http://192.168.100.12:9292       |
      +--------------+----------------------------------+
      
- Add a User and Database on MariaDB for Glance.
::

      [root@controllernode ~(keystone)]# mysql -u root -p
      Enter password: 
      Welcome to the MariaDB monitor.  Commands end with ; or \g.
      Your MariaDB connection id is 16
      Server version: 10.3.17-MariaDB MariaDB Server

      Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

      Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

      MariaDB [(none)]> create database glance; 
      Query OK, 1 row affected (0.007 sec)

      MariaDB [(none)]> grant all privileges on glance.* to glance@'localhost' identified by 'password';
      Query OK, 0 rows affected (0.016 sec)

      MariaDB [(none)]> grant all privileges on glance.* to glance@'%' identified by 'password';
      Query OK, 0 rows affected (0.000 sec)

      MariaDB [(none)]> flush privileges; 
      Query OK, 0 rows affected (0.008 sec)

      MariaDB [(none)]> exit
      Bye
- Install Glance.
  * install from Ussuri, EPEL, PowerTools::
  
      [root@controllernode ~(keystone)]# dnf --enablerepo=centos-openstack-ussuri,PowerTools,epel -y install openstack-glance
      
- Configure Glance
 ::
      
      [root@controllernode ~(keystone)]# mv /etc/glance/glance-api.conf /etc/glance/glance-api.conf.org
      [root@controllernode ~(keystone)]# vi /etc/glance/glance-api.conf
      # create new
      [DEFAULT]
      bind_host = 0.0.0.0

      [glance_store]
      stores = file,http
      default_store = file
      filesystem_store_datadir = /var/lib/glance/images/

      [database]
      # MariaDB connection info
      connection = mysql+pymysql://glance:password@192.168.100.12/glance

      # keystone auth info
      [keystone_authtoken]
      www_authenticate_uri = http://192.168.100.12:5000
      auth_url = http://192.168.100.12:5000
      memcached_servers = 192.168.100.12:11211
      auth_type = password
      project_domain_name = default
      user_domain_name = default
      project_name = service
      username = glance
      password = servicepassword

      [paste_deploy]
      flavor = keystone


      [root@controllernode ~(keystone)]# chmod 640 /etc/glance/glance-api.conf
      [root@controllernode ~(keystone)]# chown root:glance /etc/glance/glance-api.conf
      [root@controllernode ~(keystone)]# su -s /bin/bash glance -c "glance-manage db_sync"

      [root@controllernode ~(keystone)]# systemctl enable --now openstack-glance-api
      Created symlink /etc/systemd/system/multi-user.target.wants/openstack-glance-api.service → /usr/lib/systemd/system/openstack-glance-api.service.


3.10. Add VM Images


* Create these databases::

   mysql -u root -p
   
   #Keystone
   CREATE DATABASE keystone;
   GRANT ALL ON keystone.* TO 'keystoneUser'@'%' IDENTIFIED BY 'keystonePass';
   
   #Glance
   CREATE DATABASE glance;
   GRANT ALL ON glance.* TO 'glanceUser'@'%' IDENTIFIED BY 'glancePass';

   #Quantum
   CREATE DATABASE quantum;
   GRANT ALL ON quantum.* TO 'quantumUser'@'%' IDENTIFIED BY 'quantumPass';

   #Nova
   CREATE DATABASE nova;
   GRANT ALL ON nova.* TO 'novaUser'@'%' IDENTIFIED BY 'novaPass';      

   #Cinder
   CREATE DATABASE cinder;
   GRANT ALL ON cinder.* TO 'cinderUser'@'%' IDENTIFIED BY 'cinderPass';

   quit;
 
2.5. Others
-------------------

* Install other services::

   apt-get install -y vlan bridge-utils

* Enable IP_Forwarding::

   sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf

   # To save you from rebooting, perform the following
   sysctl net.ipv4.ip_forward=1

2.6. Keystone
-------------------

* Start by the keystone packages::

   apt-get install -y keystone

* Adapt the connection attribute in the /etc/keystone/keystone.conf to the new database::

   connection = mysql://keystoneUser:keystonePass@10.10.10.51/keystone

* Restart the identity service then synchronize the database::

   service keystone restart
   keystone-manage db_sync

* Fill up the keystone database using the two scripts available in the `Scripts folder <https://github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/tree/OVS_MultiNode/KeystoneScripts>`_ of this git repository::

   #Modify the **HOST_IP** and **EXT_HOST_IP** variables before executing the scripts
   
   wget https://raw.github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/OVS_MultiNode/KeystoneScripts/keystone_basic.sh
   wget https://raw.github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/OVS_MultiNode/KeystoneScripts/keystone_endpoints_basic.sh

   chmod +x keystone_basic.sh
   chmod +x keystone_endpoints_basic.sh

   ./keystone_basic.sh
   ./keystone_endpoints_basic.sh

* Create a simple credential file and load it so you won't be bothered later::

   nano creds

   #Paste the following:
   export OS_TENANT_NAME=admin
   export OS_USERNAME=admin
   export OS_PASSWORD=admin_pass
   export OS_AUTH_URL="http://192.168.100.51:5000/v2.0/"

   # Load it:
   source creds

* To test Keystone, we use a simple CLI command::

   keystone user-list

2.7. Glance
-------------------

* We Move now to Glance installation::

   apt-get install -y glance

* Update /etc/glance/glance-api-paste.ini with::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   delay_auth_decision = true
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = glance
   admin_password = service_pass

* Update the /etc/glance/glance-registry-paste.ini with::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = glance
   admin_password = service_pass

* Update /etc/glance/glance-api.conf with::

   sql_connection = mysql://glanceUser:glancePass@10.10.10.51/glance

* And::

   [paste_deploy]
   flavor = keystone
   
* Update the /etc/glance/glance-registry.conf with::

   sql_connection = mysql://glanceUser:glancePass@10.10.10.51/glance

* And::

   [paste_deploy]
   flavor = keystone

* Restart the glance-api and glance-registry services::

   service glance-api restart; service glance-registry restart

* Synchronize the glance database::

   glance-manage db_sync

* To test Glance, upload the cirros cloud image directly from the internet::

   glance image-create --name myFirstImage --is-public true --container-format bare --disk-format qcow2 --location http://download.cirros-cloud.net/0.3.1/cirros-0.3.1-x86_64-disk.img

* Now list the image to see what you have just uploaded::

   glance image-list

2.8. Quantum
-------------------

* Install the Quantum server and the OpenVSwitch package collection::

   apt-get install -y quantum-server

* Edit the OVS plugin configuration file /etc/quantum/plugins/openvswitch/ovs_quantum_plugin.ini with:: 

   #Under the database section
   [DATABASE]
   sql_connection = mysql://quantumUser:quantumPass@10.10.10.51/quantum

   #Under the OVS section
   [OVS]
   tenant_network_type = gre
   tunnel_id_ranges = 1:1000
   enable_tunneling = True

   #Firewall driver for realizing quantum security group function
   [SECURITYGROUP]
   firewall_driver = quantum.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

* Edit /etc/quantum/api-paste.ini ::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass

* Update the /etc/quantum/quantum.conf::

   [keystone_authtoken]
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass
   signing_dir = /var/lib/quantum/keystone-signing

* Restart the quantum server::

   service quantum-server restart

2.9. Nova
------------------

* Start by installing nova components::

   apt-get install -y nova-api nova-cert novnc nova-consoleauth nova-scheduler nova-novncproxy nova-doc nova-conductor

* Now modify authtoken section in the /etc/nova/api-paste.ini file to this::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = nova
   admin_password = service_pass
   signing_dirname = /tmp/keystone-signing-nova
   # Workaround for https://bugs.launchpad.net/nova/+bug/1154809
   auth_version = v2.0

* Modify the /etc/nova/nova.conf like this::

   [DEFAULT] 
   logdir=/var/log/nova
   state_path=/var/lib/nova
   lock_path=/run/lock/nova
   verbose=True
   api_paste_config=/etc/nova/api-paste.ini
   compute_scheduler_driver=nova.scheduler.simple.SimpleScheduler
   rabbit_host=10.10.10.51
   nova_url=http://10.10.10.51:8774/v1.1/
   sql_connection=mysql://novaUser:novaPass@10.10.10.51/nova
   root_helper=sudo nova-rootwrap /etc/nova/rootwrap.conf

   # Auth
   use_deprecated_auth=false
   auth_strategy=keystone

   # Imaging service
   glance_api_servers=10.10.10.51:9292
   image_service=nova.image.glance.GlanceImageService

   # Vnc configuration
   novnc_enabled=true
   novncproxy_base_url=http://192.168.100.51:6080/vnc_auto.html
   novncproxy_port=6080
   vncserver_proxyclient_address=10.10.10.51
   vncserver_listen=0.0.0.0

   # Network settings
   network_api_class=nova.network.quantumv2.api.API
   quantum_url=http://10.10.10.51:9696
   quantum_auth_strategy=keystone
   quantum_admin_tenant_name=service
   quantum_admin_username=quantum
   quantum_admin_password=service_pass
   quantum_admin_auth_url=http://10.10.10.51:35357/v2.0
   libvirt_vif_driver=nova.virt.libvirt.vif.LibvirtHybridOVSBridgeDriver
   linuxnet_interface_driver=nova.network.linux_net.LinuxOVSInterfaceDriver
   #If you want Quantum + Nova Security groups
   firewall_driver=nova.virt.firewall.NoopFirewallDriver
   security_group_api=quantum
   #If you want Nova Security groups only, comment the two lines above and uncomment line -1-.
   #-1-firewall_driver=nova.virt.libvirt.firewall.IptablesFirewallDriver

   #Metadata
   service_quantum_metadata_proxy = True
   quantum_metadata_proxy_shared_secret = helloOpenStack

   # Compute #
   compute_driver=libvirt.LibvirtDriver

   # Cinder #
   volume_api_class=nova.volume.cinder.API
   osapi_volume_listen_port=5900

* Synchronize your database::

   nova-manage db sync

* Restart nova-* services::

   cd /etc/init.d/; for i in $( ls nova-* ); do sudo service $i restart; done   

* Check for the smiling faces on nova-* services to confirm your installation::

   nova-manage service list

2.10. Cinder
--------------

* Install the required packages::

   apt-get install -y cinder-api cinder-scheduler cinder-volume iscsitarget open-iscsi iscsitarget-dkms

* Configure the iscsi services::

   sed -i 's/false/true/g' /etc/default/iscsitarget

* Restart the services::
   
   service iscsitarget start
   service open-iscsi start

* Configure /etc/cinder/api-paste.ini like the following::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   service_protocol = http
   service_host = 192.168.100.51
   service_port = 5000
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = cinder
   admin_password = service_pass
   signing_dir = /var/lib/cinder

* Edit the /etc/cinder/cinder.conf to::

   [DEFAULT]
   rootwrap_config=/etc/cinder/rootwrap.conf
   sql_connection = mysql://cinderUser:cinderPass@10.10.10.51/cinder
   api_paste_config = /etc/cinder/api-paste.ini
   iscsi_helper=ietadm
   volume_name_template = volume-%s
   volume_group = cinder-volumes
   verbose = True
   auth_strategy = keystone
   iscsi_ip_address=10.10.10.51

* Then, synchronize your database::

   cinder-manage db sync

* Finally, don't forget to create a volumegroup and name it cinder-volumes::

   dd if=/dev/zero of=cinder-volumes bs=1 count=0 seek=2G
   losetup /dev/loop2 cinder-volumes
   fdisk /dev/loop2
   #Type in the followings:
   n
   p
   1
   ENTER
   ENTER
   t
   8e
   w

* Proceed to create the physical volume then the volume group::

   pvcreate /dev/loop2
   vgcreate cinder-volumes /dev/loop2

**Note:** Beware that this volume group gets lost after a system reboot. (Click `Here <https://github.com/mseknibilel/OpenStack-Folsom-Install-guide/blob/master/Tricks%26Ideas/load_volume_group_after_system_reboot.rst>`_ to know how to load it after a reboot) 

* Restart the cinder services::

   cd /etc/init.d/; for i in $( ls cinder-* ); do sudo service $i restart; done

* Verify if cinder services are running::

   cd /etc/init.d/; for i in $( ls cinder-* ); do sudo service $i status; done

2.11. Horizon
--------------

* To install horizon, proceed like this ::

   apt-get install -y openstack-dashboard memcached

* If you don't like the OpenStack ubuntu theme, you can remove the package to disable it::

   dpkg --purge openstack-dashboard-ubuntu-theme 

* Reload Apache and memcached::

   service apache2 restart; service memcached restart

* Check OpenStack Dashboard at http://192.168.100.51/horizon. We can login with the admin / admin_pass


3. Network Node
================

3.1. Preparing the Node
------------------

* After you install Ubuntu 12.04 or 13.04 Server 64bits, Go in sudo mode::

   sudo su

* Add Grizzly repositories [Only for Ubuntu 12.04]::

   apt-get install -y ubuntu-cloud-keyring 
   echo deb http://ubuntu-cloud.archive.canonical.com/ubuntu precise-updates/grizzly main >> /etc/apt/sources.list.d/grizzly.list

* Update your system::

   apt-get update -y
   apt-get upgrade -y
   apt-get dist-upgrade -y

* Install ntp service::

   apt-get install -y ntp

* Configure the NTP server to follow the controller node::
   
   #Comment the ubuntu NTP servers
   sed -i 's/server 0.ubuntu.pool.ntp.org/#server 0.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   sed -i 's/server 1.ubuntu.pool.ntp.org/#server 1.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   sed -i 's/server 2.ubuntu.pool.ntp.org/#server 2.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   sed -i 's/server 3.ubuntu.pool.ntp.org/#server 3.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   
   #Set the network node to follow up your conroller node
   sed -i 's/server ntp.ubuntu.com/server 10.10.10.51/g' /etc/ntp.conf

   service ntp restart  

* Install other services::

   apt-get install -y vlan bridge-utils

* Enable IP_Forwarding::

   sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
   
   # To save you from rebooting, perform the following
   sysctl net.ipv4.ip_forward=1

3.2.Networking
------------

* 3 NICs must be present::
   
   # OpenStack management
   auto eth0
   iface eth0 inet static
   address 10.10.10.52
   netmask 255.255.255.0

   # VM Configuration
   auto eth1
   iface eth1 inet static
   address 10.20.20.52
   netmask 255.255.255.0

   # VM internet Access
   auto eth2
   iface eth2 inet static
   address 192.168.100.52
   netmask 255.255.255.0

3.4. OpenVSwitch (Part1)
------------------

* Install the openVSwitch::

   apt-get install -y openvswitch-switch openvswitch-datapath-dkms

* Create the bridges::

   #br-int will be used for VM integration	
   ovs-vsctl add-br br-int

   #br-ex is used to make to VM accessible from the internet
   ovs-vsctl add-br br-ex

3.5. Quantum
------------------

* Install the Quantum openvswitch agent, l3 agent and dhcp agent::

   apt-get -y install quantum-plugin-openvswitch-agent quantum-dhcp-agent quantum-l3-agent quantum-metadata-agent

* Edit /etc/quantum/api-paste.ini::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass

* Edit the OVS plugin configuration file /etc/quantum/plugins/openvswitch/ovs_quantum_plugin.ini with:: 

   #Under the database section
   [DATABASE]
   sql_connection = mysql://quantumUser:quantumPass@10.10.10.51/quantum

   #Under the OVS section
   [OVS]
   tenant_network_type = gre
   tunnel_id_ranges = 1:1000
   integration_bridge = br-int
   tunnel_bridge = br-tun
   local_ip = 10.20.20.52
   enable_tunneling = True

   #Firewall driver for realizing quantum security group function
   [SECURITYGROUP]
   firewall_driver = quantum.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

* Update /etc/quantum/metadata_agent.ini::
   
   # The Quantum user information for accessing the Quantum API.
   auth_url = http://10.10.10.51:35357/v2.0
   auth_region = RegionOne
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass

   # IP address used by Nova metadata server
   nova_metadata_ip = 10.10.10.51

   # TCP Port used by Nova metadata server
   nova_metadata_port = 8775

   metadata_proxy_shared_secret = helloOpenStack

* Make sure that your rabbitMQ IP in /etc/quantum/quantum.conf is set to the controller node::

   rabbit_host = 10.10.10.51

   #And update the keystone_authtoken section

   [keystone_authtoken]
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass
   signing_dir = /var/lib/quantum/keystone-signing

* Edit /etc/sudoers.d/quantum_sudoers to give it full access like this (This is unfortunatly mandatory) ::

   nano /etc/sudoers.d/quantum_sudoers
   
   #Modify the quantum user
   quantum ALL=NOPASSWD: ALL

* Restart all the services::

   cd /etc/init.d/; for i in $( ls quantum-* ); do sudo service $i restart; done

3.4. OpenVSwitch (Part2)
------------------
* Edit the eth2 in /etc/network/interfaces to become like this::

   # VM internet Access
   auto eth2
   iface eth2 inet manual
   up ifconfig $IFACE 0.0.0.0 up
   up ip link set $IFACE promisc on
   down ip link set $IFACE promisc off
   down ifconfig $IFACE down

* Add the eth2 to the br-ex::

   #Internet connectivity will be lost after this step but this won't affect OpenStack's work
   ovs-vsctl add-port br-ex eth2

   #If you want to get internet connection back, you can assign the eth2's IP address to the br-ex in the /etc/network/interfaces file.

4. Compute Node
=========================

4.1. Preparing the Node
------------------

* After you install Ubuntu 12.04 or 13.04 Server 64bits, Go in sudo mode::

   sudo su

* Add Grizzly repositories [Only for Ubuntu 12.04]::

   apt-get install -y ubuntu-cloud-keyring 
   echo deb http://ubuntu-cloud.archive.canonical.com/ubuntu precise-updates/grizzly main >> /etc/apt/sources.list.d/grizzly.list


* Update your system::

   apt-get update -y
   apt-get upgrade -y
   apt-get dist-upgrade -y

* Reboot (you might have new kernel)

* Install ntp service::

   apt-get install -y ntp

* Configure the NTP server to follow the controller node::
   
   #Comment the ubuntu NTP servers
   sed -i 's/server 0.ubuntu.pool.ntp.org/#server 0.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   sed -i 's/server 1.ubuntu.pool.ntp.org/#server 1.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   sed -i 's/server 2.ubuntu.pool.ntp.org/#server 2.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   sed -i 's/server 3.ubuntu.pool.ntp.org/#server 3.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   
   #Set the compute node to follow up your conroller node
   sed -i 's/server ntp.ubuntu.com/server 10.10.10.51/g' /etc/ntp.conf

   service ntp restart  

* Install other services::

   apt-get install -y vlan bridge-utils

* Enable IP_Forwarding::

   sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
   
   # To save you from rebooting, perform the following
   sysctl net.ipv4.ip_forward=1

4.2.Networking
------------

* Perform the following::
   
   # OpenStack management
   auto eth0
   iface eth0 inet static
   address 10.10.10.53
   netmask 255.255.255.0

   # VM Configuration
   auto eth1
   iface eth1 inet static
   address 10.20.20.53
   netmask 255.255.255.0

4.3 KVM
------------------

* make sure that your hardware enables virtualization::

   apt-get install -y cpu-checker
   kvm-ok

* Normally you would get a good response. Now, move to install kvm and configure it::

   apt-get install -y kvm libvirt-bin pm-utils

* Edit the cgroup_device_acl array in the /etc/libvirt/qemu.conf file to::

   cgroup_device_acl = [
   "/dev/null", "/dev/full", "/dev/zero",
   "/dev/random", "/dev/urandom",
   "/dev/ptmx", "/dev/kvm", "/dev/kqemu",
   "/dev/rtc", "/dev/hpet","/dev/net/tun"
   ]

* Delete default virtual bridge ::

   virsh net-destroy default
   virsh net-undefine default

* Enable live migration by updating /etc/libvirt/libvirtd.conf file::

   listen_tls = 0
   listen_tcp = 1
   auth_tcp = "none"

* Edit libvirtd_opts variable in /etc/init/libvirt-bin.conf file::

   env libvirtd_opts="-d -l"

* Edit /etc/default/libvirt-bin file ::

   libvirtd_opts="-d -l"

* Restart the libvirt service and dbus to load the new values::

    service dbus restart && service libvirt-bin restart

4.4. OpenVSwitch
------------------

* Install the openVSwitch::

   apt-get install -y openvswitch-switch openvswitch-datapath-dkms

* Create the bridges::

   #br-int will be used for VM integration	
   ovs-vsctl add-br br-int

4.5. Quantum
------------------

* Install the Quantum openvswitch agent::

   apt-get -y install quantum-plugin-openvswitch-agent

* Edit the OVS plugin configuration file /etc/quantum/plugins/openvswitch/ovs_quantum_plugin.ini with:: 

   #Under the database section
   [DATABASE]
   sql_connection = mysql://quantumUser:quantumPass@10.10.10.51/quantum

   #Under the OVS section
   [OVS]
   tenant_network_type = gre
   tunnel_id_ranges = 1:1000
   integration_bridge = br-int
   tunnel_bridge = br-tun
   local_ip = 10.20.20.53
   enable_tunneling = True
   
   #Firewall driver for realizing quantum security group function
   [SECURITYGROUP]
   firewall_driver = quantum.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

* Make sure that your rabbitMQ IP in /etc/quantum/quantum.conf is set to the controller node::
   
   rabbit_host = 10.10.10.51

   #And update the keystone_authtoken section

   [keystone_authtoken]
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass
   signing_dir = /var/lib/quantum/keystone-signing

* Restart all the services::

   service quantum-plugin-openvswitch-agent restart

4.6. Nova
------------------

* Install nova's required components for the compute node::

   apt-get install -y nova-compute-kvm

* Now modify authtoken section in the /etc/nova/api-paste.ini file to this::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = nova
   admin_password = service_pass
   signing_dirname = /tmp/keystone-signing-nova
   # Workaround for https://bugs.launchpad.net/nova/+bug/1154809
   auth_version = v2.0

* Edit /etc/nova/nova-compute.conf file ::
   
   [DEFAULT]
   libvirt_type=kvm
   libvirt_ovs_bridge=br-int
   libvirt_vif_type=ethernet
   libvirt_vif_driver=nova.virt.libvirt.vif.LibvirtHybridOVSBridgeDriver
   libvirt_use_virtio_for_bridges=True

* Modify the /etc/nova/nova.conf like this::

   [DEFAULT] 
   logdir=/var/log/nova
   state_path=/var/lib/nova
   lock_path=/run/lock/nova
   verbose=True
   api_paste_config=/etc/nova/api-paste.ini
   compute_scheduler_driver=nova.scheduler.simple.SimpleScheduler
   rabbit_host=10.10.10.51
   nova_url=http://10.10.10.51:8774/v1.1/
   sql_connection=mysql://novaUser:novaPass@10.10.10.51/nova
   root_helper=sudo nova-rootwrap /etc/nova/rootwrap.conf

   # Auth
   use_deprecated_auth=false
   auth_strategy=keystone

   # Imaging service
   glance_api_servers=10.10.10.51:9292
   image_service=nova.image.glance.GlanceImageService

   # Vnc configuration
   novnc_enabled=true
   novncproxy_base_url=http://192.168.100.51:6080/vnc_auto.html
   novncproxy_port=6080
   vncserver_proxyclient_address=10.10.10.53
   vncserver_listen=0.0.0.0

   # Network settings
   network_api_class=nova.network.quantumv2.api.API
   quantum_url=http://10.10.10.51:9696
   quantum_auth_strategy=keystone
   quantum_admin_tenant_name=service
   quantum_admin_username=quantum
   quantum_admin_password=service_pass
   quantum_admin_auth_url=http://10.10.10.51:35357/v2.0
   libvirt_vif_driver=nova.virt.libvirt.vif.LibvirtHybridOVSBridgeDriver
   linuxnet_interface_driver=nova.network.linux_net.LinuxOVSInterfaceDriver
   #If you want Quantum + Nova Security groups
   firewall_driver=nova.virt.firewall.NoopFirewallDriver
   security_group_api=quantum
   #If you want Nova Security groups only, comment the two lines above and uncomment line -1-.
   #-1-firewall_driver=nova.virt.libvirt.firewall.IptablesFirewallDriver
   
   #Metadata
   service_quantum_metadata_proxy = True
   quantum_metadata_proxy_shared_secret = helloOpenStack

   # Compute #
   compute_driver=libvirt.LibvirtDriver

   # Cinder #
   volume_api_class=nova.volume.cinder.API
   osapi_volume_listen_port=5900
   cinder_catalog_info=volume:cinder:internalURL

* Restart nova-* services::

   cd /etc/init.d/; for i in $( ls nova-* ); do sudo service $i restart; done   

* Check for the smiling faces on nova-* services to confirm your installation::

   nova-manage service list


5. Your first VM
================

To start your first VM, we first need to create a new tenant, user and internal network.

* Create a new tenant ::

   keystone tenant-create --name project_one

* Create a new user and assign the member role to it in the new tenant (keystone role-list to get the appropriate id)::

   keystone user-create --name=user_one --pass=user_one --tenant-id $put_id_of_project_one --email=user_one@domain.com
   keystone user-role-add --tenant-id $put_id_of_project_one  --user-id $put_id_of_user_one --role-id $put_id_of_member_role

* Create a new network for the tenant::

   quantum net-create --tenant-id $put_id_of_project_one net_proj_one 

* Create a new subnet inside the new tenant network::

   quantum subnet-create --tenant-id $put_id_of_project_one net_proj_one 50.50.1.0/24 --dns_nameservers list=true 8.8.8.7 8.8.8.8

* Create a router for the new tenant::

   quantum router-create --tenant-id $put_id_of_project_one router_proj_one

* Add the router to the running l3 agent (if it wasn't automatically added)::

   quantum agent-list (to get the l3 agent ID)
   quantum l3-agent-router-add $l3_agent_ID router_proj_one

* Add the router to the subnet::

   quantum router-interface-add $put_router_proj_one_id_here $put_subnet_id_here

* Restart all quantum services::

   cd /etc/init.d/; for i in $( ls quantum-* ); do sudo service $i restart; done

* Create an external network with the tenant id belonging to the admin tenant (keystone tenant-list to get the appropriate id)::

   quantum net-create --tenant-id $put_id_of_admin_tenant ext_net --router:external=True

* Create a subnet for the floating ips::

   quantum subnet-create --tenant-id $put_id_of_admin_tenant --allocation-pool start=192.168.100.102,end=192.168.100.126 --gateway 192.168.100.1 ext_net 192.168.100.100/24 --enable_dhcp=False

* Set your router's gateway to the external network:: 

   quantum router-gateway-set $put_router_proj_one_id_here $put_id_of_ext_net_here

* Source creds relative to your project one tenant now::

   nano creds_proj_one

   #Paste the following:
   export OS_TENANT_NAME=project_one
   export OS_USERNAME=user_one
   export OS_PASSWORD=user_one
   export OS_AUTH_URL="http://192.168.100.51:5000/v2.0/"

   source creds_proj_one

* Add this security rules to make your VMs pingable::

   nova --no-cache secgroup-add-rule default icmp -1 -1 0.0.0.0/0
   nova --no-cache secgroup-add-rule default tcp 22 22 0.0.0.0/0

* Start by allocating a floating ip to the project one tenant::

   quantum floatingip-create ext_net

* Start a VM::

   nova --no-cache boot --image $id_myFirstImage --flavor 1 my_first_vm 

* pick the id of the port corresponding to your VM::

   quantum port-list

* Associate the floating IP to your VM::

   quantum floatingip-associate $put_id_floating_ip $put_id_vm_port

That's it ! ping your VM and enjoy your OpenStack.

6. Licensing
============

OpenStack Grizzly Install Guide is licensed under a Creative Commons Attribution 3.0 Unported License.

.. image:: http://i.imgur.com/4XWrp.png
To view a copy of this license, visit [ http://creativecommons.org/licenses/by/3.0/deed.en_US ].

7. Contacts
===========

Bilel Msekni  : msekni.bilel@gmail.com

8. Credits
=================

This work has been based on:

* Bilel Msekni's Folsom Install guide [https://github.com/mseknibilel/OpenStack-Folsom-Install-guide]
* OpenStack Grizzly Install Guide (Master Branch) [https://github.com/mseknibilel/OpenStack-Grizzly-Install-Guide]

9. To do
=======

Your suggestions are always welcomed.





