# OpenStack安装指南（1）

## 环境安装

> [OpenStack Installation Guide](https://docs.openstack.org/install-guide/)



### 网络设置

#### esxi设置

所有虚拟交换机接受混杂模式和伪传输

 <img src="D:\BOX\openstack-pic\13.png" alt="13" style="zoom:80%;" />

| host       | ens160                               | ens192                                 |
| ---------- | ------------------------------------ | -------------------------------------- |
| controller | 192.168.10.10/24（openstack-manage） | 192.168.20.10/24（openstack-provider） |
| compute1   | 192.168.10.20/24（openstack-manage） | 192.168.20.20/24（openstack-provider） |
| gateway    | 10.112.185.107/16（BUPT-SCHOOL-NET） | 192.168.20.1/24（openstack-provider）  |

#### controller

` # vim /etc/network/interfaces`

```
auto lo
iface lo inet loopback

auto ens160
iface ens160 inet static
address 192.168.10.10
netmask 255.255.255.0

auto ens192
iface ens192 inet static
address 192.168.20.10
netmask 255.255.255.0
gateway 192.168.20.1
dns-nameservers 223.5.5.5
```

重启虚拟机

#### compute1

` # vim /etc/network/interfaces`

```auto lo
iface lo inet loopback

auto ens160
iface ens160 inet static
address 192.168.10.20
netmask 255.255.255.0

auto ens192
iface ens192 inet static
address 192.168.20.20
netmask 255.255.255.0
gateway 192.168.20.1
dns-nameservers 223.5.5.5
```

重启虚拟机

#### gateway

` # vim /etc/network/interfaces`

```auto lo
iface lo inet loopback

auto ens192
iface ens192 inet static
address 192.168.20.1
netmask 255.255.255.0
dns-nameservers 223.5.5.5
```

` # echo "1"> /proc/sys/net/ipv4/ip_forward`

` # vim /etc/sysctl.conf`

` net.ipv4.ip_forward = 1`

重启虚拟机

` # iptables -t nat -A POSTROUTING -o ens160 -s 192.168.20.0/24 -j SNAT --to 10.112.185.107`



### 配置名称解析

#### controller

`# vim /etc/hosts`

```
127.0.0.1       localhost
# 127.0.1.1      controller

192.168.10.10   controller
192.168.10.20   compute1
```

> 虚机拷贝时注意更改

#### compute1

`# vim /etc/hosts`

```
127.0.0.1       localhost
# 127.0.1.1      compute1

192.168.10.10   controller
192.168.10.20   compute1
```

> 虚机拷贝时注意更改

### 网络时间同步

#### controller

`# apt install chrony`

`# vim /etc/chrony/chrony.conf`

```
# pool 2.debian.pool.ntp.org offline iburst
server s1a.time.edu.cn iburst	# 北邮ntp服务器

allow 192.168.10.0/24
```

> 虚机拷贝时注意更改

`# service chrony restart`

`# chronyc sources`

 ![1](D:\BOX\openstack-pic\1.png)

#### compute1

`# apt install chrony`

`# vim /etc/chrony/chrony.conf`

```
# pool 2.debian.pool.ntp.org offline iburst
server controller iburst
```

`# service chrony restart`

`# chronyc sources`

 ![2](D:\BOX\openstack-pic\2.png)



### 安装包更新

#### controller

`# add-apt-repository cloud-archive:queens`

`# apt update`

`# apt dist-upgrade`

`# apt install python3-openstackclient`

#### compute1

`# add-apt-repository cloud-archive:queens`

`# apt update`

`# apt dist-upgrade`

`# apt install python3-openstackclient`



### SQL数据库安装

#### controller

`# apt install mariadb-server python-pymysql `

`# vim /etc/mysql/mariadb.conf.d/99-openstack.cnf`

```
[mysqld]
bind-address = 192.168.10.10

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```

> 虚机拷贝时注意更改

`# service mysql restart`

`# mysql_secure_installation`



### 消息队列安装

#### controler

`# apt install rabbitmq-server`

`# rabbitmqctl add_user openstack 123	`

允许openstack用户进行配置、写入和读取操作

`# rabbitmqctl set_permissions openstack ".*" ".*" ".*"`



### Memcached安装

### controller

`# apt install memcached python-memcache`

`# vim /etc/memcached.conf`

```
# -l 127.0.0.1
-l 192.168.10.10
```

> 虚机拷贝时注意更改

`# service memcached restart`



### Etcd安装

#### controller

`# apt install etcd`

`# vim /etc/default/etcd`

```
ETCD_NAME="controller"
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER="controller=http://192.168.10.10:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.10.10:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.10.10:2379"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.10.10:2379"
```

> 虚机拷贝时注意更改

`# systemctl enable etcd`

`# systemctl restart etcd`

