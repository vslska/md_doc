##### Что такое `template` в Ansible?

Модуль `template` — это инструмент, который позволяет:

- Брать **шаблонный файл** (например, `.j2`).
- Подставлять в него **переменные из `group_vars`, `host_vars`, `defaults`, или `vars`** .
- Создавать **готовый конфигурационный файл** на удалённой машине.
##### Зачем использовать `template`?

1. **Конфигурации становятся гибкими**
    
    - Не нужно держать 10 копий `nginx.conf`.
    - Достаточно одного шаблона + разные переменные для разных хостов.
    
2. **Идемпотентность**
    
    - Каждый раз, когда ты запускаешь playbook, файл будет **создан заново** , но только если **изменились переменные** .

3. **Удобство масштабирования**
    
    - Ты хочешь поставить Apache на 5 серверов?  
        Напиши один шаблон и пять наборов переменных.
        
4. **Разделение данных и логики**
    
    - Шаблон описывает структуру.
    - Переменные — содержание.
    - Это чистота. Минимализм. Понятность.

#### Пример простого шаблона

##### Файл: `templates/apache2.conf.j2`

```
# /etc/apache2/apache2.conf — шаблон через Jinja2

ServerTokens Prod
ServerSignature Off
Timeouts {{ apache_timeout }}
MaxRequestsPerChild {{ max_requests }}

<Directory />
    Options {{ default_options }}
    AllowOverride {{ allow_override }}
    Require all granted
</Directory>
```
##### `group_vars/servers.yml`:

```
apache_timeout: 300
max_requests: 10000
default_options: MultiViews Indexes SymLinksIfOwnerMatch Includes ExecCGI
allow_override: None
```
##### `playbook`

```
- name: Configure Apache with Jinja2 template
  hosts: servers
  become: yes

  tasks:
    - name: Apply Apache config from template
      template:
        src: templates/apache2.conf.j2
        dest: /etc/apache2/apache2.conf

    - name: Restart Apache if config changed
      systemd:
        name: apache2
        state: restarted
      when: ansible_default_ipv4.address == "127.0.0.1"
```
#### Где применяется `template:`

#### 1. **Настройка сервисов**

- Nginx, Apache, PostgreSQL, MySQL, SSHD — всё, где есть `.conf` или `.cfg`.
#### 2. **Генерация HTML/JSON/INI файлов**

- Например, чтобы сделать `index.html` с текущим пользователем и временем:
```
<html><body>
  <h1>Ansible управлял этим сервером {{ ansible_date_time.date }}.</h1>
  <p>Пользователь: {{ ansible_user }}</p>
</body></html>
```

#### 3. **Файлы .env, .ini, .conf, .yaml, .json**

- Можно создавать `config.json` с разными значениями для dev/stage/prod.
- Или `.env` файлы с переменными окружения.
#### 4. **Шаблоны скриптов**

- Генерируешь bash-скрипт с переменными среды.
- Или даже свой `ansible.cfg` через Jinja2.

| Сценарий                  |  Как работает                                     |
| ------------------------- | ------------------------------------------------- |
| Настройка сайта в Nginx   | Один шаблон → много сайтов с разными доменами.    |
| Конфиг базы данных        | Меняешь только `db_host`, `db_name`, `db_user`.   |
| HTML страница приветствия | Выводишь IP, имя хоста, время.                    |
| Скрипт автоматизации      | Генерируешь bash-скрипт с переменными из Ansible. |
| Конфиг приложения         | Меняешь порт, юзера, режим — без рук в коде.      |