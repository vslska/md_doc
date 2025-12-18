Создаем каталог (roles)
Заходим в него и выполняем комманду ansible-galaxy init (имя роли)
Пример:
ansible-galaxy init deploy_nginx

```
tree
.
└── deploy_nginx
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── README.md
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml
```

В каталге будут созданы файлы ( см. выше), по своей сути файлы пустые, теперь наш плейбук можно разнести по данным файлам,  отступы можно сделать слева (пример handlers)
```
---
# handlers file for deploy_nginx
- name: Restart Nginx Debian
  systemd:
  name: nginx
  state: restarted
```

