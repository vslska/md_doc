
| ЦИКЛ            | НАЗНАЧЕНИЕ                            |
| --------------- | ------------------------------------- |
| `loop`          | Современный цикл по списку            |
| `with_items`    | Устаревший аналог `loop`              |
| `until`         | Повторять задачу, пока не станет true |
| `with_fileglob` | Поиск файлов локально по маске        |

Ниже краткое описание **основных типов циклов в Ansible** , которые позволяют выполнять задачи несколько раз с разными данными:
##### 1. **`loop` (современный способ, рекомендуется)**
Выполняет задачу для каждого элемента из списка.
###### Пример:
```
- - name: Copy mysite to Servers
    copy:
       src: "{{ source_folder }}/{{ item }}" #соурс переменная, в данном случае каталог, указан в vars, переменная item ведет нас к loop
       dest: "{{ destin_folder }}"           # dest, куда отправляем данные, указан в vars
       mode: 0555    # ======================= выставялем права 555
    loop:            # ======================= вызываем loop
      - "index.html" # ======================= копируем index.html
      - "static"     # ======================= копируем каталог static, в нем style.css
```

##### 2. **`with_items` (устаревший синтаксис, но встречается в старых playbook'ах)**
То же самое, что и `loop`, но устаревший стиль.
###### Пример:
```
- name: Копировать файлы
  copy:
    src: "{{ item }}"
    dest: "/backup/"
  with_items:
    - file1.txt
    - file2.tx
```

##### 3. **`until` (повторять до тех пор, пока не выполнится условие)**
Полезен для ожидания успешного состояния (например, доступности сервиса).
###### Пример:
```
- name: Ждать запуска сервиса
  uri:
    url: http://localhost:8080/health
  register: result
  until: result.status == 200
  retries: 10
  delay: 5
```

##### 4. **`with_fileglob`**
Используется для поиска файлов на **локальном контролирующем хосте** по маске.

> ⚠️ Работает только с модулем `copy` или `template` при использовании `delegate_to: localhost`.
###### Пример:
```
- name: Копировать все .conf файлы
  copy:
    src: "{{ item }}"
    dest: /etc/myapp/
  with_fileglob:
    - files/*.con
Или

 - name: Copy mysite to Servers
    copy:
       src: "{{ item }}" #соурс переменная, в данном случае каталог, указан в vars
       dest: "{{ destin_folder }}"           # dest, куда отправляем данные, указан в vars
       mode: 0555    # ======================= выставялем права 555
    with_fileglob: "{{ source_folder }}/*.*"
         
```

Общий пример:
```
---
- name: Пример использования разных циклов в одном плейбуке
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Использование loop (современный способ)
      debug:
        msg: "Пакет: {{ item }}"
      loop:
        - nginx
        - python3
        - curl

    - name: Использование with_items (устаревший способ)
      debug:
        msg: "Файл: {{ item }}"
      with_items:
        - file1.txt
        - file2.txt

    - name: Создать тестовые файлы локально для with_fileglob
      delegate_to: localhost
      run_once: yes
      copy:
        content: "Тестовый конфиг"
        dest: "/tmp/{{ item }}"
      loop:
        - test1.conf
        - test2.conf
        - dev.conf

    - name: Использование with_fileglob — копировать .conf файлы
      delegate_to: localhost
      copy:
        src: "{{ item }}"
        dest: "/tmp/backup/"
      with_fileglob:
        - /tmp/*.conf

    - name: Использование until — ждать появления файла
      stat:
        path: "/tmp/test1.conf"
      register: file_stat
      until: file_stat.stat.exists
      retries: 5
      delay: 2
```

1. **`loop`:** Выводит список пакетов.
2. **`with_items`:** Перечисляет файлы (устаревший синтаксис).
3. **Создаёт тестовые `.conf` файлы локально** для демонстрации `with_fileglob`.
4. **`with_fileglob`:** Копирует все `.conf` файлы из указанной директории.
5. **`until`:** Проверяет существование файла, повторяя проверку до успеха.