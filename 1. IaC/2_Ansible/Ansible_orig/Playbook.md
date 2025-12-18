
Плейбуки Ansible - это повторяющаяся, многократно используемая, простая система управления конфигурацией и развертывания на нескольких машинах, которая хорошо подходит для развертывания сложных приложений. Если вам нужно выполнить задачу с помощью Ansible более одного раза, напишите плейбук и поместите его под контроль исходного кода. Затем вы можете использовать плейбук для распространения новой конфигурации или подтверждения конфигурации удаленных систем.

Плейбуки могут:
- объявлять конфигурации
- оркестровать шаги любого ручного процесса на нескольких наборах машин в определенном порядке
- запускать задачи синхронно или асинхронно
#### Синтаксис
Плейбуки выражаются в формате YAML с минимальным количеством синтаксиса. 

Playbook состоит из одного или нескольких «play» в виде упорядоченного списка. Термины «playbook» и «play» - это спортивные аналогии. Каждый play выполняет часть общей цели плейбука, запуская одну или несколько задач. Каждая задача вызывает module Ansible.

Пример плейбука:

```
---
- name: Update web servers
  hosts: webservers
  remote_user: root

  tasks:
  - name: Ensure apache is at the latest version
    ansible.builtin.yum:
      name: httpd
      state: latest

  - name: Write the apache config file
    ansible.builtin.template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf

- name: Update db servers
  hosts: databases
  remote_user: root

  tasks:
  - name: Ensure postgresql is at the latest version
    ansible.builtin.yum:
      name: postgresql
      state: latest

  - name: Ensure that postgresql is started
    ansible.builtin.service:
      name: postgresql
      state: started
```

Плейбук запускается в порядке сверху вниз. Внутри каждого play имеются задачи (tasks) , они также выполняются в  сверху вниз. Плейбуки с несколькими «play» могут организовывать развертывание нескольких машин, запуская однин play на веб-серверах, затем другой на серверах баз данных, третий на сетевой инфраструктуре и так далее. Как минимум, каждый play определяет две вещи:

- управляемые узлы, на которые нужно нацелиться, используя шаблон
- как минимум одну задачу для выполнения
#### [[Запуск playbook]]


> [!NOTE] Сыылка на оригинальный источник
> https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html#playbooks-intros

