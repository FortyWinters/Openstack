# OpenStack指南（7）

## 网络与实例创建

#### Admin-Network-Networks

##### ext-net

 <img src="D:\BOX\openstack-pic\4.png" alt="4" style="zoom: 80%;" />

 <img src="D:\BOX\openstack-pic\5.png" alt="5" style="zoom:80%;" />

 <img src="D:\BOX\openstack-pic\6.png" alt="6" style="zoom:80%;" />

##### int-net

####  <img src="D:\BOX\openstack-pic\7.png" alt="7" style="zoom:80%;" />

 <img src="D:\BOX\openstack-pic\8.png" alt="8" style="zoom:80%;" />

 <img src="D:\BOX\openstack-pic\9.png" alt="9" style="zoom:80%;" />



#### Project-Network-Security Groups

<img src="D:\BOX\openstack-pic\10.png" alt="10" style="zoom:80%;" />



#### Project-Network-Network Topology

创建router连接ext-net和int-net，并创建实例

 <img src="D:\BOX\openstack-pic\12.png" alt="12" style="zoom:80%;" />

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

