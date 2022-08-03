# OpenStack安装指南（5）

## Horizon安装

> [Horizon Installation Tutorial](https://docs.openstack.org/horizon/queens/install/)



### controller

#### 安装配置

`# apt install openstack-dashboard`

`# vim /etc/openstack-dashboard/local_settings.py`

```
# OPENSTACK_HOST = "127.0.0.1"
OPENSTACK_HOST = "controller"

# ALLOWED_HOSTS = ['horizon.example.com', ]
ALLOWED_HOSTS = ['*',]
# 不要更改Ubuntu Settings下的ALLOWED_HOSTS

SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        # 'LOCATION': '127.0.0.1:11211',
        'LOCATION': 'controller:11211',
    },

# OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = False
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True

OPENSTACK_API_VERSIONS = {
#    "data-processing": 1.1,
    "identity": 3,
    "image": 2,
    "volume": 2,
#    "compute": 2,
}

# OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'Default'
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'Default'

# OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST

# OPENSTACK_KEYSTONE_DEFAULT_ROLE = "_member_"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
```

`# service apache2 reload`

可以通过http://controller/horizon访问
