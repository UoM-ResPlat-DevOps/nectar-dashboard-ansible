# uom-cloud-dashboard-ansible

This repository was forked from [NeCTAR-RC/nectar-dashboard-ansible](https://github.com/NeCTAR-RC/nectar-dashboard-ansible/).

This playbook has been tested to be stable with the following configuration:
```
Flavor : m2.tiny (1 VCPU, 768BM RAM, 5GB disk)
Image  : NeCTAR Ubuntu 18.04 LTS (Bionic) amd64
Ports  : ssh, http (limit IP ranges to prevent public access)
```

----
### Install dependencies:
```
sudo apt install -y ansible python-pip git vim
git clone https://github.com/UoM-ResPlat-DevOps/uom-cloud-dashboard-ansible.git
```

### Configure the playbook

Edit [roles/horizon/defaults/main.yml](roles/horizon/defaults/main.yml).

Ensure that you change the following variables:
```
### Set the server hostname (e.g. the IP address)
horizon_server_name: my.hostname.tld

### Set the Horizon secret key
horizon_secret_key: <random_hash>

### Set your desired database passwords
mysql_root_password: <your password>
horizon_mysql_password: <your password>

### Remember to choose a branch you want to edit from. e.g.:
nectar_dashboard_version: nectar/queens-develop

### Disable murano unless you want it
horizon_murano_enabled: False

### Remember to set which keystone to point to.
horizon_keystone_host: "keystone.test.rc.nectar.org.au"
```

If you wish to disable AAF:
```
disable_aaf: True
```

Edit the ``allocations_`` variables if you intend to use the allocations
system.

### Enabling SSL

SSL is disabled by default.

When enabling SSL ensure that you edit [group_vars/all.yml](group_vars/all.yml)
and uncomment the SSL certificate variables (and input file locations):
```
horizon_user_ssl_cert: "{{ lookup('file', 'credentials/ssl/nectar-wildcard-certcombined.pem') }}"
horizon_user_ssl_key: "{{ lookup('file', 'credentials/ssl/nectar-wildcard-key.pem') }}"
```

### Run playbook
```
sudo ansible-playbook -i "horizon_all," -c local playbook.yml
```

### Test
- Go to `http://<hostname>`
- Enter your keystone username & password to log in

### Default locations for development

| Location | Description |
| --- | --- |
| /opt/uom-cloud-dashboard/ | Extensions code (git). Make changes here |
| /opt/horizon/lib/python2.7/site-packages/openstack_dashboard/ | Installed Horizon location |
| /etc/horizon/local_settings.py | Installed local_settings.py file |
| /var/lib/horizon/dashboard.sqlite3 | sqlite3 database |
| /opt/horizon/bin/horizon-manage.py | For running Django commands. |
| /var/log/horizon/ | Location of logs. |
