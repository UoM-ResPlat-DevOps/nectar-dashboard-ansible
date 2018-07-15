# uom-cloud-dashboard-ansible

This repository was forked from [NeCTAR-RC/nectar-dashboard-ansible](https://github.com/NeCTAR-RC/nectar-dashboard-ansible/).

To execute the playbook on a local machine:
```
sudo ansible-playbook -i "horizon_all," -c local playbook.yml
```

----
## Notes on new "dev" machine

It can be useful to have a development virtual machine for web devs to make changes with minimal setup.

The following notes are for making a instance with:
- Horizon + dashboard extensions
- SSL certs disabled
- Disable AAF/Shibb sign-on + Enable keystone user/pass sign-on
- SQLite backend


Notes based on:
```
Flavor : m2.tiny (1 VCPU, 768BM RAM, 5GB disk)
Image  : NeCTAR Ubuntu 18.04 LTS (Bionic) amd64
Ports  : ssh, http (limit IP ranges to prevent public access)
```

### Install dependencies:
```
sudo apt install -y ansible python-pip git
git clone https://github.com/UoM-ResPlat-DevOps/uom-cloud-dashboard-ansible.git
```

### Make adjustments for a "dev" machine.

group_vars/all.yml
```
### Comment out the ssl cert variables

#horizon_user_ssl_cert: "{{ lookup('file', 'credentials/ssl/nectar-wildcard-certcombined.pem') }}"
#horizon_user_ssl_key: "{{ lookup('file', 'credentials/ssl/nectar-wildcard-key.pem') }}"
```

roles/horizon/defaults/main.yml
```
### Remember to choose a branch you want to edit from. e.g.:
nectar_dashboard_version: nectar/queens-develop

### Disable murano unless you want it
horizon_murano_enabled: False

### Remember to set which keystone to point to.
horizon_keystone_host: "keystone.test.rc.nectar.org.au"
```


roles/horizon/tasks/horizon_apache.yml
```
### Comment out the mpm_worker.
#- { state: present, name: mpm_worker }
```

roles/horizon/templates/horizon_local_settings.py.j2
```
### Disable SSO to disconnect from AAF/Shibb

WEBSSO_ENABLED = False
```



### Execute playbook
```
sudo ansible-playbook -i "horizon_all," -c local playbook.yml
```

### Post playbook install
```
### Remove the _description.html and _login.html overrides so that the regular Username/Password fields appear
cd /opt/uom-cloud-dashboard/theme/templates/auth/
sudo mv _description.html _description.html.bak
sudo mv _login.html _login.html.bak

### Add the enables/disables from extensions repo + do Django stuff to update
sudo cp --symbolic-link /opt/uom-cloud-dashboard/enabled/*.py  /opt/horizon/lib/python2.7/site-packages/openstack_dashboard/local/enabled/
cd /opt/horizon/lib/python2.7/site-packages/openstack_dashboard/
sudo /opt/horizon/bin/horizon-manage.py migrate rcallocation
sudo /opt/horizon/bin/horizon-manage.py compress

### Need to restart apache2 to complete the changes
sudo systemctl restart apache2
```


### Test
- Go to http://(ip address)
- Enter your keystone username & password to log in


### Ready for local dev

| Location | Description |
| --- | --- |
| /opt/uom-cloud-dashboard/ | Extensions code (git). Make changes here |
| /opt/horizon/lib/python2.7/site-packages/openstack_dashboard/ | Installed Horizon location |
| /etc/horizon/local_settings.py | Installed local_settings.py file |
| /var/lib/horizon/dashboard.sqlite3 | sqlite3 database |
| /opt/horizon/bin/horizon-manage.py | For running Django commands. |
| /var/log/horizon/ | Location of logs. |
