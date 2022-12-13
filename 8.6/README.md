### 8.6.Создание собственных модулей

1) Запущено виртуальное окружение ```. venv/bin/activate && . hacking/env-setup```
2) создан и откорректирован в соответствии с заданием файл my_module.py
3) запущен локальный тест
```shell
(venv) vvk@bubuntu:~/dz/ansible$ python -m ansible.modules.my_module input.json
{"changed": true, "message": "File created", "invocation": {"module_args": {"path": "/tmp/1/2/some_file.txt", "content": "Use the real uid/gid to test for access to path.\nNote that most operations will use the effective uid/gid, therefore\nthis routine can be used in a suid/sgid environment to test if the\ninvoking user has the specified access to path. mode should be F_OK to test the existence\nof path, or it can be the inclusive OR of one or more of R_OK, W_OK, and X_OK to test permissions.\nReturn True if access is allowed, False if not. See the Unix man page access(2) for more information.\n"}}}

```
Содержимое созданного файла:
```shell
(venv) vvk@bubuntu:~/dz/ansible$ cat /tmp/1/2/some_file.txt 
Use the real uid/gid to test for access to path.
Note that most operations will use the effective uid/gid, therefore
this routine can be used in a suid/sgid environment to test if the
invoking user has the specified access to path. mode should be F_OK to test the existence
of path, or it can be the inclusive OR of one or more of R_OK, W_OK, and X_OK to test permissions.
Return True if access is allowed, False if not. See the Unix man page access(2) for more information.
```

4) Проверка работы playbook 
Содержимое playbook:
```yaml
---

- name: Single task pb
  hosts: localhost
  tasks:
  - name: Run my_module
    my_own_namespace.yandex_cloud_elk.my_module:
      path: '/tmp/1/2.txt'
      content: 'test content'
```
Вывод:

```shell
(venv) vvk@bubuntu:~/dz/ansible$ ansible-playbook site.yml 
[WARNING]: You are running the development version of Ansible. You should only run Ansible from "devel" if you are modifying the Ansible engine, or trying out features under
development. This is a rapidly changing source of code and can become unstable at any point.
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] ****************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [localhost]

TASK [test my_module] ***********************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP **********************************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Проверяем на идемпотентность:
```shell
(venv) vvk@bubuntu:~/dz/ansible$ ansible-playbook site.yml 
[WARNING]: You are running the development version of Ansible. You should only run Ansible from "devel" if you are modifying the Ansible engine, or trying out features under
development. This is a rapidly changing source of code and can become unstable at any point.
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] ****************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [localhost]

TASK [test my_module] ***********************************************************************************************************************************************************
ok: [localhost]

PLAY RECAP **********************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
5) Выход из окружения ( *deactivate* ), создание коллекции:
```shell
vvk@bubuntu:~/dz/7.1/ans_try/8.6$ ansible-galaxy collection init my_own_namespace.yandex_cloud_elk
- Collection my_own_namespace.yandex_cloud_elk was created successfully
```
Перенес свой модуль:
```shell
$ tree 
.
├── docs
├── galaxy.yml
├── plugins
│   ├── modules
│   │   └── my_module.py
│   └── README.md

```
"Упаковка" коллекции: 
``` ansible-galaxy collection build ```
Проверка установки из архива:
```ansible-galaxy collection install my_own_namespace-yandex_cloud_elk-1.0.0.tar.gz```
Создание single task роли:
```shell
ansible-galaxy role init my_single_task_role
- Role my_single_task_role was created successfully
```
Заполнил параметры по умолчанию:
```shell
---
# defaults file for my_single_task_role
path: '/tmp/some_file.txt'
content: 'some strings here'
```
6) Проверил работу playbook:
```shell
vvk@bubuntu:~/dz/7.1/ans_try/8.6$ ansible-playbook site.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
ERROR! couldn't resolve module/action 'my_own_namespace.yandex_elk.my_module'. This often indicates a misspelling, missing collection, or incorrect module path.

The error appears to be in '/home/vvk/dz/7.1/ans_try/8.6/site.yml': line 6, column 7, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

  tasks:
    - name: Run my_module
      ^ here
vvk@bubuntu:~/dz/7.1/ans_try/8.6$ ansible-playbook site.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
ERROR! couldn't resolve module/action 'my_own_namespace.yandex_elk.my_module'. This often indicates a misspelling, missing collection, or incorrect module path.

The error appears to be in '/home/vvk/dz/7.1/ans_try/8.6/site.yml': line 6, column 5, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

  tasks:
  - name: Run my_module
    ^ here
vvk@bubuntu:~/dz/7.1/ans_try/8.6$ ansible-playbook site.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Single task pb] ***********************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [localhost]

TASK [Run my_module] ************************************************************************************************************************************************************
fatal: [localhost]: FAILED! => {"changed": false, "msg": "missing required arguments: content"}

PLAY RECAP **********************************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

vvk@bubuntu:~/dz/7.1/ans_try/8.6$ ansible-playbook site.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Single task pb] ***********************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [localhost]

TASK [Run my_module] ************************************************************************************************************************************************************
ok: [localhost]

PLAY RECAP **********************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

vvk@bubuntu:~/dz/7.1/ans_try/8.6$ rm -rf /tmp/1
vvk@bubuntu:~/dz/7.1/ans_try/8.6$ ansible-playbook site.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Single task pb] ***********************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [localhost]

TASK [Run my_module] ************************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP **********************************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

vvk@bubuntu:~/dz/7.1/ans_try/8.6$ ansible-playbook site.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Single task pb] ***********************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************
ok: [localhost]

TASK [Run my_module] ************************************************************************************************************************************************************
ok: [localhost]

PLAY RECAP **********************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```
7) Работа опубликована tag 1.0.0 https://github.com/Bambrino/my_own_collection

8) Архив и playbook находятся в папке <a href="https://github.com/Bambrino/my_own_collection/tree/main/Arch%26pbook"> Arch&pbook </a>