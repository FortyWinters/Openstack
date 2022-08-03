# OpenStack安装指南（5）

## Neutron安装

> [Neutron Installation Tutorial](https://docs.openstack.org/neutron/queens/install/)



### controller

#### 数据库注册

`# mysql`

`MariaDB[(none)]> CREATE DATABASE neutron;`

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY '123';`

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY '123';`

#### 创建服务

`# . admin-openrc`

`# openstack user create --domain default --password-prompt neutron`

`# openstack role add --project service --user neutron admin`

`# openstack service create --name neutron --description "OpenStack Networking" network`

```
# openstack endpoint create --region RegionOne network public http://controller:9696
# openstack endpoint create --region RegionOne network internal http://controller:9696
# openstack endpoint create --region RegionOne network admin http://controller:9696
```

#### 安装配置

`# apt install neutron-server neutron-plugin-ml2 neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent`

`# vim /etc/neutron/neutron.conf`

```
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:123@controller
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true

[database]
connection = mysql+pymysql://neutron:123@controller/neutron

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = 123

[nova]
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = 123

[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
```

`# vim /etc/neutron/plugins/ml2/ml2_conf.ini`

```
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security

[ml2_type_flat]
flat_networks = provider

[ml2_type_vxlan]
vni_ranges = 1:1000

[securitygroup]
enable_ipset = true
```

`# vim /etc/neutron/plugins/ml2/linuxbridge_agent.ini`

```
[linux_bridge]
physical_interface_mappings = provider:ens192

[vxlan]
enable_vxlan = true
local_ip = 192.168.10.10
l2_population = true

[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

> 拷贝虚拟机时需注意更改

`# vim /etc/sysctl.conf`

```
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
```

`# vim /etc/neutron/l3_agent.ini `

```
[DEFAULT]
interface_driver = linuxbridge
```

`# vim /etc/neutron/dhcp_agent.ini`

```
[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```

`# vim /etc/neutron/metadata_agent.ini`

```
[DEFAULT]
nova_metadata_host = controller
metadata_proxy_shared_secret = 123
```

`# vim /etc/nova/nova.conf`

```
[neutron]
url = http://controller:9696
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = 123
service_metadata_proxy = true
metadata_proxy_shared_secret = 123
```

`# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron`

```
# service nova-api restart
# service neutron-server restart
# service neutron-linuxbridge-agent restart
# service neutron-dhcp-agent restart
# service neutron-metadata-agent restart
# service neutron-l3-agent restart
```



### compute1

`# apt install neutron-linuxbridge-agent`

`# vim /etc/neutron/neutron.conf`

```
[DEFAULT]
transport_url = rabbit://openstack:123@controller
auth_strategy = keystone

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = 123

[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
```

`# vim /etc/neutron/plugins/ml2/linuxbridge_agent.ini`

```
[linux_bridge]
physical_interface_mappings = provider:ens192

[vxlan]
enable_vxlan = true
local_ip = 192.168.10.20
l2_population = true

[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

> 拷贝虚机时需注意更改

`# vim /etc/sysctl.conf`

```
net.bridge.bridge-nf-call-iptables =1
net.bridge.bridge-nf-call-ip6tables = 1
```

`# vim /etc/nova/nova.conf`

```
[neutron]
url = http://controller:9696
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = 123
```

```
# service nova-compute restart
# service neutron-linuxbridge-agent restart
```
