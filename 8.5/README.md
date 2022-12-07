## 08.05 Тестирование roles

Сделано:

##### Prereq - установлено:
```shell
pip3 install "molecule==3.5.2"
pip3 install "ansible-lint"
pip3 install "flake8"
pip3 install "molecule_docker"
```
#### Molecule
1) запущен сценарий ```molecule test -s centos7``` для роли clickhouse (исправлено в converge с ansible-clickhouse на clickhouse)

<s>

```
        name: "ansible-clickhouse"
```

</s>

```name: "clickhouse" ```

<details>
        <summary> Конец вывода </summary>

```shell
....

TASK [clickhouse : Config | Create database config] ****************************
skipping: [centos_7] => (item={'name': 'testu1'}) 
skipping: [centos_7] => (item={'name': 'testu2'}) 
skipping: [centos_7] => (item={'name': 'testu3'}) 
skipping: [centos_7] => (item={'name': 'testu4', 'state': 'absent'}) 

TASK [clickhouse : include_tasks] **********************************************
included: /home/vvk/dz/7.1/8.5/roles/clickhouse/tasks/configure/dict.yml for centos_7

TASK [clickhouse : Config | Generate dictionary config] ************************
ok: [centos_7]

TASK [clickhouse : include_tasks] **********************************************
skipping: [centos_7]

PLAY RECAP *********************************************************************
centos_7                   : ok=25   changed=0    unreachable=0    failed=0    skipped=9    rescued=0    ignored=0

INFO     Idempotence completed successfully.
INFO     Running centos_7 > side_effect
WARNING  Skipping, side effect playbook not configured.
INFO     Running centos_7 > verify
INFO     Running Ansible Verifier

PLAY [Verify] ******************************************************************

TASK [Example assertion] *******************************************************
ok: [centos_7] => {
    "changed": false,
    "msg": "All assertions passed"
}

PLAY RECAP *********************************************************************
centos_7                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Verifier completed successfully.
INFO     Running centos_7 > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running centos_7 > destroy

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos_7)

TASK [Wait for instance(s) deletion to complete] *******************************
FAILED - RETRYING: [localhost]: Wait for instance(s) deletion to complete (300 retries left).
changed: [localhost] => (item=centos_7)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory
```

</details>

<br>

2) Создан сценарий для роли vector ```molecule init scenario --driver-name docker```
Сценарий default на CentOS 8 не заработал, поменял image docker`а на Centos7 \
```image: docker.io/pycontribs/centos:7```
В сценариях тестирования определены шаги (убраны неиспользованные мной), добавлен lint; исправлены ошибки по форматированиию (лишние пробелы, строки и т.п.) и именованию (добавлены полные конструкции ansible.builtin.[]), изменено название роли с ansible-role на ansible_role - ошибка проверки допустивых символов в названии роли в meta. Также добавлено ```changed_when: false``` в task вывода версии vector по окончанию установки

```yaml
...
lint: |
  yamllint .
  ansible-lint
  flake8
...
scenario:
  test_sequence:
    - destroy
    - lint
    - syntax
    - create
    - converge
    - idempotence
    - verify
    - destroy

```

```yaml
---
- name: Get vector version
  ansible.builtin.command: "vector --version"
  register: vector_info
  changed_when: false

- name: Print version
  ansible.builtin.debug:
    var: vector_info.stdout
  changed_when: false
```

```molecule test -s default```

<details>
      <summary> Конец вывода </summary>

```shell
....

INFO     Running default > idempotence

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [centos_7]

TASK [Include vector_role] *****************************************************

TASK [vector_role : Download vector rpm] ***************************************
ok: [centos_7]

TASK [vector_role : Install vector rpm] ****************************************
ok: [centos_7]

TASK [vector_role : Get vector version] ****************************************
ok: [centos_7]

TASK [vector_role : Print version] *********************************************
ok: [centos_7] => {
    "vector_info.stdout": "vector 0.22.2 (x86_64-unknown-linux-gnu 0024c92 2022-06-15)"
}

PLAY RECAP *********************************************************************
centos_7                   : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Idempotence completed successfully.
INFO     Running default > verify
INFO     Running Ansible Verifier

PLAY [Verify] ******************************************************************

TASK [Include vars] ************************************************************
ok: [centos_7]

TASK [Get version] *************************************************************
ok: [centos_7]

TASK [Check version] ***********************************************************
ok: [centos_7] => {
    "changed": false,
    "msg": "vector 0.22.2 (x86_64-unknown-linux-gnu 0024c92 2022-06-15)"
}

PLAY RECAP *********************************************************************
centos_7                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Verifier completed successfully.
INFO     Running default > destroy

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos_7)

TASK [Wait for instance(s) deletion to complete] *******************************
FAILED - RETRYING: [localhost]: Wait for instance(s) deletion to complete (300 retries left).
changed: [localhost] => (item=centos_7)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory

```
</details>
<br>

3) Создан сценарий для CentOS8 - "выпадания" с ошибкой на разных образах разбирались на вэбинаре, собрал свой образ.

4) Создан assert:
```yaml
---
- name: Verify
  hosts: all
  gather_facts: false
  tasks:
  - name: 'Include vars'
    ansible.builtin.include_vars:
      file: ../../defaults/main.yml
      name: vector
  - name: 'Get version'
    ansible.builtin.command: vector \--version
    changed_when: false
    register: vector_std
  - name: 'Check version'
    ansible.builtin.assert:
      that:
        - "'{{ vector.vector_version }}' in vector_std.stdout"
      success_msg: "{{ vector_std.stdout }}"
```
5) Запуск ```molecule test -s centos_8```
  <details>
      <summary> Полный вывод </summary>

```shell
INFO     centos_8 scenario test matrix: destroy, lint, syntax, create, converge, idempotence, verify, destroy
INFO     Performing prerun with role_name_check=0...
INFO     Set ANSIBLE_LIBRARY=/home/vvk/.cache/ansible-compat/e3fa2b/modules:/home/vvk/.ansible/plugins/modules:/usr/share/ansible/plugins/modules
INFO     Set ANSIBLE_COLLECTIONS_PATH=/home/vvk/.cache/ansible-compat/e3fa2b/collections:/home/vvk/.ansible/collections:/usr/share/ansible/collections
INFO     Set ANSIBLE_ROLES_PATH=/home/vvk/.cache/ansible-compat/e3fa2b/roles:/home/vvk/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
INFO     Using /home/vvk/.cache/ansible-compat/e3fa2b/roles/myownspace.vector_role symlink to current repository in order to enable Ansible to find the role using its expected full name.
INFO     Running centos_8 > destroy
INFO     Sanity checks: 'docker'

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos_8)

TASK [Wait for instance(s) deletion to complete] *******************************
FAILED - RETRYING: [localhost]: Wait for instance(s) deletion to complete (300 retries left).
ok: [localhost] => (item=centos_8)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Running centos_8 > lint
WARNING: PATH altered to include /usr/bin
INFO     Running centos_8 > syntax

playbook: /home/vvk/dz/7.1/ans_try/8.5/roles/vector_role/molecule/centos_8/converge.yml
INFO     Running centos_8 > create

PLAY [Create] ******************************************************************

TASK [Log into a Docker registry] **********************************************
skipping: [localhost] => (item=None) 
skipping: [localhost]

TASK [Check presence of custom Dockerfiles] ************************************
ok: [localhost] => (item={'image': 'bambrino/centos:8', 'name': 'centos_8'})

TASK [Create Dockerfiles from image names] *************************************
changed: [localhost] => (item={'image': 'bambrino/centos:8', 'name': 'centos_8'})

TASK [Discover local Docker images] ********************************************
ok: [localhost] => (item={'diff': [], 'dest': '/home/vvk/.cache/molecule/vector_role/centos_8/Dockerfile_bambrino_centos_8', 'src': '/home/vvk/.ansible/tmp/ansible-tmp-1670407819.008274-196938-221934042038583/source', 'md5sum': 'eb299fe8a2c76b89f8302b3f8ddfbf9e', 'checksum': 'f92a8b343016d31b6400210088fa6f10b5ef8597', 'changed': True, 'uid': 1000, 'gid': 1000, 'owner': 'vvk', 'group': 'vvk', 'mode': '0600', 'state': 'file', 'size': 1046, 'invocation': {'module_args': {'src': '/home/vvk/.ansible/tmp/ansible-tmp-1670407819.008274-196938-221934042038583/source', 'dest': '/home/vvk/.cache/molecule/vector_role/centos_8/Dockerfile_bambrino_centos_8', 'mode': '0600', 'follow': False, '_original_basename': 'Dockerfile.j2', 'checksum': 'f92a8b343016d31b6400210088fa6f10b5ef8597', 'backup': False, 'force': True, 'unsafe_writes': False, 'content': None, 'validate': None, 'directory_mode': None, 'remote_src': None, 'local_follow': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None}}, 'failed': False, 'item': {'image': 'bambrino/centos:8', 'name': 'centos_8'}, 'ansible_loop_var': 'item', 'i': 0, 'ansible_index_var': 'i'})

TASK [Build an Ansible compatible image (new)] *********************************
ok: [localhost] => (item=molecule_local/bambrino/centos:8)

TASK [Create docker network(s)] ************************************************

TASK [Determine the CMD directives] ********************************************
ok: [localhost] => (item={'image': 'bambrino/centos:8', 'name': 'centos_8'})

TASK [Create molecule instance(s)] *********************************************
changed: [localhost] => (item=centos_8)

TASK [Wait for instance(s) creation to complete] *******************************
FAILED - RETRYING: [localhost]: Wait for instance(s) creation to complete (300 retries left).
changed: [localhost] => (item={'failed': 0, 'started': 1, 'finished': 0, 'ansible_job_id': '47955819360.197145', 'results_file': '/home/vvk/.ansible_async/47955819360.197145', 'changed': True, 'item': {'image': 'bambrino/centos:8', 'name': 'centos_8'}, 'ansible_loop_var': 'item'})

PLAY RECAP *********************************************************************
localhost                  : ok=7    changed=3    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

INFO     Running centos_8 > converge

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [centos_8]

TASK [Include vector-role] *****************************************************

TASK [vector_role : Download vector rpm] ***************************************
changed: [centos_8]

TASK [vector_role : Install vector rpm] ****************************************
changed: [centos_8]

TASK [vector_role : Get vector version] ****************************************
ok: [centos_8]

TASK [vector_role : Print version] *********************************************
ok: [centos_8] => {
    "vector_info.stdout": "vector 0.22.2 (x86_64-unknown-linux-gnu 0024c92 2022-06-15)"
}

PLAY RECAP *********************************************************************
centos_8                   : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Running centos_8 > idempotence

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [centos_8]

TASK [Include vector-role] *****************************************************

TASK [vector_role : Download vector rpm] ***************************************
ok: [centos_8]

TASK [vector_role : Install vector rpm] ****************************************
ok: [centos_8]

TASK [vector_role : Get vector version] ****************************************
ok: [centos_8]

TASK [vector_role : Print version] *********************************************
ok: [centos_8] => {
    "vector_info.stdout": "vector 0.22.2 (x86_64-unknown-linux-gnu 0024c92 2022-06-15)"
}

PLAY RECAP *********************************************************************
centos_8                   : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Idempotence completed successfully.
INFO     Running centos_8 > verify
INFO     Running Ansible Verifier

PLAY [Verify] ******************************************************************

TASK [Include vars] ************************************************************
ok: [centos_8]

TASK [Get version] *************************************************************
ok: [centos_8]

TASK [Check version] ***********************************************************
ok: [centos_8] => {
    "changed": false,
    "msg": "vector 0.22.2 (x86_64-unknown-linux-gnu 0024c92 2022-06-15)"
}

PLAY RECAP *********************************************************************
centos_8                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Verifier completed successfully.
INFO     Running centos_8 > destroy

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos_8)

TASK [Wait for instance(s) deletion to complete] *******************************
FAILED - RETRYING: [localhost]: Wait for instance(s) deletion to complete (300 retries left).
changed: [localhost] => (item=centos_8)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory

```
</details>
<br>


5) Опубликовано с тегом <a href="https://github.com/Bambrino/vector_role/tree/0.1.0">0.1.0</a>

#### Tox

1) Скачаны указанные файлы tox.ini и tox-requiremets.txt
2) Запуск ```docker run --privileged=True -v ~/dz/7.1/ans_try/8.5/roles/vector_role:/opt/vector_role -w /opt/vector_role -it aragast/netology:latest /bin/bash``` выдал ошибку об отсутствии сценария compatibility
3) Создан сценарий ```molecule init scenario compatibility --driver-name podman```
4) Повторный запуск tox прошел успешно:
```shell
...
INFO     Pruning extra files from scenario ephemeral directory
______________________________________________________________________________________ summary _______________________________________________________________________________________
  py37-ansible210: commands succeeded
  py37-ansible30: commands succeeded
  py39-ansible210: commands succeeded
  py39-ansible30: commands succeeded
  congratulations :)
```
5) Опубликовано с тегом <a href="https://github.com/Bambrino/vector_role/tree/0.2.0">0.2.0</a>
