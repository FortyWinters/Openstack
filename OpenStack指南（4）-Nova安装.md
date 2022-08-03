# OpenStack安装指南（4）

## Nova安装

> [Nova Installation Tutorial](https://docs.openstack.org/nova/queens/install/)



### controller

#### 数据库注册

`# mysql`

`MariaDB[(none)]> CREATE DATABASE nova_api;`

`MariaDB[(none)]> CREATE DATABASE nova;`

`MariaDB[(none)]> CREATE DATABASE nova_cell0;`

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY '123';`

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY '123';`

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY '123';`

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY '123';`

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY '123';`

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY '123';`



#### 创建服务

`# . admin-openrc`

`# openstack user create --domain default --password-prompt nova`

`# openstack role add --project service --user nova admin`

`# openstack service create --name nova --description "OpenStack Compute" compute`

```
# openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
# openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
# openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
```

`# openstack user create --domain default --password-prompt placement`

`# openstack role add --project service --user placement admin`

`# openstack service create --name placement --description "Placement API" placement`

```
# openstack endpoint create --region RegionOne placement public http://controller:8778
# openstack endpoint create --region RegionOne placement internal http://controller:8778
# openstack endpoint create --region RegionOne placement admin http://controller:8778
```



#### 安装配置

`# apt install nova-api nova-conductor nova-consoleauth nova-novncproxy nova-scheduler nova-placement-api`

`# vim /etc/nova/nova.conf`

```
[DEFAULT]
# log_dir = /var/log/nova
transport_url = rabbit://openstack:123@controller
my_ip = 192.168.10.10
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[api_database]
connection = mysql+pymysql://nova:123@controller/nova_api

[database]
connection = mysql+pymysql://nova:123@controller/nova

[api]
auth_strategy = keystone

[keystone_authtoken]
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 123

[vnc]
enabled = true
server_listen = $my_ip
server_proxyclient_address = $my_ip

[glance]
api_servers = http://controller:9292

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[placement]
# os_region_name = openstack
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = 123
```

> 虚机拷贝时注意更改

`# su -s /bin/sh -c "nova-manage api_db sync" nova`

`# su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova`

`# su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova`

`# su -s /bin/sh -c "nova-manage db sync" nova`

```
# service nova-api restart
# service nova-consoleauth restart
# service nova-scheduler restart
# service nova-conductor restart
# service nova-novncproxy restart
```



### compute1

`# apt install nova-compute`

`# vim /etc/nova/nova.conf`

```
[DEFAULT]
# log_dir = /var/log/nova
transport_url = rabbit://openstack:123@controller
my_ip = 192.168.10.10
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[api]
auth_strategy = keystone

[keystone_authtoken]
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 123

[vnc]
enabled = True
server_listen = 0.0.0.0
server_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html

[glance]
api_servers = http://controller:9292

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[placement]
# os_region_name = openstack
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = 123
```

`# vim /etc/nova/nova-compute.conf`

```
[libvirt] 
virt_type  =  qemu
```

`# service nova-compute restart`



### controller

`# . admin-openrc`

`# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova`

`# openstack compute service list`

确认安装成功

