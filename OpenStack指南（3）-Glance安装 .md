# OpenStack安装指南（3）

## Glance安装

> [Glance Installation Tutorial](https://docs.openstack.org/glance/queens/install/)



### controller

#### 数据库注册

`# mysql`

`MariaDB[(none)]> CREATE DATABASE glance;`

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY '123';`

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY '123';`



#### 创建服务

`# . admin-openrc`

`# openstack user create --domain default --password-prompt glance`

`# openstack role add --project service --user glance admin`

`# openstack service create --name glance --description "OpenStack Image" image`

```
# openstack endpoint create --region RegionOne image public http://controller:9292
# openstack endpoint create --region RegionOne image internal http://controller:9292
# openstack endpoint create --region RegionOne image admin http://controller:9292
```



#### 安装配置

`# apt install glance`

`# vim /etc/glance/glance-api.conf`

```
[database]
connection = mysql+pymysql://glance:123@controller/glance

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = 123

[paste_deploy]
flavor = keystone

[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
```

`# vim /etc/glance/glance-registry.conf`

```
[database]
connection = mysql+pymysql://glance:123@controller/glance

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = 123

[paste_deploy]
flavor = keystone
```

`# su -s /bin/sh -c "glance-manage db_sync" glance`

`# service glance-api restart`

`# service glance-registry restart`



#### 上传镜像

`# . admin-openrc`

`# wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img`

`# openstack image create "cirros" --file cirros-0.4.0-x86_64-disk.img --disk-format qcow2 --container-format bare --public`
