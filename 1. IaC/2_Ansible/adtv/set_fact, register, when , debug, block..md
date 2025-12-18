Ниже краткое объяснение пяти ключевых директив и функций Ansible: **`set_fact`** , **`register`** , **`when`** , **`debug`** и **`block`** . Они очень часто используются при написании playbook'ов

| Модуль   | Назначение                              |
| -------- | --------------------------------------- |
| set_fact | Cоздание пользовательских переменных    |
| register | Сохранение результата выполнения задачи |
| when     | Условное выполнение задачи              |
| debug    | Отладка, и вывод информации             |
| block    | Группировка задачь с общими директивами |
##### 1. **`set_act`**
Позволяет задать пользовательские переменные в ходе выполнения playbook'а.
###### Пример:
```
- name: Установить свою переменную
  set_fact:
    my_var: "Hello World"
```

##### 2. **`register`**
Сохраняет результат выполнения задачи в переменную для дальнейшего использования.
###### Пример:
```
- name: Выполнить команду и сохранить результат
  shell: echo "Result"
  register: command_result

- name: Вывести результат
  debug:
    msg: "{{ command_result.stdout }}"
```

##### 3. **`when`**
Условие выполнения задачи — задача выполняется только если условие истинно.
###### Пример:
```
- name: Перезапустить сервис, если ОС Ubuntu
  service:
    name: nginx
    state: restarted
  when: ansible_distribution == "Ubuntu"
```

##### 4. **`debug`**
Выводит отладочную информацию (например, значения переменных) в консоль во время выполнения.
###### Пример:
```
- name: Вывести значение переменной
  debug:
    msg: "Текущий хост: {{ inventory_hostname }}"

- name: Вывести все доступные переменные
  debug:
    var: hostvars
```

##### 5. **`block`**
Группирует несколько задач под одним блоком, чтобы применить общие параметры, например `when`, `tags`, `rescue`, `always`
###### Пример:
```
- block:
    - name: Задача 1
      debug:
        msg: "Задача 1"

    - name: Задача 2
      debug:
        msg: "Задача 2"

  when: ansible_distribution == "Ubuntu"
```

#### Пример:

```
---
- name: Пример использования set_fact, register, when, debug, block
  hosts: localhost
  gather_facts: yes
  tasks:
    - name: Установить пользовательскую переменную через set_fact
      set_fact:
        custom_message: "Привет от Ansible!"

    - name: Отладка — показать значение custom_message
      debug:
        msg: "{{ custom_message }}"

    - name: Выполнить shell-команду и зарегистрировать результат
      shell: echo "Текст из команды"
      register: shell_output

    - name: Отладка — показать stdout команды
      debug:
        msg: "{{ shell_output.stdout }}"

    - name: Группа задач под блоком с условием
      block:
        - name: Задача 1 внутри блока
          debug:
            msg: "Эта задача внутри блока."

        - name: Задача 2 внутри блока
          debug:
            msg: "ОС: {{ ansible_distribution }} {{ ansible_distribution_version }}"
      when: ansible_distribution == "Ubuntu" or ansible_distribution == "Debian"

    - name: Условная задача (выполняется только на Ubuntu)
      debug:
        msg: "Это Ubuntu!"
      when: ansible_distribution == "Ubuntu"

    - name: Условная задача (выполняется не на Ubuntu)
      debug:
        msg: "Это не Ubuntu!"
      when: ansible_distribution != "Ubuntu"
```

- Устанавливается кастомная переменная `custom_message`.
- Выводится её значение через `debug`.
- Выполняется команда `echo`, её результат сохраняется в `shell_output`.
- Этот результат выводится.
- Блок задач выполняется только если ОС — Ubuntu или Debian.