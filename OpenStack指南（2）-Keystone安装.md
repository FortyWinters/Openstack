# OpenStack安装指南（2）

## Keystone安装

> [Keystone Installation Tutorial](https://docs.openstack.org/keystone/queens/install/)



### controller

#### 数据库注册

`# mysql`

`MariaDB[(none)]> CREATE DATABASE keystone;`

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY '123';`

`MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY '123';`



#### 安装配置

`# apt install keystone apache2 libapache2-mod-wsgi`

`# vim /etc/keystone/keystone.conf`

```
[database]
connection = mysql+pymysql://keystone:123@controller/keystone

[token]
provider = fernet
```

`# su -s /bin/sh -c "keystone-manage db_sync" keystone`

`# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone`

`# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone `

`# keystone-manage bootstrap --bootstrap-password 123 --bootstrap-admin-url http://controller:5000/v3/ --bootstrap-internal-url http://controller:5000/v3/ --bootstrap-public-url http://controller:5000/v3/ --bootstrap-region-id RegionOne`

`# vim /etc/apache2/apache2.conf`

```
ServerName controller
# 此行需额外添加
```

`# service apache2 restart`

```
# export OS_USERNAME=admin
# export OS_PASSWORD=123
# export OS_PROJECT_NAME=admin
# export OS_USER_DOMAIN_NAME=Default
# export OS_PROJECT_DOMAIN_NAME=Default
# export OS_AUTH_URL=http://controller:5000/v3
# export OS_IDENTITY_API_VERSION=3
```



#### 创建域、项目、用户、角色

`# openstack domain create --description "An Example Domain" example`

`# openstack project create --domain default --description "Service Project" service`

`# openstack project create --domain default --description "Demo Project" demo`

`# openstack user create --domain default --password-prompt demo`

`# openstack role create user`

`# openstack role add --project demo --user demo user`



#### 登录凭证

`# unset OS_AUTH_URL OS_PASSWORD`

`# vim admin-openrc`

```
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=123
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

`# . admin-openrc`

