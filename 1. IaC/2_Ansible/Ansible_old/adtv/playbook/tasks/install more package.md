
```
- name: install many packages
    apt:
      name: "{{ item }}"
      state: latest
    loop:
      - ranger
      - python3
      - tree
```

name: "{{ item }}" # имя айтемов которые указаны в loop, заносим в кавычки.