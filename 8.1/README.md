## 08.01 Введение в Ansible

1) 
```shell
$ ansible-playbook -i ./inventory/test.yml site.yml 
...
TASK [Print fact] ****************************************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": 12
}
...
```

2) 
```shell
$ cat ./group_vars/all/examp.yml 
---
  some_fact: "12"

$ cat ./group_vars/all/examp.yml 
---
  some_fact: "all default fact"

```

3) 
```yaml
version: "3"

services:

  centos:
    image: centos:7
    container_name: centos
    tty: true

  debian:
    image: python:bullseye
    container_name: debian
    tty: true
```

4) 
```shell
$ ansible-playbook -i ./inventory/prod.yml site.yml 
...
TTASK [Print fact] ****************************************************************************************************************************************************************************************************************************************************************
ok: [centos] => {
    "msg": "el"
}
ok: [debian] => {
    "msg": "deb"
}
... 
```
5) 
```shell
$ cat ./group_vars/deb/examp.yml
---
  some_fact: "deb default fact"
```
```shell
$ cat ./group_vars/el/examp.yml
---
  some_fact: "el default fact"

```

6) 
```shell
$ ansible-playbook -i ./inventory/prod.yml site.yml 
...
TASK [Print fact] ****************************************************************************************************************************************************************************************************************************************************************
ok: [centos] => {
    "msg": "el default fact"
}
ok: [debian] => {
    "msg": "deb default fact"
}
...
```
7) 
```shell
$ ansible-vault encrypt_string 'el default fact'
New Vault password: 
Confirm New Vault password: 
Encryption successful
```
```shell
$ ansible-vault encrypt_string 'deb default fact'
New Vault password: 
Confirm New Vault password: 
Encryption successful
```
```shell
$ cat ./group_vars/deb/examp.yml
---
  some_fact: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          35653435353134383932663864363436653736356639383439363831366161363665623730663034
          6661343139363132326431643065666334653364613035610a613861363264643737646636323939
          65646632383636316162323235666137643735373235373862383035316131343764623733623266
          6263393538653031300a633561613262643965356333373131343832396566663562653861663636
          61343764613831313438663931316534336136663461363139356565373038326362
```
```shell
$ cat ./group_vars/el/examp.yml 
---
  some_fact: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          66393661376138373338333435373338393235366162313334616530373465303963656637393238
          3662643531663965383731353065346431323038313833640a656537306465343539616630306331
          33393337646539316439666337643730323838343561623165666631376637333037646532653331
          6435633939346235320a643134616231333461373230373534396364613639633064386239306638
          3131
```
8) 
```shell
$ ansible-playbook -i ./inventory/prod.yml --ask-vault-pass site.yml 
Vault password: 
...
TASK [Print fact] ****************************************************************************************************************************************************************************************************************************************************************
ok: [centos] => {
    "msg": "el default fact"
}
ok: [debian] => {
    "msg": "deb default fact"
}
...
```
9) 
```shell
$ ansible-doc -t connection -l
...                                                                                                                                                                                                   
local                          execute on controller                                                                                                                                                                                                                         
...
```
10) 
```shell
$ cat ./inventory/prod.yml 
---
  el:
    hosts:
      centos:
        ansible_connection: docker

  deb:
    hosts:
      debian:
        ansible_connection: docker

  local:
    hosts:
      localhost:
        ansible_connection: local
```
11) 
```shell
1$ ansible-playbook -i ./inventory/prod.yml --ask-vault-pass site.yml 
Vault password: 

PLAY [Print os facts] ************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************************************************************************
ok: [localhost]
ok: [debian]
ok: [centos]

TASK [Print OS] ******************************************************************************************************************************************************************************************************************************************************************
ok: [centos] => {
    "msg": "CentOS"
}
ok: [debian] => {
    "msg": "Debian"
}
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ****************************************************************************************************************************************************************************************************************************************************************
ok: [centos] => {
    "msg": "el default fact"
}
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [debian] => {
    "msg": "deb default fact"
}

PLAY RECAP ***********************************************************************************************************************************************************************************************************************************************************************
centos                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
debian                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
