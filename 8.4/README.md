## 08.04 Работа с Roles

Сделано:

- установлена роль clihouse (ansible-galaxy install -r requirements.yml)
- playbook разенсен по ролям (vector-role и lighthouse-role - ansible-galaxy init [role-name])
- указанные роли опубликованы в git
-  роли добавлены в requirements для использования через ansible-galaxy install -r requirements.yml
```shell
---
  - src: git@github.com:AlexeySetevoi/ansible-clickhouse.git
    scm: git
    version: "1.11.0"
    name: clickhouse 

  - src: git@github.com:Bambrino/vector-role.git
    scm: git
    version: "1.0"
    name: vector-role 

  - src: git@github.com:Bambrino/lighthouse-role.git
    scm: git
    version: "1.0"
    name: lighthouse-role

```
- Заполнено описание ролей в README.md каждой роли

    <a href="https://github.com/Bambrino/lighthouse-role"> lighthouse-role </a>
    <a href="https://github.com/Bambrino/vector-role"> vector-role </a>