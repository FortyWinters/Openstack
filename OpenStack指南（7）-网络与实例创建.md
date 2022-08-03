# OpenStack指南（7）

## 网络与实例创建

#### Admin-Network-Networks

##### ext-net

```
Name: ext-net
Project: admin
Provider Network Type: Flat
Physcial Network: provider

Enable Admin State: yes
Shared: yes
External Network: yes
Create Subnet: yes
```



##### ext-subnet

```
Name: ext-subnet
Network Address: 192.168.20.0/24
IP Version: IPv4
Gateway IP: 192.168.20.1

Disable Gateway: no
Enable DHCP: yes

DNS Name Servers: 223.5.5.5
```



##### int-net

```
Name: int-net
Project: admin
Provider Network Type: VXLAN
Segmentation ID: 1

Enable Admin State: yes
Shared: no
External Network: no
Create Subnet: yes
```



##### int-subnet

```
Name: int-subnet
Network Address: 172.16.1.0/24
IP Version: IPv4
Gateway IP: 172.16.1.1

Disable Gateway: no
Enable DHCP: yes

DNS Name Servers: 223.5.5.5
```



#### Project-Network-Security Groups

设置ICMP、TCP、UDP的Egress、Ingress共六个安全组



#### Project-Network-Network Topology

创建router连接ext-net和int-net，并在int-net下创建实例

```
# ubuntu系统ssh连接
#!/bin/sh
passwd ubuntu<<EOF
123
123
EOF
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
service ssh restart
```

