#### Официальная документация (на английском):

- **Основной сайт**: [https://cloudinit.readthedocs.io/](https://cloudinit.readthedocs.io/?spm=a2ty_o01.29997173.0.0.bdbec921SHbUE4)
- **Список всех модулей**: [https://cloudinit.readthedocs.io/en/latest/reference/modules.html](https://cloudinit.readthedocs.io/en/latest/reference/modules.html)
- **Примеры конфигов**: [https://cloudinit.readthedocs.io/en/latest/faq.html#where-can-i-find-examples-of-cloud-config](https://cloudinit.readthedocs.io/en/latest/faq.html#where-can-i-find-examples-of-cloud-config)
####  На русском (меньше):

- [https://help.ubuntu.com/community/CloudInit](https://help.ubuntu.com/community/CloudInit)
- В официальной документации Ubuntu: [https://ubuntu.com/server/docs/cloud-init](https://ubuntu.com/server/docs/cloud-init)

| Возможность                 | Пример в YAML                                               |
| --------------------------- | ----------------------------------------------------------- |
| **Добавить SSH-ключи**      | `ssh_authorized_keys: [ "ssh-ed25519 AAA..." ]`             |
| **Создать пользователей**   | `users: [ { name: "app", shell: "/bin/bash" } ]`            |
| **Установить пакеты**       | `packages: [ nginx, docker.io ]`                            |
| **Запустить команды**       | `runcmd: [ "systemctl restart nginx" ]`                     |
| **Записать файлы**          | `write_files: [ { path: "/etc/motd", content: "Hello!" } ]` |
| **Настроить сеть**          | `network: { version: 2, ethernets: ... }`                   |
| **Запустить systemd-юниты** | `bootcmd`, `runcmd`, или через `write_files` + `systemd`    |