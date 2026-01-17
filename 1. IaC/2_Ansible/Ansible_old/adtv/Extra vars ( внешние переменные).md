
Допустим, чтобы нам в основном плейбуке, чтобы каждый раз  не менять значение hosts, можно задать переменную:д

```
- name: install nginx
  hosts: "{{ HOSTS }}"
  become: yes

  roles:
    - { role: deploy_nginx }
```

Мы создали переменную HOSTS, заключена в {{ }}
Для применения внешней переменной, дополнительно при запуске плейбука вызываем --extra-var:
```
ansible-playbook playbook/nginx.yml --extra-vars "HOSTS=servers owner=admin" --ask-become-pass
```

Переменные в extra vars имеют наивысшее значение,  они заменяют все остыльные переменые, если переменные с таким же именем. 